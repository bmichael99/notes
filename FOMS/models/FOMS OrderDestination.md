---
project: FOMS
type: model
php_file: vendor/joeybab3/foms/src/OrderDestination/OrderDestination.php
---
Back to [[JLab OMS (FOMS)]]

Order destinations are destinations where [[FOMS FulfillmentOrder|FulfillmentOrders]] can be sent. Much like [[FOMS OrderSource|OrderSources]], they have a storefront column that is not related to [[FOMS Storefront|FOMS Storefronts]].

Destinations provide [[FOMS Inventory|InventoryLevels]] that push to [[FOMS OrderSource|OrderSources]] via [[FOMS InventoryFeed|InventoryFeeds]].
# Create Table Syntax
```sql
CREATE TABLE `OrderDestinations` (
  `id` int NOT NULL AUTO_INCREMENT,
  `handle` varchar(255) NOT NULL,
  `type` varchar(255) NOT NULL,
  `storefront` varchar(255) NOT NULL,
  `enabled` tinyint(1) DEFAULT '1',
  `name` varchar(255) DEFAULT NULL,
  `config` json,
  `netsuite_location_id` int DEFAULT '229',
  `attributes` json,
  PRIMARY KEY (`id`),
  UNIQUE KEY `handle` (`handle`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
```
# OrderDestination Types
- [[FBA Warehouse|FBA/MCF/Amazon Destination]]
- [[Phoenix Warehouse|Logiwa/Phoenix Warehouse]]
# Queries
## List line item quantity at a destination by sku and count
```sql
SELECT
    foli.destination_sku AS sku,
    SUM(foli.quantity) AS total_quantity
FROM
    FulfillmentOrder fo
JOIN
    FulfillmentOrderLineItems foli ON fo.id = foli.fo_id
WHERE
    fo.destination = 'logiwa_phoenix'
    AND fo.status_handle IN ('pending', 'submitted', 'confirmed', 'error', 'on_hold', 'out_of_stock')
GROUP BY
    foli.destination_sku
ORDER BY
    total_quantity DESC;
```
## List line item quantity at a destination by customer
```sql
SELECT
    fo.customer,
    SUM(foli.quantity) AS total_items
FROM
    FulfillmentOrder fo
JOIN
    FulfillmentOrderLineItems foli ON fo.id = foli.fo_id
WHERE
    fo.destination = 'logiwa_phoenix'
    AND fo.status_handle IN ('pending', 'submitted', 'confirmed', 'error', 'on_hold', 'out_of_stock')
GROUP BY
    fo.customer
ORDER BY
    total_items DESC;
```
## List line item quantity at a destination by date
```sql
SELECT
    DATE(fo.created_date) AS order_date,
    SUM(foli.quantity) AS total_items
FROM
    FulfillmentOrder fo
JOIN
    FulfillmentOrderLineItems foli ON fo.id = foli.fo_id
WHERE
    fo.destination = 'logiwa_phoenix'
    AND fo.status_handle IN ('pending', 'submitted', 'confirmed', 'error', 'on_hold', 'out_of_stock')
GROUP BY
    DATE(fo.created_date)
ORDER BY
    order_date DESC;
```
## List line item quantity at a destination by order item quantity
```sql
SELECT
    t.total_quantity,
    COUNT(*) AS order_count
FROM (
    SELECT
        fo.id,
        SUM(foli.quantity) AS total_quantity
    FROM
        FulfillmentOrder fo
    JOIN
        FulfillmentOrderLineItems foli ON fo.id = foli.fo_id
    WHERE
        fo.destination = 'logiwa_phoenix'
        AND fo.status_handle IN ('pending', 'submitted', 'confirmed', 'error', 'on_hold', 'out_of_stock')
    GROUP BY
        fo.id
) AS t
GROUP BY
    t.total_quantity
ORDER BY
    t.total_quantity ASC;
```
## List single unit orders by customer at a destination
```sql
SELECT
    fo.customer AS retailer,
    foli.destination_sku AS sku,
    COUNT(DISTINCT fo.id) AS order_count,
    SUM(foli.quantity) AS total_quantity
FROM
    FulfillmentOrder fo
JOIN
    FulfillmentOrderLineItems foli ON fo.id = foli.fo_id
JOIN (
    SELECT
        fo.id
    FROM
        FulfillmentOrder fo
    JOIN
        FulfillmentOrderLineItems foli ON fo.id = foli.fo_id
    WHERE
        fo.destination = 'logiwa_phoenix'
        AND fo.status_handle IN ('pending', 'submitted', 'confirmed', 'error', 'on_hold', 'out_of_stock')
    GROUP BY
        fo.id
    HAVING
        SUM(foli.quantity) = 1
) t ON t.id = fo.id
WHERE
    fo.destination = 'logiwa_phoenix'
    AND fo.status_handle IN ('pending', 'submitted', 'confirmed', 'error', 'on_hold', 'out_of_stock')
GROUP BY
    fo.customer,
    foli.destination_sku
ORDER BY
    total_quantity DESC;
```