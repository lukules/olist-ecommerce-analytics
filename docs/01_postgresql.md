# Database Creation
```sql
CREATE DATABASE ecommerce;
```

# Schema Creation
Two schemas were created within the database:
```sql
CREATE SCHEMA IF NOT EXISTS staging;
CREATE SCHEMA IF NOT EXISTS analytics;
```

The schemas were designed to separate raw imported data from analytical objects.

| Schema | Purpose |
| --- | --- |
| `staging` | Storage of source data imported from CSV files, with a structure based on the original dataset. |
| `analytics` | Intended location for analytical tables, views, and other objects prepared for reporting and analysis. |


# Expected Outcome
The following database structure was established:
```text
ecommerce
├── staging
└── analytics
```


# Table Structure
The tables were created in the `staging` schema.


### Category Translation

Table: `staging.product_category_name_translation`

Stores the mapping between Portuguese product category names and their English translations.

| Column | Data type | Constraint | Description |
| --- | --- | --- | --- |
| `product_category_name` | `VARCHAR(100)` | Primary key | Original product category name. |
| `product_category_name_english` | `VARCHAR(100)` | `NOT NULL` | English translation of the category name. |

This table supports consistent category labeling in analytical reports.



### Geolocation

Table: `staging.geolocation`

Stores geographic information associated with Brazilian postal code prefixes.

| Column | Data type | Constraint | Description |
| --- | --- | --- | --- |
| `geolocation_zip_code_prefix` | `VARCHAR(10)` | `NOT NULL` | Postal code prefix. |
| `geolocation_lat` | `NUMERIC(12,8)` | `NOT NULL` | Latitude. |
| `geolocation_lng` | `NUMERIC(12,8)` | `NOT NULL` | Longitude. |
| `geolocation_city` | `VARCHAR(100)` | `NOT NULL` | City name. |
| `geolocation_state` | `CHAR(2)` | `NOT NULL` | State abbreviation. |

The postal code prefix was stored as text to preserve its identifier format, including potential leading zeros.

No primary key was defined for this table because the source data may contain multiple records associated with the same postal code prefix.



### Customers

Table: `staging.customers`

Stores customer records and their location information.

| Column | Data type | Constraint | Description |
| --- | --- | --- | --- |
| `customer_id` | `VARCHAR(32)` | Primary key | Identifier of a customer record associated with an order. |
| `customer_unique_id` | `VARCHAR(32)` | `NOT NULL` | Identifier used to distinguish unique customers across records. |
| `customer_zip_code_prefix` | `VARCHAR(10)` | `NOT NULL` | Customer postal code prefix. |
| `customer_city` | `VARCHAR(100)` | `NOT NULL` | Customer city. |
| `customer_state` | `CHAR(2)` | `NOT NULL` | Customer state abbreviation. |

The distinction between `customer_id` and `customer_unique_id` is important for customer-level analysis.

A single real-world customer may be represented by multiple `customer_id` values. Therefore, customer order counts and unique customer counts may differ.



### Sellers

Table: `staging.sellers`

Stores seller identifiers and location information.

| Column | Data type | Constraint | Description |
| --- | --- | --- | --- |
| `seller_id` | `VARCHAR(32)` | Primary key | Unique seller identifier. |
| `seller_zip_code_prefix` | `VARCHAR(10)` | `NOT NULL` | Seller postal code prefix. |
| `seller_city` | `VARCHAR(100)` | `NOT NULL` | Seller city. |
| `seller_state` | `CHAR(2)` | `NOT NULL` | Seller state abbreviation. |

The seller table can later be used to analyze seller performance and geographic distribution.



### Products

Table: `staging.products`

Stores product attributes and physical dimensions.

| Column | Data type | Constraint | Description |
| --- | --- | --- | --- |
| `product_id` | `VARCHAR(32)` | Primary key | Unique product identifier. |
| `product_category_name` | `VARCHAR(100)` | Nullable | Original product category name. |
| `product_name_length` | `INT` | Nullable | Length of the product name. |
| `product_description_length` | `INT` | Nullable | Length of the product description. |
| `product_photos_qty` | `INT` | Nullable | Number of product photos. |
| `product_weight_g` | `INT` | Nullable | Product weight in grams. |
| `product_length_cm` | `INT` | Nullable | Product length in centimeters. |
| `product_height_cm` | `INT` | Nullable | Product height in centimeters. |
| `product_width_cm` | `INT` | Nullable | Product width in centimeters. |

Several attributes were allowed to contain `NULL` values because missing product information may occur in the source dataset.

The table can support product category, product catalog, and shipping-related analyses.



### Orders

Table: `staging.orders`

Stores order-level information, including order status and timestamps.

