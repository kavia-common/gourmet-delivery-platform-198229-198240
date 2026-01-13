# Gourmet Delivery Platform — MySQL Schema Notes

This schema and seed data were applied using the MySQL CLI one statement at a time (per runtime rules). Connection used:

`mysql -u appuser -pdbuser123 -h localhost -P 5000 myapp`

## Standards Used
- Engine: **InnoDB** for all tables
- Identifier naming: snake_case
- Primary key strategy (consistent across all tables):
  - `uid BIGINT UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY`
- Mandatory timestamps on every table:
  - `created_date TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP`
  - `modified_date TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP`
- Foreign keys:
  - `ON UPDATE CASCADE`
  - `ON DELETE RESTRICT`

## Tables
### users
Columns:
- `email` (unique, required)
- `password_hash` (required)
- `role` (required, default `customer`, CHECK in `customer|restaurant_admin|courier|admin`)

### profiles
1:1 with users:
- `user_uid` (unique FK -> `users.uid`)
Optional: name/phone/address fields.

### restaurants
Basic restaurant fields + `is_open BOOLEAN DEFAULT TRUE`.
`name` is unique.

### menu_items
- FK: `restaurant_uid -> restaurants.uid`
Pricing stored as integer cents: `price_cents INT UNSIGNED`.

### orders
- FK: `user_uid -> users.uid`
- FK: `restaurant_uid -> restaurants.uid`
- Status CHECK: `placed|accepted|preparing|ready|out_for_delivery|delivered|cancelled`
- Totals in cents: `subtotal_cents`, `delivery_fee_cents`, `total_cents`
- Delivery address snapshot columns (nullable)

### order_items
- FK: `order_uid -> orders.uid`
- FK: `menu_item_uid -> menu_items.uid`
- `quantity`, `unit_price_cents`, `line_total_cents`

### tracking_events
- FK: `order_uid -> orders.uid`
- `event_type`, optional `event_message`
- Optional `latitude/longitude`
- `occurred_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP`

## Seed Data (E2E Baseline)
Seed data was inserted after a clean reset (dropping tables then recreating).

Expected baseline counts:
- `users`: 1 (`testuser@example.com`)
- `profiles`: 1 (linked to user 1)
- `restaurants`: 3 (`Pasta Palace`, `Sushi Central`, `Taco Town`)
- `menu_items`: 6 (2 per restaurant)
- `orders`: 1 (user 1 ordering from restaurant 1)
- `order_items`: 2 (two items on order 1)
- `tracking_events`: 3 (`placed`, `accepted`, `preparing` for order 1)

Note: If additional rows already exist in a persistent DB, auto-increment `uid` values may differ; the schema remains the same, but seed rows will not necessarily have the same numeric IDs.
""""""
