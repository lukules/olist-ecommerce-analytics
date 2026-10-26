# Database Creation
```sql
CREATE DATABASE olist_ecommerce;
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
olist_ecommerce
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

#### Geolocation

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