| Column | Data type | Constraint | Description |
| --- | --- | --- | --- |
| `order_id` | `VARCHAR(32)` | Primary key | Unique order identifier. |
| `customer_id` | `VARCHAR(32)` | `NOT NULL`, foreign key | Customer associated with the order. |
| `order_status` | `VARCHAR(20)` | `NOT NULL` | Current or recorded order status. |
| `order_purchase_timestamp` | `TIMESTAMP` | `NOT NULL` | Timestamp when the order was placed. |
| `order_approved_at` | `TIMESTAMP` | Nullable | Timestamp when the order was approved. |
| `order_delivered_carrier_date` | `TIMESTAMP` | Nullable | Timestamp when the order was handed over to the carrier. |
| `order_delivered_customer_date` | `TIMESTAMP` | Nullable | Timestamp when the order was delivered to the customer. |
| `order_estimated_delivery_date` | `TIMESTAMP` | `NOT NULL` | Estimated delivery timestamp. |

The foreign key references `staging.customers(customer_id)`.

The timestamp columns support analysis of order processing, delivery performance, and delivery delays.

Nullable timestamps are retained because some orders may not have reached the corresponding processing or delivery stage.



### Order Items

Table: `staging.order_items`

Stores individual product line items associated with orders.

| Column | Data type | Constraint | Description |
| --- | --- | --- | --- |
| `order_id` | `VARCHAR(32)` | Primary key component, foreign key | Associated order identifier. |
| `order_item_id` | `INT` | Primary key component | Item sequence number within the order. |
| `product_id` | `VARCHAR(32)` | `NOT NULL`, foreign key | Associated product identifier. |
| `seller_id` | `VARCHAR(32)` | `NOT NULL`, foreign key | Associated seller identifier. |
| `shipping_limit_date` | `TIMESTAMP` | `NOT NULL` | Deadline for handing the item over for shipping. |
| `price` | `NUMERIC(10,2)` | `NOT NULL` | Item price. |
| `freight_value` | `NUMERIC(10,2)` | `NOT NULL` | Shipping cost associated with the item. |

A composite primary key was defined using `(order_id, order_item_id)`.

This allows multiple products to be associated with one order while maintaining a unique identifier for each order item.

Foreign keys connect order items to the corresponding orders, products, and sellers.

The `price` and `freight_value` columns can be used to calculate product sales value and shipping costs.



### Order Payments

Table: `staging.order_payments`

Stores payment records associated with orders.

| Column | Data type | Constraint | Description |
| --- | --- | --- | --- |
| `order_id` | `VARCHAR(32)` | Primary key component, foreign key | Associated order identifier. |
| `payment_sequential` | `INT` | Primary key component | Sequence number of the payment within the order. |
| `payment_type` | `VARCHAR(30)` | `NOT NULL` | Payment method. |
| `payment_installments` | `INT` | `NOT NULL` | Number of installments. |
| `payment_value` | `NUMERIC(10,2)` | `NOT NULL` | Recorded payment value. |

The composite primary key `(order_id, payment_sequential)` allows multiple payment records to be associated with the same order.

This table supports payment method analysis, installment analysis, and comparisons between recorded payments and order values.



### Order Reviews

Table: `staging.order_reviews`

Stores customer review scores, comments, and timestamps.

| Column | Data type | Constraint | Description |
| --- | --- | --- | --- |
| `review_id` | `VARCHAR(32)` | Primary key component | Review identifier. |
| `order_id` | `VARCHAR(32)` | Primary key component, foreign key | Associated order identifier. |
| `review_score` | `SMALLINT` | `NOT NULL`, check constraint | Review score from 1 to 5. |
| `review_comment_title` | `TEXT` | Nullable | Review title. |
| `review_comment_message` | `TEXT` | Nullable | Review text. |
| `review_creation_date` | `TIMESTAMP` | `NOT NULL` | Review creation timestamp. |
| `review_answer_timestamp` | `TIMESTAMP` | `NOT NULL` | Timestamp of the review response. |

A composite primary key `(review_id, order_id)` was defined.

The check constraint ensures that review scores fall within the range from 1 to 5.

Review comments were allowed to contain `NULL` values because not every review necessarily includes written feedback.




# Relational Structure

Foreign key relationships were defined between the following tables:

| Child table | Foreign key | Referenced table |
| --- | --- | --- |
| `orders` | `customer_id` | `customers` |
| `order_items` | `order_id` | `orders` |
| `order_items` | `product_id` | `products` |
| `order_items` | `seller_id` | `sellers` |
| `order_payments` | `order_id` | `orders` |
| `order_reviews` | `order_id` | `orders` |

The foreign keys help prevent references to non-existent parent records when data is inserted or modified while the constraints are active.



# CSV Data Import
The objective of this stage was to import the source CSV files from the Brazilian E-Commerce Public Dataset by Olist into the PostgreSQL `staging` schema.

### Source Data

The following CSV files were imported:

| Source file | Target table | Purpose |
| --- | --- | --- |
| `product_category_name_translation.csv` | `staging.product_category_name_translation` | Product category translations. |
| `geolocation.csv` | `staging.geolocation` | Geographic coordinates and location data. |
| `customers.csv` | `staging.customers` | Customer records and locations. |
| `sellers.csv` | `staging.sellers` | Seller records and locations. |
| `products.csv` | `staging.products` | Product attributes and dimensions. |
| `orders.csv` | `staging.orders` | Order status and timestamps. |
| `order_items.csv` | `staging.order_items` | Items, prices, and shipping costs. |
| `order_payments.csv` | `staging.order_payments` | Payment methods and values. |
| `order_reviews.csv` | `staging.order_reviews` | Review scores, comments, and timestamps. |


