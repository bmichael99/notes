---
project: FOMS
type: controller
file_path: /var/www/html/ecomm/cron/foms/import_confirmations.php
log_file_path: /var/www/html/ecomm/cron/foms/logs/confirm.log
---
Previous step: [[FOMS Submit Fulfillment Orders Controller]]
Next step: [[FOMS Import Shipments Controller]]

## Overview
The Confirm Fulfillment Orders Controller checks submitted fulfillment orders to confirm they have been accepted by their destination fulfillment partners. This controller:

1. Retrieves all fulfillment orders with status `submitted`
2. Calls destination to verify order was accepted
3. Updates the order source system with confirmation
4. Updates order status based on confirmation results
5. Records confirmation events for audit trail

## Command Line Options

```bash
php import_confirmations.php [options]

Options:
  -d, --debug    Enable debug mode for verbose output
  -h, --help     Show help information
```

### Examples
```bash
# Process all submitted orders
php import_confirmations.php

# Process with debug output
php import_confirmations.php --debug
```

## Processing Flow

### 1. Order Discovery
- Finds all fulfillment orders with status `submitted`
- Processes orders in sequence

### 2. Confirmation Check
For each fulfillment order:
- Calls `OrderDestination::confirm()` to verify order was accepted
- Retrieves destination order identifier for tracking
- Calls `OrderSource::confirmOrder()` to update source system

### 3. Status Management
- For external destinations, sets status to `confirmed` with confirmation message
- Logs `destination_confirm_not_found` event if order not found at destination
- Logs `destination_confirm_failed_automated` event on errors

## Related Components

### Models
- [[FOMS FulfillmentOrder]]
- [[FOMS OrderDestination]]
- [[FOMS OrderSource]]

### Events
- [[FOMS Event]]
- `destination_confirmed_automated`: Logged when orders are automatically confirmed
- `destination_confirm_not_found`: Logged when order not found at destination
- `destination_confirm_failed_automated`: Logged when confirmation check fails

### Repositories
- `FulfillmentOrderRepository::findByStatus('submitted')`
- `OrderDestinationRepository`
- `OrderSourceRepository`

### Destinations
- `AmazonDestination::confirm()`
- `TaggDestination::confirm()`
- `NetSuiteDestination::confirm()`
- `ExternalDestination::confirm()`

## Dependencies
- EventService: For logging confirmation events
- OrderMetrics: For execution time and order age tracking
- Order Destinations: Destination implementations for various 3PLs