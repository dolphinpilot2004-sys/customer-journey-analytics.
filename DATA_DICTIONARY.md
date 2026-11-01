# Data dictionary

All timestamps are ISO-8601 UTC strings. IDs are immutable strings. Monetary values are decimal USD. `required` means non-null in generated baseline unless marked optional.

| Entity / field | Type | Required | Allowed values / units | Uniqueness / validation |
|---|---|---:|---|---|
| Customer.customer_id | string | yes | `CUS-000001` | PK, unique |
| Customer.created_at | timestamp | yes | UTC | <= consent_updated_at |
| Customer.consent_status | enum | yes | granted, denied, unknown | enum |
| Customer.consent_updated_at | timestamp | yes | UTC | >= created_at |
| Customer.acquisition_channel | enum | yes | web, app, support, referral, email, paid_search | enum |
| Customer.country | string | yes | synthetic ISO-like country code | non-empty |
| Session.session_id | string | yes | `SES-2026-000001` | PK, unique |
| Session.customer_id | string | no | Customer ID or null | FK; only populated if consent granted |
| Session.anonymous_id | string | yes | `ANON-...` | non-empty |
| Session.device_id | string | yes | `DEV-...` | non-empty |
| Session.started_at | timestamp | yes | UTC | <= ended_at |
| Session.ended_at | timestamp | yes | UTC | >= started_at |
| Session.initial_channel | enum | yes | web, app, support | enum |
| Session.is_bot | boolean | yes | true/false | boolean |
| Session.consent_status | enum | yes | granted, denied, unknown | enum |
| JourneyEvent.event_id | string | yes | `EVT-<session-id>-000001` | PK, unique |
| JourneyEvent.session_id | string | yes | Session ID | FK |
| JourneyEvent.customer_id | string | no | Customer ID or null | FK; consent gated |
| JourneyEvent.event_name | enum | yes | taxonomy in README | enum |
| JourneyEvent.channel | enum | yes | web, app, support, purchase | enum |
| JourneyEvent.event_ts | timestamp | yes | UTC | within session +/- injected delay tolerance |
| JourneyEvent.ingested_at | timestamp | yes | UTC | >= event_ts normally; labelled exception may differ |
| JourneyEvent.page_id | string | no | Page ID | FK |
| JourneyEvent.product_id | string | no | Product ID | FK |
| JourneyEvent.cart_id | string | no | Cart ID | FK |
| JourneyEvent.order_id | string | no | Order ID | FK |
| JourneyEvent.ticket_id | string | no | Ticket ID | FK |
| JourneyEvent.experiment_id | string | no | Experiment ID | FK |
| JourneyEvent.device_id | string | yes | Device ID | non-empty |
| JourneyEvent.is_bot | boolean | yes | true/false | boolean |
| JourneyEvent.identity_candidate_customer_id | string | no | Customer ID | suspicious candidate only; never authoritative |
| Page.page_id | string | yes | `PAG-000001` | PK |
| Page.page_name | string | yes | synthetic page name | unique |
| Page.funnel_stage | enum | yes | discovery, consideration, cart, checkout, purchase, support | enum |
| Page.page_type | enum | yes | landing, listing, detail, cart, checkout, help | enum |
| Product.product_id | string | yes | `PRD-000001` | PK |
| Product.product_name | string | yes | synthetic product | unique |
| Product.category | enum | yes | electronics, home, apparel, beauty, sports | enum |
| Product.unit_price | decimal | yes | USD > 0 | > 0 |
| Product.active | boolean | yes | true/false | boolean |
| Cart.cart_id | string | yes | `CRT-000001` | PK |
| Cart.session_id | string | yes | Session ID | FK |
| Cart.created_at | timestamp | yes | UTC | within session |
| Cart.status | enum | yes | open, converted, abandoned | lifecycle |
| CartItem.cart_item_id | string | yes | `CIT-000001` | PK |
| CartItem.cart_id | string | yes | Cart ID | FK |
| CartItem.product_id | string | yes | Product ID | FK |
| CartItem.quantity | integer | yes | units >= 1 | positive |
| Order.order_id | string | yes | `ORD-000001` | PK |
| Order.session_id | string | yes | Session ID | FK |
| Order.customer_id | string | no | Customer ID | FK; consent gated |
| Order.ordered_at | timestamp | yes | UTC | within session |
| Order.status | enum | yes | placed, paid, cancelled | lifecycle |
| Order.total_amount | decimal | yes | USD >= 0 | sum of order items |
| OrderItem.order_item_id | string | yes | `OIT-000001` | PK |
| OrderItem.order_id | string | yes | Order ID | FK |
| OrderItem.product_id | string | yes | Product ID | FK |
| OrderItem.quantity | integer | yes | units >= 1 | positive |
| OrderItem.unit_price | decimal | yes | USD > 0 | transaction-time price |
| SupportTicket.ticket_id | string | yes | `TKT-000001` | PK |
| SupportTicket.session_id | string | yes | Session ID | FK |
| SupportTicket.customer_id | string | no | Customer ID | FK; consent gated |
| SupportTicket.created_at | timestamp | yes | UTC | within session |
| SupportTicket.status | enum | yes | open, resolved | lifecycle |
| SupportTicket.reason | enum | yes | payment, delivery, product, account, technical | enum |
| Experiment.experiment_id | string | yes | `EXP-2026-000001` | PK |
| Experiment.experiment_name | string | yes | synthetic name | unique |
| Experiment.start_date | date | yes | ISO date | <= end_date |
| Experiment.end_date | date | yes | ISO date | >= start_date |
| SessionExperiment.assignment_id | string | yes | `ASN-000001` | PK |
| SessionExperiment.session_id | string | yes | Session ID | FK |
| SessionExperiment.experiment_id | string | yes | Experiment ID | FK |
| SessionExperiment.variant | enum | yes | control, treatment | enum |
| JourneyOutcome.outcome_id | string | yes | `OUT-<session-id>` | PK, unique |
| JourneyOutcome.session_id | string | yes | Session ID | FK, unique 1:1 |
| JourneyOutcome.outcome | enum | yes | converted, support_contact, abandoned, bot_excluded | derived |
| JourneyOutcome.funnel_stage_reached | enum | yes | discovery, consideration, cart, checkout, purchase, none | derived |
| JourneyOutcome.conversion_value | decimal | yes | USD >= 0 | reconciles to valid order total |
| JourneyOutcome.dropoff_stage | enum | no | consideration, cart, checkout, purchase | nullable |
| JourneyOutcome.derived_at | timestamp | yes | UTC | >= session end |
| GroundTruth.anomaly_id | string | yes | `ANM-000001` | PK |
| GroundTruth.entity_id | string | yes | affected event/session/order ID | reference to affected record |
| GroundTruth.issue_type | enum | yes | delayed_ingestion, anonymous_identity_merge, duplicate_conversion, bot_contamination | enum |
| GroundTruth.expected_treatment | string | yes | documented decision | non-empty |
| GroundTruth.reason | string | yes | explanation | non-empty |
