---
project: "[[JLab OMS (FOMS)]]"
type: controller
file_path: /var/www/html/ecomm/cron/foms/invoice_shipments.php
---
Previous step: [[FOMS Import Shipments Controller]]
## Overview
The Invoice Orders Controller processes pending shipments and creates invoices and credit memos in NetSuite. This controller:
1. Finds pending shipments with status `pending`
2. Creates invoices in NetSuite for each shipment
3. Creates credit memos for price discounts (when MSRP > price)
4. Updates shipment status to `invoiced` and line item status to `invoiced`
5. Marks fulfillment orders as `completed` when all line items are invoiced
For some customers, they utilize the [[FOMS Bulk Invoice Orders Controller]] rather than this one.
## Command Line Options
```bash
php invoice_shipments.php [options]

Options:
  -o, --order=ID       Process only a specific fulfillment order ID
  -d, --debug          Enable debug mode for verbose output
  -h, --help           Show help information
```
### Examples
```bash
# Process all pending shipments
php invoice_shipments.php

# Process with debug output
php invoice_shipments.php --debug

# Process only a specific order
php invoice_shipments.php --order=12345
```
## Processing Flow

### 1. Shipment Discovery
- Finds all shipments with status `pending`
- Supports filtering to a specific fulfillment order ID using the `--order` option
### 2. Invoice Processing
For each shipment:
- Gets the associated fulfillment order
- Calls `NetSuiteAdapter::processShipmentInvoice()` to create invoice
- Creates NetSuiteInfo record linking shipment to NetSuite transaction
- If discounts exist (MSRP > price), creates credit memo
- Updates all line items in shipment to `invoiced` status
### 3. Status Management
- Updates shipment status to `invoiced`
- Checks if all line items in fulfillment order are invoiced
- If all invoiced, sets fulfillment order status to `completed` and sets closed date
### 4. Error Handling
- Sets shipment status to `error` and logs error message
- Handles shipments skipped for bulk invoicing (keeps pending)
- Prevents duplicate NetSuiteInfo records for same invoice
## Related Components
### Events
- [[FOMS Event]]
- Invoice creation events logged by NetSuite adapter