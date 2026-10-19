Opener/Intro
- I know that before I started working for Jlab, the company moved over to an internal custom solution for label generation for orders that don't run through our fulfillment order management system.
- This tool gives the operations team visibility into the label generation process so that errors can be resolved by anyone on operations.
- Visibility into errors along with the needed information to understand why this error is occurring. This information gives you the ability to either fix the issue yourself or escalate the issue to the relevant parties.

A little bit of an introduction, my name is Michael Brandmeyer, I'm the new junior software engineer here at Jlab. I've been here for about 5 weeks I believe, and one of the first few things I've been working on is providing visibility into the existing label generation pipeline, so that errors can be resolved by the operations team and provide context when things go wrong.

### **Actual Intro:**
My name is Michael Brandmeyer, I'm the new junior software engineer here at Jlab. I've been here for about 5 weeks now, and one of the things i've been working on is a tool that provides visibility and error handling for our label generation process for shipment orders in Logiwa. So, since we have our own custom solution for generating labels, you may have ran into situations where you didn't receive labels for a shipment order, and maybe you had to notify Scott or someone else in order to resolve that issue. The goal of this tool is to give the operations team the context they need in order to resolve those errors. The reason that's possible is because most of the time when labels can't be generated it's because a piece of important information is missing either from Logiwa or NetSuite, and this tool will help provide a way for you to see what's missing and the ability to update that information.

Demo Path:
- Start from home page
	- Navigate through navigation bar to the shipment orders page
- Shipment Orders Page
	- This page does two things that I will explain detail in a moment
		1. Provides an at-a-glance health check of the label generation pipeline.
		2. Allows you to view and filter shipment orders within the context of the label generation pipeline along with errors associated with those errors.
	- Start with the Label Monitoring section at the top



- 3 goals (find stuck work, understand why, fix/escalate)
- Demo path (6 clicks)
- 5 likely questions + your answers
- Close: “use this as the morning board”


CREATE SOP:
- click show errors to filter by open errors
- select shipment order in the "active shipment orders" table
	- This will open the shipment order detail page for the selected shipment order
- Errors for this shipment order will appear at the top of the page
- Click an error that you want to resolve
	- This will move the screen to where the error occurred 
- Read the description for the selected error, and update the relevant information to resolve that error.
- If updating a form, the error should automatically resolve.

which PO's are they receiving on incorrectly
- are they canceled
once an order is closed or cancelled are there any adjustments 

we need to figure out how to filter our update to date information an compare that against walmart's invoices so that we can correct their possible mis-information

When status is cancelled the total quantity received is zero'd out, are they receiving on canceled?
"totalQuantityReceived": {
	"vendorPacks": 0,
	"sellableUnits": 0,
	"warehousePacks": 0
},

Status closed does still show possibly the correct information

"quantityOrdered": {
	"vendorPacks": 1678,
	"sellableUnits": 1678,
	"warehousePacks": 1678
},
"quantityReceived": {
	"vendorPacks": 1678,
	"sellableUnits": 0,
	"warehousePacks": 1678,
	"receiverQuantityData": [
		{
			"receivedOverQuantity": "0",
			"receivedShortQuantity": "0",
			"receivedDamagedQuantity": "0"
		}
	]
},


add column for the created date 


Email list for sending emails automatically when there is an error

"export error" button that extracts what the errors are so that it can be copy and pasted into an email:

Contain the list of errors and a link to the shipment order.


Middleware for auth based permissions
- what a user has access to based on their role
- Add authentik to the logistics website
- add roles for logistics



Compare the data from the walmart API, compare against netsuite

We are first comparing the data in the invoice with the matching order in the walmart API.

We are second comparing the missing purchase orders that are not in netsuite but that ARE in the walmart API. We only care about cases where they owe us money. They will do their own due diligence to make sure that they have received money that we owe them.

daily_table_upload_customer_defaults.py 
and
data_set = 'custdataset_logiwa_customer_edi_tracking_requirements'
url = f"https://{environment}.suitetalk.api.netsuite.com/services/rest/query/v1/dataset/{data_set}/result"

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

