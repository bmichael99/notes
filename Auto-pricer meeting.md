Sale Price Strikethrough price Discount price
Listing price
MSRP price is defined within Amazon, that's our reference price



When we lose buybox we get an email.
Auto-pricer on weekends since no one in office

Fraud
UK, Australia, Canada, 

US First

auto-pricer on weekend
weekday manually checking
Auto Auto-Pricer and Auto Promo Calendar
[[Auto Auto-Pricer]]
[[Auto Promo Calendar]]
Auto Auto-Pricer:

Page in playground for the following:

Auto-Pricer:
- A list of all SKU's and whether or not Auto-Pricer is enabled, along with another button to set a schedule for that SKU.
- Ability to bulk apply schedules based on filters, and the ability to both export and import a CSV to change the schedules in bulk

Steps:
1. Create API endpoints for 

Table lists Seller (AMZN) SKU, Parent SKU, and child_asin, along with schedule and a schedule picker

ecomm/amazon/pricing/
  index.php                 # Auto-Pricer page: portal header, empty div, script tag
  calendar.php              # Promo Calendar page: same shell, plus the CSV form
  pricing.js                # All fetch() calls for both pages
  api/bootstrap.php         # Auth middleware + pricing database connection
  api/rules.php             # GET/POST/DELETE scheduling rules, keyed by SKU
  api/calendar.php          # GET the saved promo schedule, POST one row
  api/upload.php            # POST CSV to validate, POST again to commit
  api/skus.php              # GET existing SKU/ASIN matches for the picker
  api/log.php               # GET what the cron has sent to Amazon
includes/class/pricing/
  PricingStore.php          # All SQL: rules, calendar rows, uploads, apply log
ecomm/amazon/pricing/schema.sql
                            # CREATE TABLE for those four tables
ecomm/cron/amazon/apply_scheduled_prices.php
                            # Cron entry: due rows → Amazon price update → log


Fine grain scheduler for one SKU
Bulk scheduler for SKU's
- based on current filters

When an auto pricer is flipped on, we can show enabled to the user but on the backend we're still sending that request to Amazon, if amazon rejects for some reason we have a pop up showing an error and flip those back off. We also make sure to take a "screenshot" of the current price so that we can restore that price if we want to.

"Bulk Rules"


"Add filtered to bulk rule" -> opens modal of all bulk rules that looks similar to the other modal but this time includes a list of the SKU's this rule applies to and the rules of that bulk rule. You can edit the list of SKU's to add to remove sku's. When adding a SKU to the bulk rule there should be a search bar at the top and the drop down should actually be a checkbox instead of a selector, and it should include the sku's you've already added so you can check or uncheck any sku's you want from that drop down. There should also just be a 

Runs every day after midnight -> Promo Calendar and our CSV of normal listing prices gets applied to Amazon, so we set our price to the promo calendar price if it exists, if it doesn't exist on the promo calendar set "our price" to the price on the normal listing price CSV. Let Auto-Pricer re-adjust our price

Auto Promo-Calendar:
	A page listing SKU's and the same schedule button as in the auto-pricer. We can import these via CSV -- provide the US National Promos document to cursor so it can view the format. The table will also list current MSRP, savings, Promo MSRP, and % off. The calendar column indicates whether or not there's a scheduled promotion and that modal shows the schedule, the MSRP, savings, promo MSRP, and % off. Sort the modal by the closest date to today so they can view the closest promotion first. 

Need to add:
- A way of visualizing what SKU's are applying to amazon. Maybe add a way to configure what markets these SKU's are being applied to? Not sure.

Can we view current price and whether or not it's competing with another price, and what that other price is. If it's 12:10 AM and our price is currently competing with another price, we may not need to update our price to promo.

[[Prompt]]

1. match pricing and keep buy box above minimum 
2. show the new MSRP
3. get a slash price
4. when we don't have a slash, alert

ask jordan for manage products page

add date added column to promo calendar, and date updated?

if our sale price is below the auto pricer, it will take the sale price

