---
project: FOMS
type: controller
file_path: /var/www/html/ecomm/cron/foms/import_cancelation_requests.php
log_file_path: /var/www/html/ecomm/cron/foms/logs/cancelations.log
---
Previous step: [[FOMS Confirm Fulfillment Orders Controller]]
Next step: [[FOMS Invoice Shipments Controller]]
## Overview
The Import Cancelation Requests Controller is responsible for processing cancellation requests that originate from external order sources (primarily Shopify). This controller automates the handling of cancellation workflows by:
1. Detecting cancellation requests from configured order sources
2. Processing each request by attempting to cancel orders in both FOMS and destination systems
3. Accepting or rejecting requests in the source system based on processing success
4. Logging comprehensive audit events for all cancellation activities
## Command Line Options

```bash
php import_cancelation_requests.php [options]

Options:
  -d, --debug    Enable debug mode for verbose output
  -s, --source   Process only a specific order source (by handle)
  -h, --help     Show help information
```
### Examples
```bash
# Process all order sources
php import_cancelation_requests.php

# Process only Shopify orders with debug output
php import_cancelation_requests.php --source shopify --debug

# Debug mode for troubleshooting
php import_cancelation_requests.php -d
```
## Processing Flow

### 1. Source Discovery
- Loads all enabled order sources from the system
- Supports filtering to a specific source using the `--source` option
- Validates that requested sources exist and are enabled
### 2. Cancellation Request Detection
For each order source:
- Calls `fetchCancelationRequests()` to retrieve pending cancellation requests
- Currently supports Shopify order sources with fulfillment order cancellations
- Logs the number of requests found for each source
### 3. Order Processing
For each cancellation request:
- Converts Shopify fulfillment order IDs to FOMS format (`FO_{shopify_id}`)
- Retrieves the fulfillment order from FOMS database
- Records `shopify_cancel_requested` system event
### 4. Cancellation Execution
- Calls `fulfillmentOrderRepository->cancel()` with `createReplacement=false`
- For Shopify orders, calls `acceptCancellationRequest()`
- Increments cancellation counters and logs success
### 5. Error Handling
When cancellation fails:
- Sets FOMS order status to 'error' with failure message
- For Shopify orders, calls `rejectCancellationRequest()` with error details
- Increments error counters and logs detailed error information
## Related Components
### Models
- [[FOMS FulfillmentOrder]]
- [[FOMS OrderSource]]
### Events
Stored in [[FOMS Event]]
- shopify_cancel_requested: Logged when cancellation requests are detected
- fulfillment_order_cancelled_both: Logged when orders are successfully cancelled
### Repositories
- `FulfillmentOrderRepository::cancel()`
- `OrderSourceRepository::getOrderSourceInstanceByHandle()`
- `ShopifyOrderSource::fetchCancelationRequests()`
- `ShopifyOrderSource::acceptCancellationRequest()`
- `ShopifyOrderSource::rejectCancellationRequest()`
## Dependencies
- EventService: For logging cancellation request events
- OrderMetrics: For execution time tracking
- Order Sources: Shopify order source implementation required for cancellation handling
