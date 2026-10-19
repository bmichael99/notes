June 30th 2027


Have a 3PL client in this or a warehouse
- depends on capacity


Demonstrate that we can ship LTL, b2b products on time and on target


omni channel fulfillment, ship dtc at scale



Issues to solve
- how would a customer get orders to us
- how would a customer monitor and edit/ensure those orders are being processed, SLA's
- How would we let them know they're shipped


build a system that's useful to us and to cutomers that have the same problems and concerns as us


providing an add-on service of if transit times are accurates


baseline:
- what do u want it to ship
	- did it ship that day?



January 1st
- ?

Biggest needs
- transparency
- visibility
- What's going on?

What does the client need?:
- visibility for wether the package is on time
	- by day how many on time how many late

Download a report
- here's a report of what shipped out, what didn't, etc

every single table should be exportable to CSV
- here's what we're using to generate this

On boarding process:
- someone needs to hold their hand
	- first 90 days
- in terms of visiblity:
	- what they need
	- "what the delay is"
- what orders are leaving
- desired ship date
- carriers
- daily
	- what the output looks like
- automated reporting
- emails for what could or couldn't ship out
- self-service on the website
- cs representitive that's responsible for a set of accounts

Integrations:
- flat file you ingest it
- send an api response and we consume that and store it
- shopify integration, etc.
- sales order, purchase order, we return an id that reflects the UUID of the sales order.

flat file
- would include a customer id or something to identify who the client is

frustrations:
- real reporting on where our orders are
- what is the realistic time for when orders would go out in rushes
	- labor planning issues for black friday, etc
		- ecommerce not b2b
- inventory
- identify things that will be late
	- how late are they going to be?
		- telling client "I don't know" -> really bad

priority plan, maybe redo priorities for orders (1 is important for jlab,):
- by client we can configure these tags
- 5 different clients that have their own priorities
- waves will go through
	- only wave priority 1 customer 1 -> customer 2 -> customer 3
	- priority should have a weight for whoever is later

current demand

waves are already set for the day. work goes to the pool.
- priority orders
- international orders
- single identity skus
- then multis, mediums, large, extra large

when those waves run we don't utilize that information because of how slow it is
- during peak when had priority issues, waves were being assigned to individuals

different workers were 

everything waved that we didn't work gets "cancelled" and then those jobs go into the waves for the next day

by client waving?
- worst case scenario 24h delay to get bodies into the building

"drop everything else, this order needs to go out RIGHT NOW"

clients have their priorities
- map those priorities based on lateness and other weights

"walmart marketplace is priority 1"
- let them self service that

is the shipment cheapest best

routing rules and priorities so they that clients can determine the order of things on their own

may need to contact CS in order to set something is a true priority 0




ingest
monitor
return

retail compliance
- rework of ASM as well





Walmart dashboard:
List out the walmart purchase orders
- which ones have a matching purchase order in Netsuite
	- Link to netsuite
- If there is a match then have a line item detail as well
	- what are the differences between line items
- If there is no match then still list out the line item detail but the diff will be complete

the walmart dashboard would need to accommodate for updated walmart purchase orders

think of the case where we have multiple netsuite invoices. We need to sum those line items against the walmart order, and when the walmart order updates then 

inventory in location overtime

transaction history 





## TODO:
Wrap all cron jobs on the scottlot
cron job -> is ? running consistently.

System dashboards
wrap all of them

look back in recording for the specific of what scott was saying here ^ 
