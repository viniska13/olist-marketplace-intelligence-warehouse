# Source-to-Target Mapping

Names in the Staging and Target columns are proposals. They will be finalized in the modeling tasks.

## Table-level mapping

| Source file | Staging model | Cleaning in staging | Proposed target |
|---|---|---|---|
| olist_orders_dataset | `stg_orders` | Cast timestamps; keep nulls for unfinished orders | `fct_orders` |
| olist_order_items_dataset | `stg_order_items` | Cast `shipping_limit_date`; flag out-of-range dates | `fct_order_items` |
| olist_order_payments_dataset | `stg_payments` | Map `not_defined` to `unknown`; flag zero values and zero installments | `fct_payments` |
| olist_order_reviews_dataset | `stg_reviews` | Key on (`review_id`, `order_id`); keep comments nullable | `fct_reviews` |
| olist_customers_dataset | `stg_customers` | Keep both `customer_id` and `customer_unique_id` | `dim_customers` |
| olist_sellers_dataset | `stg_sellers` | Standardize city and state text | `dim_sellers` |
| olist_products_dataset | `stg_products` | Rename misspelled columns; treat weight 0 as unknown; label missing category `unknown` | `dim_products` |
| product_category_name_translation | `stg_category_translation` | Add 2 missing English names manually | used by `dim_products` |
| olist_geolocation_dataset | `stg_geolocation` | Reduce to one row per ZIP prefix | `dim_geography` |

## Column-level changes

| Source table | Source column | Staging column | Change |
|---|---|---|---|
| products | product_name_lenght | product_name_length | Spelling fix |
| products | product_description_lenght | product_description_length | Spelling fix |
| customers | customer_unique_id | customer_unique_id | Used as the person key for retention and RFM |
| payments | payment_type | payment_type | `not_defined` becomes `unknown` |
| geolocation | geolocation_zip_code_prefix | zip_code_prefix | Deduplicated to one row per prefix |

## Key relationships

| Child | Column | Parent | Column |
|---|---|---|---|
| orders | customer_id | customers | customer_id |
| order_items | order_id | orders | order_id |
| order_items | product_id | products | product_id |
| order_items | seller_id | sellers | seller_id |
| payments | order_id | orders | order_id |
| reviews | order_id | orders | order_id |
| products | product_category_name | category translation | product_category_name |
| customers | customer_zip_code_prefix | geolocation | geolocation_zip_code_prefix |
| sellers | seller_zip_code_prefix | geolocation | geolocation_zip_code_prefix |