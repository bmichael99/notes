---
project: FOMS
type: model
php_file: vendor/joeybab3/foms/src/NetSuite/NetSuiteCustomer.php
---
Back to [[JLab OMS (FOMS)]]

NetSuiteCustomers hold information about netsuite customers, and are linked to by their handle from [[FOMS FulfillmentOrder|FulfillmentOrders]]' customer column.
# Create Table Syntax
```sql
CREATE TABLE IF NOT EXISTS NetSuiteCustomers (
    id INT AUTO_INCREMENT PRIMARY KEY,
    handle VARCHAR(255) NOT NULL UNIQUE,
    netsuite_internal_id INT,
    attributes JSON,
    bulk_invoiced BOOLEAN DEFAULT FALSE
)
```
# Queries