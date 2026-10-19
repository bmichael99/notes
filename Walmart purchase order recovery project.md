Compare the data from the walmart API, compare against netsuite

We are first comparing the data in the invoice with the matching order in the walmart API.

We are second comparing the missing purchase orders that are not in netsuite but that ARE in the walmart API. We only care about cases where they owe us money. They will do their own due diligence to make sure that they have received money that we owe them.

daily_table_upload_customer_defaults.py 
and
data_set = 'custdataset_logiwa_customer_edi_tracking_requirements'
url = f"https://{environment}.suitetalk.api.netsuite.com/services/rest/query/v1/dataset/{data_set}/result"


TODO:
- Since closed purchase orders can keep receiving orders even though it's marked closed, we should 

Two main things: If there is an invoice in Netsuite that we've created that matches a purchase order within the Walmart API, then we should compare the data we received from Walmart and their received items. If walmart received more items than what we invoiced them for, then we create an invoice for that and get our money back.

Second thing: If there is a purchase order within walmart that is not within our Netsuite dataset then we need to make an invoice for that and get our money back.

We need to be able to adapt for having multiple invoices for 1 purchase order. So when we notice the discrepancy and generate another invoice, we now need to sum up the first invoice, the second invoice, and compare that against what walmart received.

Another feature that's needed is if the raw_json is ever updated and the amount received changes, we should probably just keep track of the highest number of amount received?

Audit: Are there any older 


is where we can pull a list of invoices, this is a custom built data set within netsuite.

We invoice walmart and they pay us what we invoice, we may accidentally invoice for less than what they actually received because they have another purchase order they're fulfilling against and then they say we actually only received this much because they look at the matching purchase order that we invoice on and they say they only received this much on that purchase order even though they were filling more than one purchase order. That's the case where we go and find the purchase orders that we haven't invoiced on but that walmart has in their system.

vendor packs = case packs

go through each line item:
vendor packs * casepack # * unit price

The Netsuite data set flattens out all the line items for an invoice. The transaction line (BPN) should match the Walmart item number.

generate invoices for purchase orders that are in walmart but not in netsuite. (they received items we haven't invoiced)

add visibility for this, so have a web page that shows us the differences between the walmart purchase orders and the purchase orders within netsuite.

vendorPacksReceived

- walmart
	- raw
		- shipment orders


meet @ jlab @ 4:30


do walmart purchase orders need to wait for a certain status before we attempt to process them?

ability to see the changes of the raw json within the website


does "purchaseOrderLineStatus": "APPROVED" for a line item need to be set for us to be able to count the received quantity?

need Purchase order status column
need Purchase order supply chain status column

We can just say that the status doesn't actually matter, what matters is that we sum up the line items for every order no matter what, and if the received quantity is above 0 then we need to compare it against our purchase orders, if there is a matching purchase order then we compare the amounts. If the amount on the walmart purchase order is higher than our invoice then we need to create an invoice with the differences, for now just log what the differences are, multiply by our unit price, and sum up the amounts. If there is a received quantity above 0 and we have don't have an invoice for that order then there is nothing to compare, and we simply diff and generate an invoice for all the items in that walmart order.

There is the case where there is more than one netsuite invoice for the walmart order, in this case then we want to sum up our invoices. Maybe there is also the case where there is more than one walmart order for our invoice, in that case then we want to sum up the walmart orders and compare against our summed up invoices?

The hostPurchaseOrderId is how connect the walmart purchase order to our invoice.
- Walmart: hostPurchaseOrderid -> Netsuite: PO/Check Number (otherrefnum) from api

For line items, under walmart the key is "purchaseOrderLines" which contains a list of line items:
- Walmart: walmartItemNumber -> Netsuite: transaction line: BPN (custcol_sps_bpn) from api


## Walmart PO STATUSES

Domestic & Distribution Center PO Statuses

- **OPEN:** The purchase order is active and open in the system.

- **RECEIVED:** Walmart has received the goods or shipment at the facility.

- **CLOSED:** The purchase order is fully processed and closed.

- **CANCELLED:** The purchase order has been cancelled. [[1](https://developer.walmart.com/suppliers/reference/getallwalmartpurchaseorders)]

Import PO Statuses

- **AT_PORT:** The import shipment/order has arrived at a port.

- **CANCELLED:** The import order was cancelled.

- **CLOSED:** The import purchase order is closed


walmart receiving 

currently receiving on incorrect purchase orders

we invoice from the correct PO

walmart is receiving is receiving on closed PO or partially the correct PO

store all our data for these PO's so that we know what actually occurred 

walmart retail link related

variant column contains the entire json response so that we have a history of everything in the case where we want to build a new column

tik tok invoicing 



database for raw -> schema for each data source
database for 



database: department
- Schema: source
	- Tables: business process from that source

load data from source into a local database
- cron job that runs on frequent intervals that checks if data exists in source that isn't stored in local database.
- bulk load once a day into snowflake

"nextCursor" is used for pagination, it's an identifier that lets you know where you're at in the list of walmartOrders

if we get invalid request due to auth then we retry fetching access token

fetch all purchase orders for the past ? days

### ETL FLOW:
- fetch all orders for the past DAYS_TO_FETCH days
- store in orders array
- go through orders array and fetch all by id.
- store each json payload for a unique purchase order identifier into database
- if id already exists then check last updated 

if we get 401 not authorized then we should try to get access token
if we get 429 too many requests then we should wait and retry again with exponential backoff



api can update purchase orders, "lastChangeDate"
- append new row if there's a new date here?
- upsert

- `purchase_order_id` (PK)
- `host_purchase_order_id`
- `raw_json` (JSON array of versions)
- `updated_at` (last append time)
- first_extract_at 

json:
- extracted_at
- hash
- payload


data for the walmart dashboard should be connected to a client code for jlab. We may be doing this for other people and want to ahev the data be connected to a specific client.

Client table for different clients, fk referencing this data.




algorithm:
ns_data = get_netsuite_data()

// Key by BPN instead of hostpurchaseorderid

loop through walmart orders:
	ns_data(bpn)(qty) -= walmart_order(bpn)(vendorpack) * netsuite_item_master_info(casepack)

BPN is just a unique ID for that sku that walmart references






There is a case where purchaseOrderSupplyChainStatus is just non existant, this is another instance why we don't care about this status and instead just count by quantity received

as soon as we ship something, we auto invoice in the fulfillment record

they send a purchase order, they have additional that they're not closing out


TODO:
Check if Scott's inputted CSV's only include purchase order status of CLOSED, or if it also shows non CLOSED.

### Purchase order invoice tables

purchase_order_invoice:
	id
	purchase_order_id @FK -> purchase_order (RESTRICT)
	invoice_name
	sps_url_path
	created at
	updated at


purchase_order_invoice_line
	id
	purchase_order_invoice_id @fk references purchase_order_invoice(id) cascade delete
	purchase_order_id 
	walmart_item_number
	invoiced_eaches
	unit_price
	invoiced_amount
	status
	created at
	updated at