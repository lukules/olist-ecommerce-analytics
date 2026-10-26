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
