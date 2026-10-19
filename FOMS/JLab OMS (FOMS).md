 [[FOMS Flowchart.canvas|FOMS Flowchart]]
 [[FOMS Models.base]]
 [[FOMS Controllers.base]]
# To Do
- app should hold credentials instead of individual files
- setup GitHub actions
- need to implement [[JLab FOMS Bulk Operations (Tasks)|mass actions]]
	- ~~mass cancel~~
	- ~~mass move~~
	- mass SKU update/translate
	- ~~mass submit~~
- [[JLab FOMS Event Subscriptions|Event subscription system]]
- [[FOMS Move to MVC Architecture]]
- ~~use rules for [[FOMS SkuTranslations]]~~
- unhold held orders when inventory comes back into stock
- search results system for orders, similar to Shopify
# Models
- [[FOMS FulfillmentOrder]]
- [[FOMS FulfillmentOrderLineItem]]
- [[FOMS Storefront]]
- [[FOMS OrderDestination]]
- [[FOMS NetSuiteCustomer]]
- [[FOMS ShippingInfo]]
- [[FOMS Shipment]]
- [[FOMS ShipmentLineItem]]
- [[FOMS Inventory]]
- [[FOMS InventoryFeed]]
- [[FOMS OrderSource]]
- [[FOMS NetSuiteInfo]]
- [[FOMS SkuTranslations]]
- [[FOMS Router]]
- [[FOMS Rule]]
- [[FOMS Criteria]]
- [[FOMS Event]]
- [[FOMS EventType]]
# Controllers (CRON jobs)
- [[FOMS Import Fulfillment Orders Controller]]
- [[FOMS Submit Fulfillment Orders Controller]]
- [[FOMS Confirm Fulfillment Orders Controller]]
- [[FOMS Import Shipments Controller]]
- [[FOMS Invoice Shipments Controller]]
- [[FOMS Import Cancelation Requests Controller]]
- [[FOMS Bulk Invoice Orders Controller]]
# Order Flow Chart
https://lucid.app/lucidchart/ae09c638-e7e9-4d7f-a6f6-d1b36ce32525/edit?invitationId=inv_62300080-1a35-40d6-85f2-0953247c644e&page=0_0#

![[foms_flow_chart.png]]
# DB Schema
https://lucid.app/lucidchart/ae09c638-e7e9-4d7f-a6f6-d1b36ce32525/edit?invitationId=inv_62300080-1a35-40d6-85f2-0953247c644e&page=qBfKEzCZnkzF#

![[foms_schema.png]]
## Order Flow
- Integrate with anywhere we have orders
- pull their status
- push orders to 3PL
- update status
- push fulfillments
- cancel in marketplaces
- cancel FO's in 3PL
- all in bulk
- tag known OOS orders
- filter by expected ship date
- split/combine
- sku translation
	- by market/marketplace/availability
## Inventory 
- Update inventory in marketplaces
- Pull current inventory counts from 3PLs
- fetch inbound
## Reporting
- Inventory
- Order Flow
# PSR Structure
`vendor/joeybab3/router`
- order router package
`vendor/joeybab3/foms`
- main FOMS package
# Interface
Mostly table based, individual FO's can be clicked on to inspect that:
- change SKU
- push
- cancel
	- can recreate a new pending order and link old/new orders through attributes
- pull
- change location
## Move / Cancel Rules
- Closed orders (`cancelled`, `completed`, `shipped`) cannot be moved.
- Submitted orders cannot be moved once they have a destination URL or destination order ID.
- Open local-only orders can still be moved.
- Canceling can recreate a new pending fulfillment order and preserve linkage through attributes such as `created_from_fo_id` and `new_fulfillment_order_id`.