---
project: FOMS
type: controller
file_path: /var/www/html/ecomm/cron/foms/update_inventory.php
---
# Overview
The Update Inventory Controller synchronizes [[FOMS Inventory]] quantities from internal inventory sources to external storefronts (Shopify, Walmart, ShipStation, etc.). This controller:
1. Finds enabled [[FOMS InventoryFeed|InventoryFeeds]] or a specific feed by handle
2. Collects inventory quantities from configured source locations
3. Sums available inventory across multiple locations per SKU
4. Applies minimum inventory thresholds (if configured)
5. Pushes updated inventory quantities to external storefronts via feed-specific implementations
# Command Line Options
```bash
php update_inventory.php [options]

Options:
  -s, --sku=SKU          Update inventory for a specific SKU only
  -h, --handle=HANDLE    Update inventory for a specific feed handle only
  -d, --debug            Enable debug mode for verbose output
```
## Examples
```bash
# Update all enabled inventory feeds
php update_inventory.php

# Update with debug output
php update_inventory.php --debug

# Update a specific SKU across all feeds
php update_inventory.php --sku=ABC123

# Update a specific feed handle
php update_inventory.php --handle=shopify_main

# Update a specific SKU for a specific feed
php update_inventory.php --sku=ABC123 --handle=shopify_main
```
## Processing Flow
### 1. Feed Selection
- If no handle is specified, finds all enabled inventory feeds using `InventoryFeedRepository::runAll()`
- If handle is specified, finds the specific feed using `InventoryFeedRepository::findByHandle()`
- Ensures feed exists and is enabled before processing
### 2. Inventory Aggregation
For each inventory feed:
- Gets inventory from all configured source locations
- Groups inventory by SKU across all source locations
- Sums available quantities for each SKU
- If `--sku` option is provided, only processes that specific SKU
### 3. Threshold Processing
- If feed has `minimum_inventory` config, quantities below threshold are set to 0
- If target SKU has no inventory found, creates entry with 0 quantity
### 4. Feed Update
- Calls `InventoryFeed::updateInventory()` which is implemented per feed type:
	  - Shopify: Updates inventory via Shopify API
	  - Walmart: Updates inventory via Walmart API
	  - ShipStation: Updates inventory via ShipStation API
- Records success/failure status for each feed
### Feed Types
- ShopifyInventoryFeed: Updates Shopify storefront inventory
- WalmartInventoryFeed: Updates Walmart marketplace inventory
- ShipStationInventoryFeed: Updates ShipStation inventory
## Dependencies
- InventoryFeedRepository: For managing inventory feed configurations
- InventoryRepository: For retrieving inventory from source locations
