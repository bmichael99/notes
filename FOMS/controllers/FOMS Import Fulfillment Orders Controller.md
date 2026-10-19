---
project: FOMS
type: controller
file_path: /var/www/html/ecomm/cron/foms/import_fulfillment_orders.php
log_file_path: /var/www/html/ecomm/cron/foms/logs/import.log
---
Back to [[JLab OMS (FOMS)]]

Next step: [[FOMS Submit Fulfillment Orders Controller]]
## Overview
The Import Fulfillment Orders Controller is the entry point for the FOMS order processing pipeline. It imports [[FOMS FulfillmentOrder|FulfillmentOrders]] from external order sources (Shopify, ShipStation, Walmart) and creates FulfillmentOrder records in the database. This controller handles:
1. Fetches orders from configured [[FOMS OrderSource|OrderSources]] and [[FOMS Storefront|Storefronts]]
2. Applies routing rules to transform and route orders
3. Translates source SKUs to destination SKUs based on customer and destination
4. Verifies inventory availability at destinations
5. Creates FulfillmentOrder, FulfillmentOrderLineItem, and ShippingInfo records
6. Acknowledges orders in source systems after successful import
## Command Line Options
```bash
php import_fulfillment_orders.php [options]

Options:
  -s, --storefront       Process only a specific storefront (by handle)
  -d, --debug            Enable debug mode for verbose output
  -dump-order-data       Dump order data to stdout and exit (for debugging)
  -dry-run               Run in dry-run mode (no data will be imported)
  -h, --help            Show help information
```
### Examples
```bash
# Process all storefronts
php import_fulfillment_orders.php

# Process only FBT storefront with debug output
php import_fulfillment_orders.php --storefront fbt --debug

# Dry run to test without importing
php import_fulfillment_orders.php --storefront shopify --dry-run

# Dump order data for debugging
php import_fulfillment_orders.php --storefront fbt -dump-order-data
```
## Processing Flow
### 1. Storefront Discovery
- Loads all enabled storefronts from the system
- Supports filtering to a specific storefront using the `--storefront` option
- Validates that requested storefronts exist
### 2. Router Setup
- Loads the `data_transformation` router with associated rules and criteria
- Scans and registers FOMS actions from the Actions directory
- Applies routing rules to transform order data
### 3. Order Fetching
For each storefront:
- Retrieves the associated order source (Shopify, ShipStation, etc.)
- Applies storefront filters to fetch matching orders
- Skips orders that have already been imported
### 4. Order Data Processing
For each order:
- Determines NetSuite customer based on order source and data
- Calculates distributed prices for line items based on MSRP ratios
- Applies data transformation router rules to modify order data
- Translates source SKUs to destination SKUs when needed
- Verifies inventory availability at destinations
### 5. Fulfillment Order Creation
- Groups line items by destination
- Creates separate FulfillmentOrder records for each destination
- Creates FulfillmentOrderLineItem records with translated SKUs and prices
- Creates ShippingInfo records with shipping address data
- Processes existing fulfillments if order was already fulfilled
### 6. Order Acknowledgment
- Acknowledges orders in the source system after successful import
- Handles retry logic for acknowledgment failures
## Related Components
### Models
- [[FOMS FulfillmentOrder]]
- [[FOMS FulfillmentOrderLineItem]]
- [[FOMS ShippingInfo]]
- [[FOMS OrderSource]]
- [[FOMS Storefront]]
- [[FOMS NetSuiteCustomer]]
- [[FOMS SkuTranslations]]
- [[FOMS Inventory]]
### Events
- [[FOMS Event]]
- `order_imported`: Logged when orders are successfully imported
- `line_item_added`: Logged when line items are created
- SKU translation events logged when SKUs are translated
### Repositories
- `FulfillmentOrderRepository::createFulfillmentOrderFromData()`
- `FulfillmentOrderLineItemsRepository`
- `ShippingInfoRepository`
- `OrderSourceRepository::getOrderSourceInstanceByHandle()`
- `SkuTranslationRepository::findBySourceSkuCustomerAndDestination()`
- `StorefrontRepository::findByHandle()`
### Routers
- `data_transformation`: Router that applies business rules to order data
- Uses `RuleRepository` and `CriteriaRepository` for rule evaluation
## Dependencies
- EventService: For logging import and translation events
- OrderMetrics: For execution time and order age tracking
- Inventory API: For MSRP lookup and inventory checking
- NetSuite Adapter: For customer ID lookup and import-time invoice creation
- Order Sources: Shopify, ShipStation, Walmart order source implementations