go pop+ mint, 19.98

	flag if auto pricer vs msrp is above % 

Promo Calendar page, then promo calendar by source, this only shows the actually applied promotions to that site, what the JLab Sku is and its connected SKU (Amazon in this case).

"our_offer_price" is whatever the product page lists on amazon, this could either be the sale price or "your price".

"your price" means actually "your price"

maybe if auto-pricer is below what the promo price is, promo price will not apply

auto-priced, what's below our promo price. 

be able to turn off auto pricer on a SKU for a rule set.

ADD: a script that restores "your price" to MSRP or promo price, then let the auto-pricer do its thing. If a price should be below MSRP, it should be included in the "promo price". This is only REALLY needed because sometimes auto-pricer gets turned off and the price needs to be re-adjusted.
- We would also want to set "your price" to NEW MSRP and set sale price as the old MSRP so that it looks like a strike through but we're selling at the old MSRP. so we want to have a site wide sale with the old MSRP as strike through/sale price, NEW MSRP as "your price".

RUN A TEST: on a product listing where we're auto-pricing against a competitor, set our sale price above that price, and see what price is listed.

here's where auto pricer kicked in

add our_lowest_price and our_max_price

Alright, on my end I think what Justin was looking for was a way

We'll also add a bulk upload button which opens an open file browser pop up so u can select a file, then an add promotion button where you can use a UI to

bulk upload overwrites all rows uploaded via file. The parser should not include any date ranges that we've already passed.

Importing the same promo, with the same columns (except source_file I guess), should not write another entry to the database, so if I upload the same CSV twice it should do nothing, and uploading a CSV that's the same but with new rows, this will allow it to only add the new rows. When the same SKU has a promotion with overlapping dates with a DIFFERENT promo price, we need to have a dialogue for resolving those conflicts that displays those conflicts, and the user can either delete rows to resolve those conflicts or edit rows to resolve those conflicts.

Promo Calendar loads into amazon as so:
- On sunday at midnight, load all promotions that start before next sunday into amazon as the discount_price/sale_price along with the calendar it sets.
- Should include a dashboard page showing whether or not a promotion has successfully loaded into amazon, or if it failed.


We should have pages for viewing how Jlab SKU's map to different market places SKU's/ASIN's/etc.

for Listing Prices page:
- issue identifier function
	- If auto-pricing is disabled, there's no competitors, and our price range is below MSRP.
	- If offer price drops by 10%, flag this.
		- Keep a history of offer prices for all market places, for now just amazon, have an append only table for this.
- Include JLab SKU/Description


On the issue alerts section, add a note/description column.
- For high price, include our offer price and competitive price

Be able to filter by family on the listing prices page, 

Add the ability to adjust prices via API straight through the listing prices page, 

add approve and apply button when creating a new promo

TODOs:

Have to approve PROMOs per market, and have the ability to make custom adjustments, have an icon or drop down for viewing the adjustments on the main page. Or maybe a drop down shows the promos across all markets, so you can do the drill down within the one page instead of going to the other pages if you'd like on one SKU.

when auto-pricer drops price

connect to all market places and make sure we're pulling prices for all market places

When uploading a file be able to view the failures for that file, like logs. 

Still needs to be discovered:
- Whether or not discount price wins over auto-pricer

Auto Promo Calendar:
- Ability to upload a CSV to set amazon Promos.
- 

CONSIDERATIONS:
when a promo is deleted from the main promo table, what happens....?
- cascade delete down all its children tables
	- prompt the user whether or not it wants to delete saved prices in amazon, and those prices will now be like "orphans" without us keeping track of it but still applied.

Playground tool for 

Worked from 8:00 PM - 9:38 PM on tue Oct 6

TODO:
For Listing prices:'
- [x] Include ASIN on listing_prices page and alerts section
- [ ] Automatically do strike through when we lose buy box, 
	- [x] PRIORITY: first fix buy box detection
	- [ ] Look at the endpoint that sends you webhooks for when we lose buy box
- [ ] List of SKUs:
	- [ ] POP, go sport, jbuds, go pop, lux, minis, every 5 minutes, 
