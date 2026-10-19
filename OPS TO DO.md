Fix Labels
-  Error handling, logging, event logging, and a form to fix the data that was entered incorrectly, validation on that form to make sure that data is entered correctly the second time. UI/UX to display the issue and what needs to be entered to fix it.
- We don't have control over the inputted data because it comes through netsuite
- If the python backend doesn't have the relevant error handling then it needs to be added.
	- Get a list of requirements of all things that are needed in order to create a label


set up time for demo for the new shipment orders labeling process visibility

- permission management for what people are allowed to edit
- have an example error pulled up
- add to nav
- **DONE** display the last script ran in the labeling process and the next. Maybe show a diagram that shows what steps the current label will go through and what it's on currently, with like it highlighted green what will be next and what already happened.

scott
john mallard
ryan sarnacki
joey (if he wants)

John - warehouse manager, head guy -> makes sure logiwa is working properly
ryan - warehouse manager, day to day operations - > ship it guy


### Add to shipment-orders page:
- Errors column
- Path (Label Pipeline) column
	- ADD THIS!!!!
		- just hard code the filters in the table instead of having it dynamically populated based on what exists like the other filters.
- Created At column
- ~~Dedicated template for showing label statistics.~~ 
	- ~~Move from the label monitoring test page~~
		- increase the text size inside of these cards
- Created at date filter
- ~~Update text when mousing over "cron jobs healthy" to show "Open Grafana", make sure the size of the box doesn't change when doing this~~
- make sure all health cards show loading while loading
- ~~add background to the "hide completed" checkbox like in the detail page~~
- ~~Only show orders from the last few months or find a better way to filter out dead/cancelled shipments.~~
	- ~~orders can be status cancelled and also possibly just don't even exist in logiwa any more~~
	- ~~Have a reconciliation cron job that runs through Logiwa and compares all our open shipment orders with Logiwa, we need a "deleted_at" column that would allow us to keep the old data in the case of it being "un-deleted" or "un-cancelled"~~
	- ~~A new script that fetches shipment orders that haven't been updated in 3 days and checks what their status is in Logiwa.~~
		- Setup the cron job to run once every 24 hours @ 1 am **IMPORTANT**
- failed last run should be changed, we should show domain errors instead?
	- well they auto resolved so idk, resolved but still having error
		- error events (24h)? -> click goes to show all orders (including completed) and 
			- Grafana already has this Domain errors (24h) and orders failed (24h)
- Jobs healthy is clearly not working correctly? OK (no failures in the last 24h)
- can't alt click the exact shipment orders some reason
- create Netsuite Package creation sometimes running before upload tracking info, which causes 
- ~~fix "Database error: 1038 (HY001): Out of sort memory, consider increasing server sort buffer size"~~
	- ~~temporary fix for now, just increase the sort memory for that one query in the flask app~~
- When there is any filter applied then "clear all filters" should change color and maybe do a little jig (animation)

### Add to detail page:
- Highlight order status orange when shortage
- Ability to manually run certain cron jobs within the detail page when clicking on a job under "Labeling pipeline, will need to think about how this will work and what data needs to be reset
	- hover over shows refresh symbol
	- clicking will prompt a browser message asking if you're sure and what will happen.
- Don't upload the entire json to the events table but we can include a link to the where the json is uploaded.
- Sync button that syncs orders with logiwa
- auto refresh all data every 5 minutes
- Reset button to set status to Open and run the order sync that resets all fields and restarts the label process for this order.
- Button for downloading Carton/Pallet labels if they've already been generated (check flags), polls for the most recent timestamp from Logiwa.
- Time-misalignment between detail page and shipment orders database etc? Holy so bad

DW:
- speak to alex/jason about what data they need for forecasting and what that forecasting data looks like, what are forecasts based off of.






Hello everyone,

My name is Michael Brandmeyer and I'm the new software engineer at JLab. Under the guidance of Scott Pittman and Joey Babcock I'll be building internal software to help streamline operations and improve the efficiency of existing systems. Something I've been working on recently is providing visibility into the label creation process so that operations can easily view the current status of shipments, their labels, and any errors associated with a shipment order and their labels. Our goal is to make error resolution self-service so that errors can be easily resolved by the operations team when they arise. 

I'd love to show you what that looks like and if you'd like to take a look yourself before our meeting feel free to at https://logistics.jlab.com/shipment-orders. I'll also be in Phoenix Tue Aug 25th and Wed Aug 26th.


Hello John Mallard and Ryan Sarnacki,

I'm Michael Brandmeyer, the new software engineer at JLab. Under the guidance of Scott Pittman and Joey Babcock I'll be building internal tools to make day-to-day work easier and help operations/logistics run more efficiently.

I've been working on a tool that provides visibility into label creation so operations can see shipment status, labels, and errors with the goal of making label error resolution self-service.

I'll walk through everything in our meeting, and you can preview the tool at [https://logistics.jlab.com/shipment-orders](https://logistics.jlab.com/shipment-orders). I'll be in Phoenix Tue Aug 25 and Wed Aug 26 as well so I look forward to hopefully meeting the both of you in person.

Thank you,


Hello John Mallard and Ryan Sarnacki,

I'm Michael Brandmeyer, the new junior software engineer at JLab. Under the guidance of Scott Pittman and Joey Babcock, I'll be building internal tools to make day-to-day work easier and help operations/logistics run more efficiently.

I've been working on a tool focused on the labeling process. It gives operations visibility into a shipments current status, shipment details, associated logs, errors, and more. The goal is to make label error resolution self-service.

I'll walk through everything in our meeting, and you can preview the tool at [https://logistics.jlab.com/shipment-orders](https://logistics.jlab.com/shipment-orders). I'll also be in Phoenix Tue Aug 25 and Wed Aug 26, and I look forward to meeting you both in person.

Thank you,


I've been working on a tool that provides visibility into label creation so operations can see shipment status, labels, and errors. The goal is to make label error resolution self-service.




sell in
vs
sell through



sold a bunch in but not a bunch through, this will be a problem

compare sell through data against forecast data

1800 records ->


NEW LABELING:
Page where we add a new retailer.
On this page we map what the needed fields are for the labeling process. So all the required fields from logiwa and everything that's needed for building that label is done within this page instead of the if else branches currently within the labeling pipeline pages.

These destinations will have data validation so if any information is missing that's needed for generating these labels, the errors will pop up within your shipment orders page.

Conditions/"Rules" for when certain fields are required, so based on these things, these fields are required. By default that's normally based on the retailer.
