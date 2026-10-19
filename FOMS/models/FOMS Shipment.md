---
project: FOMS
type: model
php_file: vendor/joeybab3/foms/src/Shipment/Shipment.php
---
Back to [[JLab OMS (FOMS)]]

Shipments hold information about fulfillments created for [[FOMS FulfillmentOrder|FulfillmentOrders]] at the [[FOMS OrderDestination|OrderDestination]]. Shipments are pushed back to [[FOMS OrderSource|OrderSources]].

Shipments may have one or more [[FOMS ShipmentLineItem|ShipmentLineItems]].

Once invoiced, Shipments will have one or more attached [[FOMS NetSuiteInfo|NetSuiteInfo]] records corresponding to Sales Orders, Invoices, or Credit Memos.
# Create Table Syntax
```sql
CREATE TABLE `Shipments` (
  `id` int NOT NULL AUTO_INCREMENT,
  `fo_id` int NOT NULL,
  `vendor_shipment_id` varchar(255) DEFAULT NULL,
  `tracking_number` varchar(255) DEFAULT NULL,
  `tracking_company` varchar(255) DEFAULT NULL,
  `tracking_url` varchar(255) DEFAULT NULL,
  `status` varchar(255) DEFAULT NULL,
  `shipped_date` datetime NOT NULL,
  PRIMARY KEY (`id`),
  KEY `idx_fo_id` (`fo_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
```
# Queries
Select invoiced shipments that are not marked as such, i.e. there exists a [[FOMS NetSuiteInfo|NetSuiteInfo]] that references the shipment.
```sql
SELECT DISTINCT s.*
FROM Shipments s
	INNER JOIN NetSuiteInfo nsi ON s.id = nsi.shipment_id
WHERE s.status != 'invoiced';
```

Update Shipments to status `invoiced` if there exists a linked [[FOMS NetSuiteInfo|NetSuiteInfo]] record.
```sql
UPDATE Shipments s
	INNER JOIN NetSuiteInfo nsi ON s.id = nsi.shipment_id
SET s.status = 'invoiced'
WHERE s.status != 'invoiced';
```

Find duplicated shipments (tracking number and [[FOMS FulfillmentOrder|FulfillmentOrder]] id are the same).
```sql
SELECT s.*
FROM Shipments s
INNER JOIN (
    SELECT fo_id, tracking_number
    FROM Shipments
    WHERE tracking_number IS NOT NULL 
      AND tracking_number != ''
    GROUP BY fo_id, tracking_number
    HAVING COUNT(*) > 1
) duplicates 
ON s.fo_id = duplicates.fo_id 
AND s.tracking_number = duplicates.tracking_number
ORDER BY s.fo_id, s.tracking_number, s.id;
```

Find count of yet-to-be-checked shipments.
```sql
SELECT count(id) FROM Shipments 
WHERE (
	updated_date IS NULL 
	OR updated_date < DATE_SUB(NOW(), INTERVAL 1 DAY) 
	AND carrier_status NOT IN (
		'DELIVERED', 'UNDELIVERABLE', 'RETURNED', 'unknown','not found'
	)
) OR (
	carrier_status IN ('pending')
)
```

Get dollar totals per day of uninvoiced shipments.
```sql
SELECT
    DATE(COALESCE(s.shipped_date, s.updated_date)) AS `day`,
    ROUND(
        SUM(
            sli.quantity * GREATEST(
                IFNULL(foli.msrp_price, 0),
                IFNULL(foli.price, 0)
            )
        ),
        2
    ) AS total_dollar_value
FROM Shipments s
INNER JOIN ShipmentLineItems sli
    ON s.id = sli.shipment_id
INNER JOIN FulfillmentOrderLineItems foli
    ON sli.line_item_id = foli.id
WHERE s.status IN ('pending', 'error')
GROUP BY `day`
ORDER BY `day`;
```

Get dollar totals per day for a customer
```sql
SELECT
    DATE(s.shipped_date) AS ship_date,
    COUNT(DISTINCT fo.id) AS order_count,
    COUNT(DISTINCT s.id) AS shipment_count,
    SUM(sli.quantity) AS units_shipped,
    ROUND(SUM(foli.price * sli.quantity), 2) AS total_sold_price
FROM Shipments s
JOIN FulfillmentOrder fo ON s.fo_id = fo.id
JOIN ShipmentLineItems sli ON s.id = sli.shipment_id
JOIN FulfillmentOrderLineItems foli ON sli.line_item_id = foli.id
WHERE fo.order_source = 'tiktok'
  AND s.status = 'pending'
  AND s.shipped_date IS NOT NULL
GROUP BY DATE(s.shipped_date)
ORDER BY ship_date ASC;
```