- [ ] Alert when that happens, visibility when price changes to competitive price.
- [ ] REPLY TO JOHN

Differentiate the auto-pricer discounts vs promo calendar discounts
- continue promo calendar when auto-pricer adjusts

- What to do when a price re-raise


- raise based on competitive price
	-  don't move past promo calendar

- When auto pricer is enabled:
	- If we lose buy box then lower sale_price to competitive price
	- If competitive price raises, then match sale_price to competitive price, but do not move past MAX or the current promo price
	- If we disable auto_pricer, re-set the sale price to promo price if there is a promo on that day.
	- 



When we receive notification: update bronze and silver listing prices for those SKU's based on ASIN. 

Silver responsible for triggering the auto-pricer


php script for running the following procedure, with a SKU passed in as parameter, this gets called after silver load as part of the silver listing_price cron, around the time when alerts get called. This procedure also gets called when auto-pricer is flipped on from an endpoint that gets called when auto-pricer is enabled.
- Check if auto-pricer is enabled for this SKU, return instantly if not
- Set your_price to list_price
- If we don't own buy box, Lower sale price to competitive price, with a limit of min, so max(min_price,competitive_price) is the floor
- If competitive price raises, then match sale_price to competitive price, but do not move past MAX or the current promo price, so the ceiling is min(competitive_price, promo_price, max_price)
- 


Endpoint that gets called specifically when we disable auto-pricer:
- If we disable auto_pricer on an amazon SKU, re-set the sale price to promo price if there is a promo on that day.
	- If there is no promo that day, delete sale price from amazon


raise to sale price to old MSRP, so it strike throughs with the new MSRP as the listing price


be able to wipe current sale price/promotion

sale prices currentlly block auto pricer your price

- promo calendar 

- [x] Buy box needs to be accurate
	- [x] Include weight %
- [x] Add ignore list for Amazon SKU's on promo calendar
- [x] Add min price column to promo calendar'
- [x] Enable the script that applies promo calendar discounts, daily
- [ ] Build a table with old MSRP's, when we do have an old MSRP for a sku, this becomes the default "cleared" state in the "clear()" member function. Currently we just restore both your price and list price to the new MSRP, without a discount set at all.
- [ ] Add a feature to match on-amazon price for a specific SKU, that only matches while their price stays the same and still exists
- [ ] Generate an alert when something goes inactive, manual resolve button, column for this type of alert being manual review or just have it be client side based on the issue type
- [ ] You can put the same sku for the same dates with no warning appearing on the promo calendar, fix...
- [ ] We should also figure out a way to have visibility into recently added custom promotions so that it's obvious which SKU's need attention.
- [ ] Have a script that flags when our price is below the competitive price
- [ ] Show why sale price differs, e.g due to competitive price match via auto pricer
- [ ] For the in-house auto-pricer:
	- [ ] When there's no price to compare to, set discount price to old MSRP, list price should be new MSRP.
- [ ] Add an edit button to the listing prices page that lets you edit anything in the amazon page instead of needing to go into amazon seller central
- [ ] CHeck if SP-API can set multiple categories on a product
- [ ] Add bulk actions to the listing prices page for: 
	- [ ] Deleting discount price in amazon
	- [ ] Set discount price to old MSRP and set list_price and your_price to new MSRP. You do not have data on the old MSRP's yet, I will create a table for this eventually, but you can skip this one for now.
	- [ ] Set your_price and listing_price to Netsuite MSRP
	- [ ] Enabling in-house auto-pricer
	- [ ] Disabling in-house auto-pricer
	- [ ] Enabling amazon auto-pricer
	- [ ] Disabling amazon auto-pricer
- [ ] Click a SKU in alerts to filter for that SKU's data in listing_prices
- [ ] Setup an alert system on playground
	- [ ] Email
	- [ ] Teams channel
	- [ ] Notifications on playground
