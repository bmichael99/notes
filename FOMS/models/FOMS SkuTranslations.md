---
project: FOMS
type: model
php_file: vendor/joeybab3/foms/src/SkuTranslation/SkuTranslation.php
---
Back to [[JLab OMS (FOMS)]]

This is the [[JLab OMS (FOMS)|FOMS]] SkuTranslations table. It holds SKU translation mappings between different systems and [[FOMS NetSuiteCustomer|NetSuiteCustomers]].

The source and destination SKU fields match those of [[FOMS FulfillmentOrderLineItem|FulfilmentOrderLineItems]].

Sku translations are primarily used in the [[FOMS Import Fulfillment Orders Controller]] and the [[FOMS Invoice Shipments Controller]].
# Create Table Syntax
```sql
CREATE TABLE `SkuTranslations` (
  `id` int NOT NULL AUTO_INCREMENT,
  `source_sku` varchar(255) NOT NULL,
  `destination_sku` varchar(255) NOT NULL,
  `customer_handle` varchar(255) NOT NULL,
  `created_by` int NOT NULL,
  `destination` varchar(255) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
```