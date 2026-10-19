---
project: FOMS
type: model
php_file: vendor/joeybab3/foms/src/FulfillmentOrder/FulfillmentOrder.php
---
Back to [[JLab OMS (FOMS)]]

This is the main [[JLab OMS (FOMS)|FOMS]] FulfillmentOrder table. It holds general data about fulfillment orders. FulfillmentOrders may have 1 or many [[FOMS FulfillmentOrderLineItem|LineItems]]. They are imported from an [[FOMS OrderSource|OrderSource]] and sent to an [[FOMS OrderDestination|OrderDestination]]. Once shipped they will have one or more [[FOMS Shipment|Shipments]], which will be invoiced and create a [[FOMS NetSuiteInfo|NetSuiteInfo]] record.

At import, FulfillmentOrders are mapped to a [[FOMS NetSuiteCustomer|NetSuite Customer]] in the customer mapper `Customer.php`.
# Lifecycle
1. `pending` ([[FOMS Import Fulfillment Orders Controller]])
2. `submitted` ([[FOMS Submit Fulfillment Orders Controller]])
3. `confirmed` ([[FOMS Confirm Fulfillment Orders Controller]])
4. `shipped` ([[FOMS Import Shipments Controller]])
5. `completed`([[FOMS Invoice Shipments Controller]])
Other statuses, resolved via `fo.php` FulfillmentOrder management portal
- `on_hold`
- `error`
- `out_of_stock`
- `cancelled`
# Create Table Syntax
```sql
CREATE TABLE IF NOT EXISTS FulfillmentOrder (
    id INT AUTO_INCREMENT PRIMARY KEY,
    order_id VARCHAR(32),
    order_name VARCHAR(255),
    order_source VARCHAR(255),
    customer VARCHAR(255),
    destination VARCHAR(255),
    destination_url VARCHAR(255),
    destination_order_id VARCHAR(255),
    status_handle VARCHAR(255),
    status_message VARCHAR(255),
    created_date DATETIME,
    updated_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    closed_date TIMESTAMP,
    gift_message TEXT,
    customizations TEXT,
    shipping_speed VARCHAR(255),
    attributes JSON,
    KEY idx_status_handle (status_handle),
    KEY idx_order_id (order_id),
    KEY idx_order_name (order_name),
    KEY idx_customer (customer),
    KEY idx_order_source (order_source),
    KEY idx_destination (destination),
    KEY idx_destination_order_id (destination_order_id),
    KEY idx_created_date (created_date),
    KEY idx_updated_date (updated_date),
    KEY idx_customer_status (customer, status_handle),
    KEY idx_order_name_destination (order_name, destination),
    KEY idx_destination_status (destination, status_handle),
    FULLTEXT KEY ft_search (order_id, order_name, destination_order_id)
)
```
# Queries
Select all FO's that have all [[FOMS FulfillmentOrderLineItem|FulfillmentOrderLineItems]] invoiced but the status is not completed.
```sql
SELECT fo.*
FROM FulfillmentOrder fo
WHERE fo.status_handle != 'completed'
AND NOT EXISTS (
    SELECT 1
    FROM FulfillmentOrderLineItems fol
    WHERE fol.fo_id = fo.id
    AND fol.status != 'invoiced'
);
```

Close out of FulfillmentOrders where all line items are invoiced.
```sql
UPDATE FulfillmentOrder fo
SET fo.status_handle = 'completed'
WHERE fo.status_handle != 'completed'
AND NOT EXISTS (
    SELECT 1
    FROM FulfillmentOrderLineItems fol
    WHERE fol.fo_id = fo.id
    AND fol.status != 'invoiced'
);
```

Find the total amount of orders/units for a given event type
```sql
SELECT 
    fo.destination,
    e.event_handle,
    COUNT(DISTINCT e.entity_id) AS event_count,
    SUM(line_items.quantity) AS unit_count
FROM Events e
INNER JOIN FulfillmentOrder fo ON e.entity_id = fo.id
LEFT JOIN (
    SELECT 
        fo_id,
        SUM(quantity) AS quantity
    FROM FulfillmentOrderLineItems
    GROUP BY fo_id
) AS line_items ON line_items.fo_id = fo.id
WHERE e.entity_type = 'fulfillment_order'
  AND e.event_handle IN ('confirmed', 'submitted', 'shipped')
  AND DATE(e.timestamp) = CURDATE()
GROUP BY fo.destination, e.event_handle
ORDER BY fo.destination, e.event_handle;
```

Order open/submitted orders by latest shipment date
```sql
SELECT * FROM `FulfillmentOrder`
WHERE status_handle = "confirmed" 
AND `attributes`->>'$.estimated_ship_date' IS NOT NULL
AND `attributes`->>'$.estimated_ship_date' != ''
ORDER BY `attributes`->>'$.estimated_ship_date' DESC;
```

Get order and unit count by month:
```sql
SELECT
    DATE_FORMAT(fo.created_date, '%Y-%m') AS month,
    COUNT(*) AS order_count,
    SUM(COALESCE(line_items.quantity, 0)) AS unit_count
FROM FulfillmentOrder fo
LEFT JOIN (
    SELECT
        fo_id,
        SUM(quantity) AS quantity
    FROM FulfillmentOrderLineItems
    GROUP BY fo_id
) AS line_items ON line_items.fo_id = fo.id
WHERE fo.created_date IS NOT NULL
  AND fo.status_handle != 'cancelled'
  AND fo.destination = "fba_us"
GROUP BY DATE_FORMAT(fo.created_date, '%Y-%m')
ORDER BY month DESC;
```

Get all orders from a date with unit_count > 1
```sql
SELECT
  COUNT(*) AS orders,
  COALESCE(SUM(t.unit_count), 0) AS total_units
FROM FulfillmentOrder fo
JOIN (
  SELECT fo_id, SUM(quantity) AS unit_count
  FROM FulfillmentOrderLineItems
  GROUP BY fo_id
) t ON t.fo_id = fo.id
WHERE fo.destination = 'logiwa_phoenix'
  AND fo.created_date >= '2026-05-21 00:00:00'
  AND fo.status_handle <> 'cancelled'
  AND t.unit_count > 1;
```