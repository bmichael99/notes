---
project: FOMS
type: controller
file_path: /var/www/html/ecomm/cron/foms/import_shipments.php
log_file_path: /var/www/html/ecomm/cron/foms/logs/fulfill.log
---
Previous step: [[FOMS Confirm Fulfillment Orders Controller]]
Next step: [[FOMS Invoice Shipments Controller]], [[FOMS Bulk Invoice Orders Controller]]

## Overview
The Import Shipments Controller imports [[FOMS Shipment|Shipments]] data from fulfillment [[FOMS OrderDestination|OrderDestinations]] for confirmed orders. This controller:

1. Retrieves fulfillment orders with status `confirmed` or `out_of_stock`
2. Calls destination to fetch shipment data (tracking numbers, shipping dates, etc.)
3. Creates Shipment and ShipmentLineItem records in the database
4. Updates order status based on shipment status
5. Records shipment import events

## Command Line Options

```bash
php import_shipments.php [options]

Options:
  -h, --help           Show help information
  -d, --debug          Enable debug mode
  -o, --order=ID       Process only a specific fulfillment order ID
```

### Examples
```bash
# Process all confirmed orders (up to 100,000)
php import_shipments.php

# Process with debug mode
php import_shipments.php --debug

# Process only a specific order
php import_shipments.php --order=12345

# Debug mode for specific order
php import_shipments.php --debug --order=12345
```

## Processing Flow

### 1. Order Discovery
- Finds fulfillment orders with status `confirmed` or `out_of_stock`
- Supports filtering to a specific order ID using the `--order` option
- Processes up to 100,000 orders by default

### 2. Shipment Import
For each fulfillment order:
- Validates destination exists and is valid
- For external destinations, requires destination URL
- Calls `OrderDestination::importShipments()` to fetch shipment data
- Creates Shipment records with tracking information
- Creates ShipmentLineItem records linking shipments to fulfillment order line items

### 3. Error Handling
- Skips orders without destination URLs (unless external destination)
- Logs `import_shipment_error` event and updates order status message
- Updates fulfillment order status based on shipment import results

## Related Components

### Models
- [[FOMS FulfillmentOrder]]
- [[FOMS Shipment]]
- [[FOMS ShipmentLineItem]]
- [[FOMS OrderDestination]]

### Events
- [[FOMS Event]]
- `import_shipment_error`: Logged when shipment import fails

### Repositories
- `FulfillmentOrderRepository::findByStatus(['confirmed', 'out_of_stock'])`
- `ShipmentsRepository::findByFulfillmentOrderId()`
- `OrderDestinationRepository`

### Destinations
- `AmazonDestination::importShipments()`
- `TaggDestination::importShipments()`
- `NetSuiteDestination::importShipments()`
- `ExternalDestination::importShipments()`

## Dependencies
- EventService: For logging shipment import events
- OrderMetrics: For execution time and order age tracking
- Order Destinations: Destination implementations for various 3PLs