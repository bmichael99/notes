---
project: FOMS
type: controller
file_path: /var/www/html/ecomm/cron/foms/bulk_invoice_orders.php
---
Previous step: [[FOMS Import Shipments Controller]]

## Overview
The Bulk Invoice Orders Controller processes shipments for bulk-invoiced customers by creating consolidated invoices and credit memos in NetSuite. This is a bulk version of the [[FOMS Invoice Shipments Controller|Invoice Orders Controller]] that groups shipments by customer, destination, and date into single invoices.

This controller:
1. Retrieves customers marked for bulk invoicing
2. Groups pending shipments by customer, destination, and date
3. Creates single consolidated invoice in NetSuite for all shipments
4. Creates credit memos for discount items (when MSRP > price)
5. Updates shipment statuses to `invoiced` and creates NetSuiteInfo records

## Command Line Options

```bash
php bulk_invoice_orders.php [options]

Options:
  -d, --date=YYYY-MM-DD          Date to invoice orders for (default: yesterday)
  -c, --customer=HANDLE          Process only a specific customer (default: all bulk-invoiced customers)
  --dry-run                      Run in dry-run mode (no actual changes)
  --debug                        Enable debug mode for detailed output
  --external-id-prefix=PREFIX    External ID prefix for NetSuite (default: FOMS)
  -h, --help                     Show help information
```

### Examples
```bash
# Process yesterday's shipments for all bulk-invoiced customers
php bulk_invoice_orders.php

# Process specific date
php bulk_invoice_orders.php --date=2025-01-15

# Process specific customer
php bulk_invoice_orders.php --customer=tiktok --date=2025-01-15

# Dry run to test without creating invoices
php bulk_invoice_orders.php --date=2025-01-15 --dry-run --debug
```

## Processing Flow

### 1. Customer Discovery
- Finds all customers marked for bulk invoicing (`bulk_invoiced = true`)
- Supports filtering to a specific customer using the `--customer` option

### 2. Destination Processing
For each bulk-invoiced customer:
- Finds all unique destinations for the customer
- Groups shipments by destination and target date

### 3. Shipment Aggregation
For each customer/destination/date combination:
- Finds all pending shipments matching criteria
- Groups shipped quantities by SKU
- Calculates invoice prices and discount deltas (MSRP - price)
- Identifies items with positive delta (discounts)

### 4. Bulk Invoice Creation
- Calls `NetSuiteAdapter::processBulkInvoice()` to create consolidated invoice
- Creates NetSuiteInfo records for all shipments linked to invoice
- If discount items exist, creates bulk credit memo
- Updates all shipment statuses to `invoiced`

### 5. Error Handling
- Skips customers/destinations with no pending shipments
- Logs errors and continues with next destination
- Shows what would be processed without making changes

## Related Components

### Models
- [[FOMS Shipment]]
- [[FOMS ShipmentLineItem]]
- [[FOMS FulfillmentOrder]]
- [[FOMS FulfillmentOrderLineItem]]
- [[FOMS NetSuiteInfo]]
- [[FOMS NetSuiteCustomer]]

### Events
- [[FOMS Event]]
- Bulk invoice creation events logged by NetSuite adapter

### Repositories
- `NetSuiteCustomerRepository::findAll()`
- `NetSuiteInfoRepository::save()`
- `ShipmentsRepository`

### Services
- `NetSuiteAdapter::processBulkInvoice()`
- `NetSuiteAdapter::processBulkCreditMemo()`
- `NetSuiteAdapter::getBulkInvoiceResult()`
- `NetSuiteAdapter::getBulkCreditMemoResult()`

## Dependencies
- OrderMetrics: For execution time tracking
- NetSuite Adapter: For creating bulk invoices and credit memos in NetSuite
- Database: Direct SQL queries for shipment aggregation

## Notes
- Currently optimized for TikTok orders but works with any bulk-invoiced customer
- Processes one invoice per customer/destination/date combination
- Credit memos are created separately when discounts exist
- Shipment statuses are updated only after successful invoice creation