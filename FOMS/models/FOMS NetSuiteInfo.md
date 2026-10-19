---
project: FOMS
type: model
php_file: vendor/joeybab3/foms/src/NetSuite/NetSuiteInfo.php
---
Back to [[JLab OMS (FOMS)]]

This is the FOMS NetSuiteInfo table. It holds information about NetSuite transactions related to [[FOMS FulfillmentOrder|FulfillmentOrders]] and [[FOMS Shipment|Shipments]].
# Create Table Syntax
```sql
CREATE TABLE `NetSuiteInfo` (
  `id` int NOT NULL AUTO_INCREMENT,
  `fo_id` int NOT NULL,
  `shipment_id` int NOT NULL,
  `transaction_name` varchar(255) DEFAULT NULL,
  `netsuite_url` varchar(255) DEFAULT NULL,
  `date` timestamp NULL DEFAULT NULL,
  `netsuite_internal_id` int DEFAULT NULL,
  `status` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `idx_fo_id` (`fo_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
```
# Queries