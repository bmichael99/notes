---
project: FOMS
type: model
php_file: vendor/joeybab3/foms/src/OrderSource/OrderSource.php
---
Back to [[JLab OMS (FOMS)]]

Order Sources hold configuration for different order source systems. These are polled for [[FOMS FulfillmentOrder|FulfillmentOrders]], which are imported then submitted to [[FOMS OrderDestination|OrderDestinations]].

There may be one or more [[FOMS Storefront|Storefronts]] which filter out orders from an OrderSource. The storefront column in this table is unrelated to FOMS Storefronts.

Order Sources keep a json array of allowed [[FOMS OrderDestination|OrderDestinations]] which determines which Order Destinations these orders are allowed to be submitted to.
# Create Table Syntax
```sql
CREATE TABLE IF NOT EXISTS OrderSources (
    id INT AUTO_INCREMENT PRIMARY KEY,
    handle VARCHAR(255) UNIQUE NOT NULL,
    type VARCHAR(255) NOT NULL,
    storefront VARCHAR(255) NOT NULL,
    enabled BOOLEAN DEFAULT TRUE,
    destinations JSON,
    name VARCHAR(255),
    config JSON
)
```
