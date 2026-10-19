---
project: FOMS
type: model
php_file: vendor/joeybab3/foms/src/FulfillmentOrder/ShippingInfo.php
---
This is the FOMS ShippingInfo table. It holds shipping address and contact information for [[FOMS FulfillmentOrder|FulfillmentOrders]].
# Create Table Syntax
```sql
CREATE TABLE `ShippingInfo` (
  `id` int NOT NULL AUTO_INCREMENT,
  `fo_id` int NOT NULL,
  `first_name` varchar(255) DEFAULT NULL,
  `last_name` varchar(255) DEFAULT NULL,
  `company` varchar(255) DEFAULT NULL,
  `address1` varchar(255) DEFAULT NULL,
  `address2` varchar(255) DEFAULT NULL,
  `address3` varchar(255) DEFAULT NULL,
  `city` varchar(255) DEFAULT NULL,
  `province` varchar(255) DEFAULT NULL,
  `zip` varchar(255) DEFAULT NULL,
  `phone` varchar(255) DEFAULT NULL,
  `country_code` varchar(255) DEFAULT NULL,
  `email` varchar(255) DEFAULT NULL,
  `notes` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `idx_fo_id` (`fo_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
```
# Queries
Select FulfillmentOrders that are missing ShippingInfo records
```sql
SELECT fo.*
FROM FulfillmentOrder fo
	LEFT JOIN ShippingInfo si ON fo.id = si.fo_id
WHERE si.fo_id IS NULL;
```