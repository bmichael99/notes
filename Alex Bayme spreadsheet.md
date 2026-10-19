If you get some free time can you look into a good way for Alex Bayme to upload the following spreadsheet somewhere (monthly) which contains all our forecast estimates on a monthly basis for both Units and Dollars? The goal is to get it into the FINANCE.FORECAST snowflake schema/db and eventually we will use this data for the metabase dashboard.

https://jlabaudio-my.sharepoint.com/:x:/g/personal/joeyb_jlab_com/IQBdiOC5PAmMRo5NhFqvH9MwAf8sNsYoGz8F2aocti0p2w8?rtime=YGVuJ9r-3kg




Flow:
1. Upload CSV on playground, custom page under finance
2. Check the current month based Time column
3. Second path for if month already exists: natural key of time item and customer, upsert on that composite key
4. Bulk load all rows at once
	1. snowflake API, bulk load


taking netsuite order data from amazon, remitance data, financial reports data with line items.
here's the order id, here are the records within netsuite

different sources all pointing into snowflake


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