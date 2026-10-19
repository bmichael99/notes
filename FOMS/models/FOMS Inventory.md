---
project: FOMS
type: model
php_file: vendor/joeybab3/foms/src/Inventory/Inventory.php
---
Back to [[JLab OMS (FOMS)]]

Inventory levels for different [[FOMS OrderDestination|OrderDestinations]]. Inventory Levels are pushed to [[FOMS OrderSource|OrderSources]] via [[FOMS InventoryFeed|InventoryFeeds]].
# Create Table Syntax
``` sql
CREATE TABLE `Inventory` (
  `id` int NOT NULL AUTO_INCREMENT,
  `sku` varchar(256) NOT NULL,
  `location_alias_sku` varchar(256) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL,
  `available` int NOT NULL,
  `inbound_inventory` int NOT NULL,
  `inbound_working` int NOT NULL,
  `processing` int NOT NULL,
  `researching` int NOT NULL,
  `location` varchar(256) NOT NULL,
  `updated_at` timestamp NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
```
# Queries