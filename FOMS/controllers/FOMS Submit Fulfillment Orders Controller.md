---
project: FOMS
type: controller
file_path: /var/www/html/ecomm/cron/foms/submit_fulfillment_orders.php
log_file_path: /var/www/html/ecomm/cron/foms/logs/submit.log
---
Previous step: [[FOMS Import Fulfillment Orders Controller]]
Next step: [[FOMS Confirm Fulfillment Orders Controller]]

## Overview
The Submit Fulfillment Orders Controller processes pending fulfillment orders and submits them to their configured destinations (3PLs, fulfillment partners). This controller:

1. Retrieves all fulfillment orders with status `pending`
2. Submits orders to their configured destination (Amazon, Tagg, NetSuite, etc.)
3. Updates the order source (Shopify, etc.) with submission status
4. Updates order status to `submitted` on success or `error` on failure
5. Records submission events for audit trail

## Command Line Options

```bash
php submit_fulfillment_orders.php [options]

Options:
  -d, --debug    Enable debug mode for verbose output
  -h, --help     Show help information
```

### Examples
```bash
# Process all pending orders
php submit_fulfillment_orders.php

# Process with debug output
php submit_fulfillment_orders.php --debug
```

## Processing Flow

### 1. Order Discovery
- Finds all fulfillment orders with status `pending`
- Processes orders in sequence

### 2. Order Submission
For each fulfillment order:
- Gets complete order data including line items and shipping info
- Validates that destination is set and destination instance exists
- Calls `OrderDestination::submit()` to submit order to fulfillment partner
- Calls `OrderSource::submitOrder()` to update source system

### 3. Status Management
- Updates order status to `submitted` (unless already `out_of_stock` or `error`)
- Sets status to `error` with error message
- Logs appropriate event (`submitted` or `submit_failed`)

## Related Components

### Models
- [[FOMS FulfillmentOrder]]
- [[FOMS OrderDestination]]
- [[FOMS OrderSource]]

### Events
- [[FOMS Event]]
- `submitted`: Logged when orders are successfully submitted
- `submit_failed`: Logged when submission fails

### Repositories
- `FulfillmentOrderRepository::findByStatus('pending')`
- `OrderDestinationRepository`
- `OrderSourceRepository`

### Destinations
- `AmazonDestination::submit()`
- `TaggDestination::submit()`
- `NetSuiteDestination::submit()`
- `ExternalDestination::submit()`
- `ManualDestination::submit()`

## Dependencies
- EventService: For logging submission events
- OrderMetrics: For execution time and order age tracking
- Order Destinations: Destination implementations for various 3PLs