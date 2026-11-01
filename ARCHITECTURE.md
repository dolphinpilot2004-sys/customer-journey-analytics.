# Architecture and decision traceability

## Processing layers
1. **Master data:** Customer, Page, Product, Experiment.
2. **Journey layer:** Session, SessionExperiment and JourneyEvent.
3. **Transactions:** Cart/CartItem, Order/OrderItem and SupportTicket.
4. **Derived layer:** JourneyOutcome and analytics reports.
5. **QA layer:** ground_truth.csv, validation_summary.json and source_manifest.json.

## Decision rules
- Identity: populate customer_id only when session consent is `granted`; ignore `identity_candidate_customer_id`.
- Chronology: order journey events by `event_ts`; use `ingested_at` only for latency and late-arrival analysis.
- Deduplication: conversion is distinct `order_id` per session, not raw count of purchase events.
- Bots: exclude `is_bot=true` sessions from customer funnel/conversion denominators.
- Outcome precedence: bot_excluded > converted > support_contact > abandoned.
- Funnel: stage reached is derived from event taxonomy; never independently randomized.

## Exception handling
Each injected anomaly has an entry in ground_truth.csv. This file is QA-only and is not read by analytics.py. Suspicious identity candidates are retained as evidence but are not accepted as customer links.

## Reconciliation
Order total equals the sum of order-item quantity * unit_price. Conversion value is copied from the reconciled order total. Event and ingestion times are retained separately.
