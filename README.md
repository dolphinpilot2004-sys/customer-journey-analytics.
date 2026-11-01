# Customer Journey Analytics & Friction Detection

A reproducible, synthetic, end-to-end data engineering project that unifies web, app, support and purchase events to identify customer-journey drop-offs and high-friction paths.

## What it does

- Creates relational master data and dependent transaction/event records.
- Generates cross-channel journeys with event-time and ingestion-time timestamps.
- Applies consent-aware identity stitching and explicit sessionization rules.
- Produces conversion, abandonment and support-contact outcomes from source events.
- Injects four labelled anomaly families without silently corrupting unrelated data.
- Validates keys, relationships, event ordering, enums, lifecycle rules and reproducibility.
- Computes funnel, channel, latency, drop-off and friction metrics.

**All records are synthetic.** No personal or confidential data is collected.

## Architecture

```text
Customer / Page / Product / Experiment
                |
                v
       Session (sessionized journey)
                |
                v
 JourneyEvent (web / app / support / purchase)
       |         |          |
       v         v          v
      Cart      Order   SupportTicket
       \         |          /
        \        |         /
         v       v        v
             JourneyOutcome
                    |
                    v
          Analytics & QA reports
```

## Entities and relationships

| Entity | Primary key | Relationships / cardinality |
|---|---|---|
| Customer | customer_id | 1:N sessions; optional for anonymous sessions |
| Session | session_id | N:1 customer (optional); 1:N events; 0:1 outcome |
| JourneyEvent | event_id | N:1 session; optional N:1 page/product/cart/order/ticket/experiment |
| Page | page_id | 1:N events |
| Product | product_id | 1:N events and cart items |
| Cart | cart_id | N:1 session; 1:N cart items |
| CartItem | cart_item_id | N:1 cart and product |
| Order | order_id | N:1 customer and session; 1:N order items |
| OrderItem | order_item_id | N:1 order and product |
| SupportTicket | ticket_id | N:1 session; optional N:1 customer |
| Experiment | experiment_id | 1:N session assignments |
| SessionExperiment | assignment_id | bridge between session and experiment |
| JourneyOutcome | outcome_id | 1:1 session; derived from event/transaction records |

Effective dates: customer consent has `consent_updated_at`; product price is represented by `unit_price` at transaction time; event timestamps preserve both `event_ts` and `ingested_at`. Lifecycle changes are retained as events rather than overwriting journey history.

## Event taxonomy

`page_view`, `product_view`, `search`, `add_to_cart`, `remove_from_cart`, `checkout_started`, `payment_failed`, `purchase_completed`, `support_contacted`, `support_resolved`, `app_open`, `form_error`, `shipping_selected`, `experiment_exposed`.

Channels: `web`, `app`, `support`, `purchase`.

Funnel stages: `discovery` -> `consideration` -> `cart` -> `checkout` -> `purchase`.
A session reaches a stage when it contains at least one corresponding event. Conversion requires a valid, non-bot `purchase_completed` event linked to an order. A support outcome is assigned when a support contact exists and no valid purchase occurred. Otherwise, a session with meaningful activity is abandonment. Bot sessions are labelled `bot_excluded`.

## Sessionization and identity rules

- A session is a generated visit with a single `session_id`; events are grouped by that ID.
- Events with an inactivity gap greater than 30 minutes would begin a new session in a raw event stream. The generator emits pre-sessionized sessions and records this rule for reproducibility.
- Event occurrence time is `event_ts`; arrival time is `ingested_at`. Analytics must sort by occurrence time, not ingestion time.
- Customer identity is attached only when `consent_status='granted'`. Anonymous sessions retain `customer_id=NULL`.
- Device identifiers are synthetic. Cross-device customer linkage is permitted only for consented customer records.
- Bot traffic is identified using `is_bot`; it is excluded from customer conversion and funnel denominators.

## Deliberate anomaly scenarios

1. **Out-of-order ingestion:** event occurrence and ingestion order differ; event-time analytics must restore chronology.
2. **Anonymous identity merge:** an anonymous event carries a suspicious `identity_candidate_customer_id`; the candidate is not a valid identity link and must not be used.
3. **Duplicate conversion:** a second purchase-completed event references the same order; deduplication counts one order once.
4. **Bot/customer contamination:** bot events mimic customer activity; bot sessions are excluded from customer metrics.

`ground_truth.csv` is kept separate from model/input data and records anomaly type, affected entity, expected treatment and reason.

## Quick start

Python 3.10+ recommended. Install dependencies:

```bash
pip install -r requirements.txt
```

Generate a small, inspectable dataset:

```bash
python src/generate_data.py --sessions 1000 --events-per-session 20 --seed 20261022 --out data/sample
```

Generate the required full baseline:

```bash
python src/generate_data.py --sessions 100000 --events-per-session 20 --seed 20261022 --out data/full
```

Run analytics and validation:

```bash
python src/analyze.py --input data/sample --out reports/sample
python src/validate.py --input data/sample --out reports/sample
```

Create a SQLite database from the generated CSV files:

```bash
python src/load_sqlite.py --input data/sample --db data/sample/journey.db
```

The full generator is parameterized: the same seed and parameters produce deterministic IDs and data. Use the sample data for quick review; the full run creates approximately two million journey events.

## Metrics

- Session conversion rate = unique valid converted sessions / eligible non-bot sessions.
- Funnel stage reach = unique eligible sessions with at least one event in the stage.
- Stage drop-off = (sessions reaching stage N - sessions reaching next stage) / sessions reaching stage N.
- Support-contact rate = eligible sessions with support contact / eligible sessions.
- Event ingestion delay = `ingested_at - event_ts` in seconds; report median and p95.
- Duplicate conversion count = excess purchase-completed events per order after first valid conversion.
- Bot exclusion rate = bot sessions / all sessions.
- Consent coverage = sessions with granted identity consent / all sessions.

## Data quality and reproducibility

Validation checks primary-key uniqueness, foreign-key integrity, allowed enums, timestamp ordering, valid lifecycle states, arithmetic reconciliation and duplicate conversion patterns. It writes a machine-readable `validation_summary.json`.

To demonstrate reproducibility, run the same generator command twice into separate directories and compare checksums:

```bash
python src/reproducibility.py --sessions 1000 --events-per-session 20 --seed 20261022
```

## Output files

Generator outputs CSVs for all entities, `ground_truth.csv`, `source_manifest.json` and `generation_summary.json`. Data dictionary: `DATA_DICTIONARY.md`. SQL DDL: `sql/schema.sql`.

## Important interpretation notes

The data is intentionally synthetic and distributions are configurable in source code. Results are useful for demonstrating data modelling, data quality, event processing and analytics logic; they are not estimates of real customer behaviour. Ground-truth labels are QA metadata and must never be joined into model features or operational decision inputs.
