# Source Data Profile

Profiled with `notebooks/01_data_profiling.ipynb` on 2026-10-25.
Source: Olist Brazilian E-Commerce Public Dataset (Kaggle), 9 CSV files kept in `data/raw/` (ignored by Git).

## 1. Table overview, grain and keys

| Source file | Rows | Cols | Grain (one row = ...) | Primary key |
|---|---|---|---|---|
| olist_orders_dataset | 99,441 | 8 | one customer order | `order_id` |
| olist_order_items_dataset | 112,650 | 7 | one item line within an order | `order_id` + `order_item_id` |
| olist_order_payments_dataset | 103,886 | 5 | one payment on an order | `order_id` + `payment_sequential` |
| olist_order_reviews_dataset | 99,224 | 7 | one review attached to one order | `review_id` + `order_id` (`review_id` alone is NOT unique) |
| olist_customers_dataset | 99,441 | 5 | one customer record per order | `customer_id` (the real person is `customer_unique_id`) |
| olist_sellers_dataset | 3,095 | 4 | one seller | `seller_id` |
| olist_products_dataset | 32,951 | 9 | one product | `product_id` |
| product_category_name_translation | 71 | 2 | one category-name translation | `product_category_name` |
| olist_geolocation_dataset | 1,000,163 | 5 | one coordinate record (many per ZIP prefix) | none (no clean key) |

## 2. Foreign-key coverage

| Child column | Parent | Missing | Result |
|---|---|---|---|
| orders.customer_id | customers | 0 of 99,441 | clean |
| order_items.order_id | orders | 0 of 112,650 | clean |
| order_items.product_id | products | 0 of 112,650 | clean |
| order_items.seller_id | sellers | 0 of 112,650 | clean |
| payments.order_id | orders | 0 of 103,886 | clean |
| reviews.order_id | orders | 0 of 99,224 | clean |
| products.product_category_name | translation | 13 of 32,341 (0.04%) | 2 categories have no translation |
| customers.customer_zip_code_prefix | geolocation | 278 of 99,441 (0.28%) | ZIP not in geolocation |
| sellers.seller_zip_code_prefix | geolocation | 7 of 3,095 (0.23%) | ZIP not in geolocation |

Reverse checks on orders: 775 have no items, 1 has no payments, 768 have no reviews.

## 3. Null patterns

| Table | Column | Nulls | % |
|---|---|---|---|
| orders | order_approved_at | 160 | 0.16 |
| orders | order_delivered_carrier_date | 1,783 | 1.79 |
| orders | order_delivered_customer_date | 2,965 | 2.98 |
| reviews | review_comment_title | 87,656 | 88.34 |
| reviews | review_comment_message | 58,247 | 58.70 |
| products | product_category_name | 610 | 1.85 |
| products | product_name_lenght | 610 | 1.85 |
| products | product_description_lenght | 610 | 1.85 |
| products | product_photos_qty | 610 | 1.85 |
| products | product_weight_g, product_length_cm, product_height_cm, product_width_cm | 2 each | 0.01 |

No nulls in order_items, payments, customers, sellers or geolocation.

## 4. Date ranges

| Column | Min | Max |
|---|---|---|
| orders.order_purchase_timestamp | 2016-09-04 21:15:19 | 2018-10-17 17:30:18 |
| orders.order_approved_at | 2016-09-15 12:16:38 | 2018-09-03 17:40:06 |
| orders.order_delivered_carrier_date | 2016-10-08 10:34:01 | 2018-09-11 19:48:28 |
| orders.order_delivered_customer_date | 2016-10-11 13:46:32 | 2018-10-17 13:22:46 |
| orders.order_estimated_delivery_date | 2016-09-30 00:00:00 | 2018-11-12 00:00:00 |
| order_items.shipping_limit_date | 2016-09-19 00:15:34 | 2020-04-09 22:35:08 |
| reviews.review_creation_date | 2016-10-02 00:00:00 | 2018-08-31 00:00:00 |
| reviews.review_answer_timestamp | 2016-10-07 18:32:28 | 2018-10-29 12:27:35 |

## 5. Categorical values

