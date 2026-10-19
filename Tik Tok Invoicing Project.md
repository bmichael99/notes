need to apply remittance to the correct order number under an invoice. order numbers currently not tied to an invoice.

when month is over, can no longer apply changes to an invoice.

how do we accurately apply deductions to the correct invoice. rather than oldest.

Human audit, here's what we're going to apply, what remittance to what invoice. What order numbers

what's the best approach for resolving.

multiple credit memos to a particular order.
- discount memo (credit memo) for discount, gross vs discount
- creddit memo for fees, credit memo
- credit memo for returns
- credit memo for returns that don't actually apply to this invoice, but because the other invoice is closed, it has to be applied here

we have all the orders that come through i n a day, we have a sales order for those, then we do the IF's for what's shipped that day, then the invoices are automatically generated over night.

Step 1: identifying from tik tok how do we find out which orders have come in for a particular day, list them as a sales order for a day, IF -> what shipped, x and y shipped from here, what's on the item fulfillment, orders must be on IF. every IF will generate an invoice. sales order not closed until all invoices are generated. remittance applies to these invoices on a specific order number.

Build SO, IF, Invoice day of? or after day is over?


we can build our own scripts for this, scott's were just a point of reference.

At the end of the day we push all the data into snowflake and build a SO. As items ship on that SO we build IF's for those shipped items, when items get cancelled we cancel them on the SO. At the end of the day when we close an IF we create the invoice ourselves instead of letting Netsuite do it, this allows us to get the Netsuite invoice internal ID straight back so we can connect an IF to an invoice.

We need to be able to connect all credit memo deductions to order number level of granularity. Order number being the order number we receive from tik tok. The relationship of order numbers and their deductions need to be stored in snowflake, we won't be able to have these within netsuite as a credit memo has its deductions summed by category. So for auditing purposes that data will be stored into snowflake and accounts receivable can pull that data from there.

Use historical data stored within snowflake to create an SO, once that SO is built -> build the IF and match with shipment data we have in FOMS, apply cancellations on the SO based on fulfillment information from FOMS. A single IF is generated based on the day and the destination/fulfillment(?).

One sales order per Location, foms.OrderDestinations, warehouse_id

Create credit memo on invoice
Create payment on invoice
- payment should balance out correctly

Considerations to keep in mind on deductions:
- ignore tik tok applied discounts, they eat those costs not us
- if shipping is balanced out you ignore it as well

We can do data loading and transforms in MySQL, but it has to be backed up in Snowflake.

[[Snowflake Examples]]


We need a table(cache?) for describing the relationship between an Order and its related Sales Order Line Item. Same thing for IF and Invoices, because a Sales Order will likely have multiple IF's and Invoices based on when products ship.
- IF's should probably start generating as soon as an SO is created, and generate IF's every day at the same time, some time after an SO generates for the day. We also automatically generate an Invoice as soon as we're finished creating and populating an IF.

NETSUITE_DOCUMENT_LINES:
- Each row is one TikTok line sitting on one NetSuite document line. This table represents a TikTok Order line and its relationship in Netsuite
	- Columns:

NETSUITE_DOCUMENT_LINES:

| Column               | Role                                                                 |
| -------------------- | -------------------------------------------------------------------- |
| source_order_id      | The silver line, source here is tiktok                               |
| source_line_item_id  | The silver line, source here is tiktok                               |
| document_type        | SALES_ORDER, ITEM_FULFILLMENT, or INVOICE                            |
| netsuite_internal_id | The document and its line (orderLine on the fulfillment and invoice) |
| netsuite_line        | The document and its line (orderLine on the fulfillment and invoice) |
| quantity             | How much of this TikTok line is on that document line                |
| parent_netsuite_id   | The sales-order line the fulfillment or invoice was created from     |
| parent_line          | The sales-order line the fulfillment or invoice was created from     |
| `created_at`         |                                                                      |
| `updated_at`         |                                                                      |
|                      |                                                                      |

NETSUITE_DOCUMENTS:

| Column                                  | Role                                                                                        |
| --------------------------------------- | ------------------------------------------------------------------------------------------- |
| `document_type`, `netsuite_internal_id` | The NetSuite record                                                                         |
| `location_id`, `tran_date`              | How the job groups the day's work                                                           |
| `parent_netsuite_id`                    | The sales order an item fulfillment came from, or the item fulfillment an invoice came from |
| `external_ref`                          | The `otherRefNum`, such as `tiktok_sales_orders_20260729`                                   |
| `status`                                | Open, closed, or failed                                                                     |
| `created_at`                            |                                                                                             |
| `updated_at`                            |                                                                                             |

## ALGORITHM:

Notes:
- Tik Tok's daily order data is the source of truth for the days sales order, not the days orders in FOMS
- The source of truth for a line items status is determined by the fulfillment_type
	- FULFILLMENT_BY_SELLER -> fullfillment order line items is the source of truth for status
	- FULFILLMENT_BY_TIKTOK -> tik tok order line item is the source of truth for status
