---
project: FOMS
type: model
php_file: vendor/joeybab3/foms/src/InventoryFeed/InventoryFeed.php
---
Back to [[JLab OMS (FOMS)]]

This is the FOMS InventoryFeeds table. It holds configuration for different inventory feed sources. These feeds push [[FOMS Inventory|Inventory Levels]]  from their associated [[FOMS OrderDestination|OrderDestination]] to [[FOMS OrderSource|OrderSources]].

Feeds are usually referenced by handle such as in [[FOMS Update Inventory Controller]].
# Create Table Syntax
```sql
CREATE TABLE IF NOT EXISTS InventoryFeeds (
    id INT AUTO_INCREMENT PRIMARY KEY,
    handle VARCHAR(255) NOT NULL UNIQUE,
    type VARCHAR(50) NOT NULL,
    storefront VARCHAR(50) NOT NULL,
    enabled TINYINT(1) NOT NULL DEFAULT 1,
    sources JSON NOT NULL,
    name VARCHAR(255) NOT NULL,
    config JSON,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci
```
# Queries