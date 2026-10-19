---
project: FOMS
type: model
php_file: vendor/joeybab3/foms/src/OrderSource/Storefront.php
---
Back to [[JLab OMS (FOMS)]]

FOMS Storefronts operate on a given [[FOMS OrderSource|OrderSource]] and filter down orders. These storefronts are essentially filters such as date, tag, or source storefront from which to import in one operation. This may affect which [[FOMS NetSuiteCustomer|NetSuiteCustomer]] the order is imported to.

Like Order Sources, Storefronts have a set of [[FOMS OrderDestination|OrderDestinations]] which they are allowed to submit to. 
# Create Table Syntax
```sql
CREATE TABLE IF NOT EXISTS Storefronts (
    id INT AUTO_INCREMENT PRIMARY KEY,
    handle VARCHAR(255) UNIQUE NOT NULL,
    name VARCHAR(255) NOT NULL,
    filters JSON,
    config JSON,
    destinations JSON,
    order_source_handle VARCHAR(255) NOT NULL,
    enabled BOOLEAN DEFAULT TRUE
)
```

