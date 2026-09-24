# DeliveryRisk Engine — Feature Audit

## Prediction Point

The model makes its prediction at the moment an order is placed.

Therefore, every predictive feature must either:

1. Be directly available at order time, or
2. Be derived only from information available before that order.

## Target

`late_delivery`

A delivered order is labelled:

- `1` if actual customer delivery timestamp is later than the estimated delivery timestamp.
- `0` otherwise.

Only orders with `order_status = delivered` are used for the first prediction task.

## Feature Categories

### 🟢 Safe at Order Time

These describe information that can reasonably be known when the order is placed.

| Feature | Source | Treatment |
|---|---|---|
| customer location | customers | Use |
| seller location | sellers | Use |
| product category | products | Use |
| product weight | products | Use |
| product dimensions | products | Use |
| item price | order_items | Use |
| freight value | order_items | Use |
| number of items | order_items | Derive |
| purchase hour | orders | Derive |
| purchase day | orders | Derive |
| purchase month | orders | Derive |

### 🟡 Historical Features

These can be powerful, but must only use information from orders that occurred before the current order.

Examples:

- seller historical late-delivery rate
- seller historical order volume
- seller historical average delivery performance
- customer historical order count
- customer historical late-order rate
- regional historical late-delivery rate

Historical features must be calculated chronologically to prevent future information from leaking into the model.

### 🔴 Leakage

The following describe events that happen after the prediction point and must not be used as predictive features:

- actual customer delivery timestamp
- actual carrier handoff timestamp
- review score
- review comment
- post-delivery information
- any feature derived from the future outcome

These fields may be used to construct labels or evaluate the system, but not as model inputs.

### ⚪ Identifiers

Identifiers are useful for joining tables but should generally not be supplied directly to the ML model.

Examples:

- order_id
- customer_id
- seller_id
- product_id

## Core Principle

No information from the future may be used to predict the future.

The feature pipeline must reproduce what an operations team could realistically know at the time an order is placed.