- [ ] When we have a sale price set by the auto-pricer for a certain amount of time, with a warning/alert increasing in "danger" level based on the amount of time. Visibility tool for this so that we can see when 
- [ ] Improve the flow of the promo calendar, from bulk actions to better conflict resolution, etc.
	- [ ] Conflict Resolution
	- [x] Bulk actions
- [ ] Update the way new promos are imported through CSV
- [x] Make sure sale price includes the date range, this is populated by discount_price I believe, make sure this lines up correcty.
- [x] Update the description for offer price and make sure this is actually accurate, I'm pretty sure this just pulls our price based on our seller id so it should be accurate
	- [x] Listingprice from getPricing also should show this price
- [x] Show when listing price doesn't match Netsuite MSRP
- [x] Add sync auto pricer rules as an hourly cron
- [ ] Update delete to remove the applied promo if the applied promo matches what's being deleted, so delete promo in amazon if the promo being deleted is currentlya ctively applied, or give an option on whether or not to do so, a yes or no.
- [ ] Handle overlapping promos and cutting, request id 55efee03-e408-4243-9326-ccaea2c36686
	- [ ] split_overlapping_promos_189bc4ce.plan.md
- Sale price should probably only show on the listing prices page if it's currently on promo, so will have to do a date range check on that
Continued:
- for promo calendar, Show current sale price that's on amazon and whether not they're in sync or conflicting
- View the current price and whether or not it's competing with another price, and what the other price is.
	- Look into offer price below you price and/or sale price
- Whether or not auto pricer is on, What the current price is and a history of our price, how far below MSRP is our price or how far below promo is our price if it's on promo, 
	- flag if auto pricer vs msrp/promo is above %
- A history of our price, and the ability to see "here's where auto pricer kicked in"
- A badge on the promo calendar for 
- An ignore list for amazon sku's on the promo calendar
	- shows "ignored" as the apply status
	- add a quantity column to the jlab sku -> amazon sku map so that bundles can be mapped correctly
- Inactive "Apply Status" does not need to show Inactive listings
- Ryan to do: populate the metafields on jlab.com shopify based on the promo calendar, support having a custom promo calendar by retailer, so by amazon, by jlab.com, walmart, etc.
- Issues to flag:
	- There's no competitor price listed, and our current offer is below MSRP/Promo.
	- Current offer is below competitive, MSRP, promo price by more than x%
	- Current offer is below MSRP by more than x%
	- Current offer is below promo by more than x%
	- If below the lowest of these 3 by any amount, flag
		- your_price is below msrp and promo (whatever is lowest) by any amount

	- If we don't own the buy box
		- show reason for not owning buy box, could be inventory related.
		- Options: out of stock, competitor price, unknown

	- If current offer drops by x? (10%)
		- Has price dropped by 20% in the last 14 days from the list price.
			- Priority by age of price drop
		- Keep a history of all offer prices for all market places, for now just amazon, have an append only table for this.
	- high price flag, check amazon API for this
	- check if we can see strikethrough
	- See tues/wed if any of our deals are missing the best deal badge, if strike through pricing isn't showing, flag Kathleen, if there is then tell her.
	-

We can list 

Issues get populated into a table keyed with its connected SKU, with a column for ignore/acknowledge. This hides the issue from the UI and will not re-appear unless

https://developer-docs.amazon/sp-api/docs/retrieve-featured-offers-batch-asins

Adjust prices every hour
promo calendar amazon SKU ignore list

include an ignore list for an amazon sku
check if list price is different from netsuite MSRP

have a report that shows what has changed

Conclusions for acknowledgements:
- agent chat 555057e1-38ac-44ed-bcdb-2f53c0262e17
	- Listing prices API endpoint chat ^



Define flag conditions:
- select columns 
- on operands allow for min/max/avg/median on a list of columns
- on operators allow for +, -, x, /, <, <=, >, >=, contains, is exactly, 
- operand can be a column or a custom value 

how to handle order of operations, allow for parentheses to define them?

example:
offer_price, -, min(competitor_price,list_price,promo_price), /, 100, >=, 0.1