- order_status: delivered 96,478; shipped 1,107; canceled 625; unavailable 609; invoiced 314; processing 301; created 5; approved 2.
- payment_type: credit_card 76,795; boleto 19,784; voucher 5,775; debit_card 1,529; not_defined 3.
- review_score: 1 = 11,424; 2 = 3,151; 3 = 8,179; 4 = 19,142; 5 = 57,328.
- Distinct values: 27 customer states, 23 seller states, 73 product categories (but only 71 translations).
- customer_id: 99,441 distinct; customer_unique_id: 96,096 distinct.

## 6. Order volume by month

- Sparse start: 2016-09 = 4 orders, 2016-10 = 324, 2016-11 = no orders, 2016-12 = 1.
- Ramp-up: 2017-01 = 800, 2017-02 = 1,780, 2017-03 = 2,682.
- Peak seen: 2018-08 = 6,512.
- Sparse end: 2018-09 = 16, 2018-10 = 4.
- 25 distinct months in total.

## 7. Items and payments per order

- Items per order: average 1.14, maximum 21 (98,666 orders have at least one item).
- Payments per order: average 1.04, maximum 29 (99,440 orders have at least one payment).


## 8. Known data issues

| ID | Table | Issue | Evidence | Proposed handling |
|---|---|---|---|---|
| DQ-01 | reviews | `review_id` is reused across different orders | 814 duplicate `review_id` rows; 98,410 distinct ids vs 99,224 rows; 0 duplicates on (`review_id`, `order_id`) | Key on (`review_id`, `order_id`) |
| DQ-02 | reviews | Some orders have more than one review | 547 orders | Pick one review per order in the fact, or aggregate, before joining to orders |
| DQ-03 | customers | `customer_id` is per order, not per person | 99,441 `customer_id` vs 96,096 `customer_unique_id` | Use `customer_unique_id` for retention and RFM |
| DQ-04 | geolocation | Many rows per ZIP prefix, many exact duplicates | 1,000,163 rows; 19,015 distinct prefixes; 261,831 fully duplicated rows | Reduce to one row per ZIP prefix before any join |
| DQ-05 | customers, sellers | ZIP prefix not found in geolocation | 278 customers, 7 sellers | Keep rows; leave location null |
| DQ-06 | products | No category and no listing details | 610 products (1.85%) | Label as `unknown` category |
| DQ-07 | products | Categories without an English translation | 2 categories, 13 products: `portateis_cozinha_e_preparadores_de_alimentos` (10), `pc_gamer` (3) | Add the English names manually in staging |
| DQ-08 | products | Column names misspelled in source | `product_name_lenght`, `product_description_lenght` | Rename to `length` in staging |
| DQ-09 | products | Missing or zero weight and size | 2 nulls in weight and each dimension; 4 products with weight 0 | Treat 0 as unknown; do not use for freight analysis |
| DQ-10 | orders | Orders with no items | 775: unavailable 603, canceled 164, created 5, invoiced 2, shipped 1 | Exclude from item-level facts |
| DQ-11 | orders | A delivered order with no payment | 1 order, status delivered | Flag with a test; exclude from payment metrics |
| DQ-12 | orders | Delivered orders missing timestamps | 8 no customer delivery date; 2 no carrier date; 14 no approval date | Exclude from on-time and duration metrics; flag with tests |
| DQ-13 | orders | Delivery nulls for unfinished orders | Explained by status (shipped, canceled, unavailable, invoiced, processing, created, approved) | Not an error; keep null |
| DQ-14 | payments | Zero-value payments | 9 rows: 6 voucher, 3 not_defined | Exclude zero-value `not_defined` from revenue; keep vouchers |
| DQ-15 | payments | Zero installments | 2 rows | Flag with a test |
| DQ-16 | payments | Payment type `not_defined` | 3 rows | Map to `unknown` |
| DQ-17 | order_items | Shipping limit date outside the dataset period | 4 rows after 2018; latest 2020-04-09 | Flag with a date-range test |
| DQ-18 | orders | Months with almost no data | 2016-09, 2016-10, 2016-11 (none), 2016-12, 2018-09, 2018-10 | Dashboards should use 2017-01 to 2018-08 as the full-month window |
| DQ-19 | orders | Several items and payments per order | Up to 21 items and 29 payments per order | Aggregate items and payments separately before joining |
| DQ-20 | reviews | Comments are mostly empty | Title 88.34% null; message 58.70% null | Keep nulls; never require comments |
| DQ-21 | products | Category names are in Portuguese | 73 categories | Join to translation; expose English name |