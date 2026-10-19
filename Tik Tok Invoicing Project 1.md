NETSUITE_DOCUMENT_LINES:

| Column               | Role                                                                                       |
| -------------------- | ------------------------------------------------------------------------------------------ |
| source_order_id      | The silver line, source here is tiktok, allows us to tie back netsuite data to source data |
| source_line_item_id  | The silver line, source here is tiktok, allows us to tie back netsuite data to source data |
| document_type        | SALES_ORDER, ITEM_FULFILLMENT, or INVOICE                                                  |
| netsuite_internal_id | The document and its line (orderLine on the fulfillment and invoice)                       |
| netsuite_line        | The document and its line (orderLine on the fulfillment and invoice)                       |
| quantity             | How much of this TikTok line is on that document line                                      |
| parent_netsuite_id   | The sales-order line the fulfillment or invoice was created from                           |
| parent_line          | The sales-order line the fulfillment or invoice was created from                           |
| `created_at`         |                                                                                            |
| `updated_at`         |                                                                                            |
|                      |                                                                                            |

NETSUITE_DOCUMENTS:

| Column                                  | Role                                                                                        |
| --------------------------------------- | ------------------------------------------------------------------------------------------- |
| `document_type`, `netsuite_internal_id` | The NetSuite record                                                                         |
| `location_id`, `tran_date`              | How the job groups the day's work                                                           |
| `parent_netsuite_id`                    | The sales order an item fulfillment came from, or the item fulfillment an invoice came from |
| `external_ref`                          | The `otherRefNum`, such as `tiktok_sales_orders_20260729`                                   |
| `status`                                | Open, closed                                                                                |
| `created_at`                            |                                                                                             |
| `updated_at`                            |                                                                                             |

## ALGORITHM:

**Notes:**
- Tik Tok's daily order data is the source of truth for the days sales order, not the days orders in FOMS
- The source of truth for a line items status is determined by the fulfillment_type
	- FULFILLMENT_BY_SELLER -> fullfillment order line items is the source of truth for status
	- FULFILLMENT_BY_TIKTOK -> tik tok order line item is the source of truth for status
- Decide how to handle timezones, during dev/testing this doesn't matter, but make sure this is determined before production.

timezone -> use UTC
prices -> ![[Pasted image 20261014155649.png]]

**Steps:**
1. Open a blank and open sales order for the current day in NETSUITE_DOCUMENTS, we will not actually have a document in Netsuite for sales orders.
2. Load daily order data once a day after the day has ended into a raw json table
	1. A days order data has to continue being collected until the "sales order" is closed. Loop through the open days, query data for each open day.
3. Transform and upsert the days tik tok order data into two tables:
	1. ORDER table, header level data that is shared across line items for that order
	2. ORDER_LINES table, Order line item level, there will be a quantity column, for tik tok this is always 1 and there can be multiple rows per seller SKU per order
		1. bundles need to have their own ORDER_LINES row per bundle item, for the line item id use format of s_{LINE-ITEM-ID}--{BUNDLE-ITEM-ID}. So if there are 2 items in a bundle, this is 1 line item in tik tok api, but we need to split this down into 2 rows on our end, following the line_item_id format shown above.
	3. CONSIDER: ?? "Sales order" level. a sales order document_type in NETSUITE_DOCUMENTS -> the entire days list of orders, a row per line item in NETSUITE_DOCUMENT_LINES for that days orders, either this or you just group by day to determine what is a days "sales order"
4. Reference or import Fulfillment order data from FOMS. A FO line item can be referenced by source_line_item_id and a FO can be referenced by order_id
	1. For fulfillment_type of "FULFILLMENT_BY_SELLER":
		1. Fulfillment order data is the source of truth for what has shipped
	2. For fulfillment_type of "FULFILLMENT_BY_TIKTOK"
		1. tik tok api data is the source of truth for what has shipped.
	3. For analyzing discrepancies, diff the status between tik tok order and what's stored in FOMS fulfillment order data 
5. Create an invoice PER "destination" (e.g. fba_us or logiwa_phoenix) for everything that has shipped that day, reference the correct source of truth based on fulfillment_type
	1. Create an invoice in Netsuite and store related data, line items are grouped by destination SKU. (one line item per destination SKU, sum quantity)
	2. Create a row in the NETSUITE_DOCUMENTS table for this invoice, with information like document_type (invoice), location_id (logiwa_phoenix), etc.
	3. Create rows in NETSUITE_DOCUMENT_LINES for mapping every line on the invoice to the source tik tok line. fulfillment order line can be tied to this by joining on columns source_line_item_id and order_id in FOMS.
		1. FOMs order line items table will provide our source and destination SKU's. The destination SKU from fulfillment order line is what ends up on the invoice when fulfillment_type is "FULFILLMENT_BY_SELLER". both fulfilled by tik tok and by seller will be in FOMS so we could just pull destination sku data from FOMS if we want but it doesn't matter, we can mix and match too, just make a decision.
6. in our UI, a "sales order" is "closed" if:
	1. every order line item for that day has been invoiced or cancelled.
		1. you determine a line item has been invoiced by joining with NETSUITE_DOCUMENT_LINES On tik tik order line id
		2. you determine a line item has been cancelled by:
			1. if FULFILLMENT_BY_SELLER -> reference fulfillment order line item
			2. if FULFILLMENT_BY_TIKTOK -> reference tik tok order line item
	2. You could have a script and summary table for calculating this once a day instead of needing to run the joins for that day every time the UI is loaded, if this becomes a performance issue.
		1. if using NETSUITE_DOCUMENTS for sales orders then status could be filled in there
7. orders and line items will need to have their status updated by the tik tok api until it has reached a "closed" status. Some discrepancies can be flagged by comparing these status's and what's stored in NETSUITE_DOCUMENT_LINES, as what's stored there we have already acted upon based on shipped. cancelled order status + a line in netsuite document lines = return or some other discrepancy. 



Create a level of visibility and audibility so we can see everything that's happening and what's being applied to what. May need to have a human checking off on certain things. use event logging and events table.


![[Pasted image 20261014150346.png]]


Silver script:
- Check if daily sales order in NETSUITE_DOCUMENTS has been created, if it hasn't been, create it.
- Loop through 

Bronze script: 
	Call tik tok api for the last days updated orders, you can query for created date or updated date, query by updated and just get all orders that have been updated in the last day:
		- Upsert on order_id, Create new row if its order_id doesn't exist, 
	-  insert_bronze_data():
		- create new row if there's not already a row with that order_id and update_time, silver will always upsert the most recent update_time. 
- 
	bronze table:
	- order_id
	- order_status
	- json
	- create_time (unix timestamp in the json)
	- update_time (unix timestamp in the json)
	- created_at (when we actually created this in the mysql table)



Do grouping by the rate aka "unit price", since we need to make sure that 


got it, we should have a way of surfacing issues like this in a UI so that a user can resolve an item_id if the issue arises, probably something like our log_event events table that we did for labeling, use composer 2.5 agents to research into that and see if that would be a good aproach here and we should handle this.

TO DO:
- build the functionality for auto closing sales orders
	- When all orders are at least IN-TRANSIT/cancelled I believe 
- Retrieve and perform ETL on tik tok remittances 
	- define and find relationship with existing orders, line items, sales order, invoices, credit memos, etc
	- 