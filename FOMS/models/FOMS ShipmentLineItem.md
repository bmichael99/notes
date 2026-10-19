---
project: FOMS
type: model
php_file: vendor/joeybab3/foms/src/Shipment/ShipmentLineItem.php
---
Back to [[JLab OMS (FOMS)]]

ShipmentLineItems hold information about individual line items within [[FOMS Shipment|Shipments]]. They signify partial or full shipments on a [[FOMS FulfillmentOrderLineItem|FulfillmentOrderLineItem]] and link back to their [[FOMS FulfillmentOrder|FulfillmentOrder]] via the `fo_id` column.
# Create Table Syntax
```sql
CREATE TABLE `ShipmentLineItems` (
  `id` int NOT NULL AUTO_INCREMENT,
  `fo_id` int NOT NULL,
  `shipment_id` int NOT NULL,
  `line_item_id` int DEFAULT NULL,
  `quantity` int DEFAULT NULL,
  `sku` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
```