all SKU's and columns run through all rules
	-> if conditions are met, append to an Alerts table referencing its rule id

show "competitive price", show price needed to get buy box -> featuredOfferExpectedPrice->ListingPrice->amount


goals:
- win buybox
- avoid high price banner
- don't sell too low
- get slash back

- We should always have slash price if possible:
	- Auto apply slash price by taking the auto-pricer price, setting it as the discount price, and setting the your price back to MSRP.
- a script that restores "your price" to MSRP or promo price, then let the auto-pricer do its thing. If a price should be below MSRP, it should be included in the "promo price". This is only REALLY needed because sometimes auto-pricer gets turned off and the price needs to be re-adjusted.
	- We would also want to set "your price" to NEW MSRP and set sale price as the old MSRP so that it looks like a strike through but we're selling at the old MSRP. so we want to have a site wide sale with the old MSRP as strike through/sale price, NEW MSRP as "your price".
- be able to turn off auto pricer on a SKU for a rule set.
- show discounts that appear in amazon but don't appear in the promo calendar for visibility, don't show them as if they're normal

Lower price when:
- We do not own buy box
	- Lower to a max of (discount_price, min, competitive_price)

Raise price when:
- We do own buy box
	- Raise to a min of (your_price, list_price, promo_price, competitive_price, max_price)



Make the following extremely simple adjustments to the code:
- One cron job responsible for pulling the raw data from each endpoint on a SKU.
	- store the raw JSON, each endpoint gets its own table
- Another cron job that conforms the data into one merged silver table, one table contains the current state, another contains history.


**REFACTOR:**
FLOW: PLAN IN Amazon pricing data transformation agent chat and pricing_pipeline_assessment_618ac38a.plan.md
1. loop through each sku, hit all related API endpoints for that sku, store raw JSON, each raw JSON goes into its own table
2. After all JSON is extracted, another cron job runs that transforms the data into one silver table, run data through an issue/rule checker after it's transformed which is responsible for spawning alerts/issues.
3. Silver table has a current state table and a history table.
4. An alert uses its hash to determine whether or not it should spawn a new alert/issue
5. Since all data runs through the issue/rule checker we can also auto resolve issues.








sale price is above the "your price", <- FLAG THIS, it keeps sale price prio'd over your price in this case.

EBFLEXRTEL124

Discoveries:
- if discount price is below "your price" then amazon will take the discount price

Need to Test:
- If auto-pricer "your price" is set above the discount price, will amazon take the discount price or "your price"?
- Set "your price" to MSRP and see if auto-pricer will adjust the price
	- https://www.amazon.com/dp/B0FWTKRMFV is currently at $64.99 with no competitors but with auto-pricer on
- on a product listing where we're auto-pricing against a competitor, set our sale price above that price, and see what price is listed.

Two new tables:
amazon_listing_offers_history
amazon_listing_prices_history

if all prices are exactly the same, do not insert a new row
- agent chat de730f40-136e-4822-a455-8e83d4e9db8b

Add an alerts area for appending new alerts based on a set of conditions
- ability to mark an alert as ignored or resolved

Maybe instead of an alerts area you could also just apply badges to the rows based on the current flags set


alert just appends what was in rule, but shows both the actual value AND the column name, so SKU, column_one_name, column_one_value, condition, column_two_name, column_two_value,

Then do not append a new alert when the conditions are a duplicate of what is already listed.

Temporary:
- add a flags column right after the SKU, in front of Amazon SKU include the Jlab SKU and definition.
- list of flags shown way above^


once an alert is sent, send it once, still allow for acknowledging but only for the users benefit of checking it off. We reference the parent row so that we know not to send a duplicate alert when the alert cron job runs or is called, maybe we just call it in every cron job or maybe it's a cron job of its own, not sure yet. Only send a new alert when the row is updated. reference the alert table chat, probably don't need the ack table if we're just auto ack'ing. Auto list the proble

![[Pasted image 20261001153513.png]]

![[Pasted image 20261001153534.png]]


No conflict? on promo calendar
![[Pasted image 20261009161622.png]]