### Import Method

The following pattern was used to import the CSV files:

```sql
COPY staging.customers
FROM 'D:/olist-ecommerce-analytics/data/raw/customers.csv'
WITH (
    FORMAT csv,
    HEADER true,
    DELIMITER ',',
    ENCODING 'UTF8'
);
```

The same approach was applied to the remaining source files, with the target table and file path adjusted accordingly.


### Import Order

Because foreign key constraints were defined between several tables, the import order must respect the parent-child relationships.

A suitable sequence is:

1. `product_category_name_translation`
2. `geolocation`
3. `customers`
4. `sellers`
5. `products`
6. `orders`
7. `order_items`
8. `order_payments`
9. `order_reviews`



### Import Validation
#### 1. The first ten records from each table were retrieved using `SELECT *` and `LIMIT 10`.

Example:

```sql
SELECT *
FROM staging.customers
LIMIT 10;
```

The same pattern was applied to all nine imported tables.


#### 2.Record Count Validation

The total number of records in each table was calculated using `COUNT(*)`.

The results were combined into a single result set using `UNION ALL`.

```sql
SELECT
    'product_category_name_translation' AS table_name,
    COUNT(*) AS record_count
FROM staging.product_category_name_translation

UNION ALL

SELECT
    'geolocation',
    COUNT(*)
FROM staging.geolocation

UNION ALL

SELECT
    'sellers',
    COUNT(*)
FROM staging.sellers

UNION ALL

SELECT
    'customers',
    COUNT(*)
FROM staging.customers

UNION ALL

SELECT
    'products',
    COUNT(*)
FROM staging.products

UNION ALL

SELECT
    'orders',
    COUNT(*)
FROM staging.orders

UNION ALL

SELECT
    'order_items',
    COUNT(*)
FROM staging.order_items

UNION ALL

SELECT
    'order_payments',
    COUNT(*)
FROM staging.order_payments

UNION ALL

SELECT
    'order_reviews',
    COUNT(*)
FROM staging.order_reviews;
```


| Table | Number of Records |
|---|---:|
| product_category_name_translation | 71 |
| sellers | 3,095 |
| products | 32,951 |
| order_payments | 103,886 |
| customers | 99,441 |
| order_reviews | 99,224 |
| orders | 99,441 |
| order_items | 112,650 |
| geolocation | 1,000,163 |




# Duplicate Records Validation

Duplicate checks were performed on the following tables in the `staging` schema:

- `customers` - `customer_id`
- `orders` - `order_id`
- `products` - `product_id`
- `sellers` - `seller_id`
- `order_items` - the composite key (`order_id`, `order_item_id`)

Each query grouped records by the relevant primary key or composite key and used `HAVING COUNT(*) > 1` to identify duplicates.

**Result:** No duplicate records were detected in any of the tested tables. All checked keys are unique within their respective tables, indicating that the data passed these duplicate validation checks.

**SQL validation method:** `GROUP BY` and `HAVING COUNT(*) > 1`.


# NULL Value Analysis
The following areas are examined:
- **Order delivery dates:** Missing customer delivery dates, carrier handover dates, and estimated delivery dates.
- **Customer and seller location data:** Missing postal code prefixes, city names, and state information.
- **Product data:** Missing product category information.
- **Customer reviews:** Missing review scores, comment titles, comment messages, and review timestamps.
- **Order approval:** Missing order approval timestamps.

```sql
SELECT
    'orders' AS table_name,
    'order_delivered_customer_date' AS column_name,
    COUNT(*) FILTER (
        WHERE order_delivered_customer_date IS NULL
    ) AS missing_values,
    ROUND(
        100.0 * COUNT(*) FILTER (
            WHERE order_delivered_customer_date IS NULL
        ) / NULLIF(COUNT(*), 0),
        2
    ) AS missing_percentage
FROM staging.orders;
```
| table_name | column_name | missing_values | missing_percentage |
|---|---|---:|---:|
| customers | customer_city | 0 | 0.00 |
| customers | customer_zip_code_prefix | 0 | 0.00 |
| customers | customer_state | 0 | 0.00 |
| order_reviews | review_answer_timestamp | 0 | 0.00 |
| order_reviews | review_creation_date | 0 | 0.00 |
| order_reviews | review_score | 0 | 0.00 |
| orders | order_estimated_delivery_date | 0 | 0.00 |
| sellers | seller_state | 0 | 0.00 |
| sellers | seller_city | 0 | 0.00 |
| sellers | seller_zip_code_prefix | 0 | 0.00 |
| orders | order_approved_at | 160 | 0.16 |
| products | product_category_name | 610 | 1.85 |
| orders | order_delivered_carrier_date | 1783 | 1.79 |
| orders | order_delivered_customer_date | 2965 | 2.98 |
| order_reviews | review_comment_message | 58247 | 58.70 |
| order_reviews | review_comment_title | 87656 | 88.34 |