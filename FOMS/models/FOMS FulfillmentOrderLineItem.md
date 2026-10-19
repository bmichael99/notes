---
project: FOMS
type: model
php_file: vendor/joeybab3/foms/src/FulfillmentOrder/FulfillmentOrderLineItem.php
---
Back to [[JLab OMS (FOMS)]]

This is the FOMS FulfillmentOrderLineItems table. It holds information about individual line items within a [[FOMS FulfillmentOrder|FulfillmentOrder]]. These line items are later used to create a [[FOMS Shipment|Shipment]], and are referenced in [[FOMS ShipmentLineItem|ShipmentLineItems]].
# Create Table Syntax
```sql
CREATE TABLE `FulfillmentOrderLineItems` (
  `id` int NOT NULL AUTO_INCREMENT,
  `fo_id` int NOT NULL,
  `source_line_item_id` varchar(255) DEFAULT NULL,
  `source_sku` varchar(255) DEFAULT NULL,
  `destination_line_item_id` varchar(255) DEFAULT NULL,
  `destination_sku` varchar(255) DEFAULT NULL,
  `quantity` int DEFAULT NULL,
  `price` float DEFAULT NULL,
  `msrp_price` float DEFAULT NULL,
  `currency` varchar(255) DEFAULT NULL,
  `status` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `idx_fo_id` (`fo_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
```

# Queries
Select Line Items that are apart of [[FOMS Shipment|Shipments]] that have already been invoiced.
```sql
SELECT fol.*
FROM FulfillmentOrderLineItems fol
	INNER JOIN ShipmentLineItems sli ON fol.id = sli.line_item_id
	INNER JOIN Shipments s ON sli.shipment_id = s.id
WHERE fol.status != 'invoiced'
AND s.status = 'invoiced';
```

Set fulfillment order line items to status `invoiced` where there exists `invoiced` shipments containing the line items with the status `invoiced`.
```sql
UPDATE FulfillmentOrderLineItems fol
	INNER JOIN ShipmentLineItems sli ON fol.id = sli.line_item_id
	INNER JOIN Shipments s ON sli.shipment_id = s.id
SET fol.status = 'invoiced'
WHERE fol.status != 'invoiced'
AND s.status = 'invoiced';
```