- Decide how to handle timezones, during dev/testing this doesn't matter, but make sure this is determined before production.

Steps:
1. Open a blank and open sales order for the current day in NETSUITE_DOCUMENTS, we will not actually have a document in netsuite for sales orders.
2. Load daily order data once a day after the day has ended into a raw json table
	1. A days order data has to continue being collected until the "sales order" is closed. Loop through the open days, query data for each open day.
2. Transform and upsert the days tik tok order data into two tables:
	1. ORDER table, header level data that is shared across line items for that order
	2. ORDER_LINES table, Order line item level, there will be a quantity column, for tik tok this is always 1 and there can be multiple rows per seller SKU per order
	3. CONSIDER: ?? "Sales order" level. a sales order document_type in NETSUITE_DOCUMENTS -> the entire days list of orders, a row per line item in NETSUITE_DOCUMENT_LINES for that days orders, either this or you just group by day to determine what is a days "sales order"
3. Reference or import Fulfillment order data from FOMS. A FO line item can be referenced by source_line_item_id and a FO can be referenced by order_id
	1. For fulfillment_type of "FULFILLMENT_BY_SELLER":
		1. Fulfillment order data is the source of truth for what has shipped
	2. For fulfillment_type of "FULFILLMENT_BY_TIKTOK"
		1. tik tok api data is the source of truth for what has shipped.
	3. For analyzing discrepancies, diff the status between tik tok order and what's stored in FOMS fulfillment order data 
4. Create an invoice PER "destination" (e.g. fba_us or logiwa_phoenix) for everything that has shipped that day, reference the correct source of truth based on fulfillment_type
	1. Create an invoice in Netsuite and store related data, line items are grouped by destination SKU. (one line item per destination SKU, sum quantity)
	2. Create a row in the NETSUITE_DOCUMENTS table for this invoice, with information like document_type (invoice), location_id (logiwa_phoenix), etc.
	3. Create rows in NETSUITE_DOCUMENT_LINES for mapping every line on the invoice to the source tik tok line. fulfillment order line can be tied to this by joining on columns source_line_item_id and order_id in FOMS.
		1. FOMs order line items table will provide our source and destination SKU's. The destination SKU from fulfillment order line is what ends up on the invoice when fulfillment_type is "FULFILLMENT_BY_SELLER". both fulfilled by tik tok and by seller will be in FOMS so we could just pull destination sku data from FOMS if we want but it doesn't matter, we can mix and match too, just make a decision.
5. in our UI, a "sales order" is "closed" if:
	1. every order line item for that day has been invoiced or cancelled.
		1. you determine a line item has been invoiced by joining with NETSUITE_DOCUMENT_LINES On tik tik order line id
		2. you determine a line item has been cancelled by:
			1. if FULFILLMENT_BY_SELLER -> reference fulfillment order line item
			2. if FULFILLMENT_BY_TIKTOK -> reference tik tok order line item
	2. You could have a script and summary table for calculating this once a day instead of needing to run the joins for that day every time the UI is loaded, if this becomes a performance issue.
		1. if using NETSUITE_DOCUMENTS for sales orders then status could be filled in there
6. orders and line items will need to have their status updated by the tik tok api until it has reached a "closed" status. Some discrepancies can be flagged by comparing these status's and what's stored in NETSUITE_DOCUMENT_LINES, as what's stored there we have already acted upon based on shipped. cancelled order status + a line in netsuite document lines = return or some other discrepancy. 



Level of visibility and audibility so we can see everything that's happening and what's being applied to what. May need to have a human checking off on certain things. use event logging and events table.



Needed tables:
%%	Netsuite Sales Orders
	Netsuite Item Fulfillments
	Netsuite Invoices%%
	

In Early morning:
%%1. Query Tik Tok API for order data for the previous day
1. Load into Bronze snowflake table%%
2. Transform into silver table data
	1. create SO script
	2. update SO script
	3. 
3. After SO is finished, do RAW->BRONZE->SILVER Transform in snowflake for 
	1. /finance/202309/statements
	2. /finance/202501/statements/{statement_id}/statement_transactions
	so that we can get the needed data for building credit memos on an invoice.
		[[cursor_tiktok_api_data_retrieval]]


Transform tik tok orders from Bronze into Silver Tables
- Map Tik Tok Bundle SKU's into real SKU's
	- These can have multiple SKU's and multiple QUANTITY
 - Every line on a sales order should have a way to reference its related order id, and then also its sales order line id. This will allow us to connect those lines to a remittance.
 - 

tik tok order line contains the sku mapping.


we will create a single sales order on a day, the location is TBD for that sales order, but we don't create a separate one for each location.

IF's ARE split by location, so for each day based on what shipped, an IF is created for each destination, either AMZN, FBT, or PHX. On a single sales order we can have 3-4 days worth of shiopping so there will be could easily realistically be 4x3 IF's per sales order.


![[Pasted image 20261014111227.png]]

![[Pasted image 20261014111334.png]]