# Four-Step Dimensional Design Process

## Step 1: Select the Business Process

A **business process** is a low level activity performed by an organization, such as taking orders, invoicing, receiving payments, handling service calls, registering students, performing a medical procedure, or processing claims.

Common characteristics of a business process:
* Frequently expressed as action verbs because they represent activities that the business performs.
* Typically supported by an operational system, like a billing or purchasing system
* Generate or capture key performance metrics
* Usually triggered by an input and result in output metrics.
	* Typically a series of processes in which the outputs from one process become the inputs to the next
		* This series of processes results in a series of fact tables

Organizational business departments or functions **do not** equate to business processes.
- If you design departmentally bound dimensional models, you inevitably duplicate data with diff erent labels and data values.

## Step 2: Declare the Grain

Specifying exactly what an individual fact table row represents
- "How do you describe a single row in the fact table?"

Example:
- One row per scan of an individual product on a customer's sales transaction
- One row per line item on a bill from a doctor
- One row per individual boarding pass scanned at an airport gate
- One row per daily snapshot of the inventory levels for each item in a warehouse
- One row per bank account each month

Whenever possible, **express the grain in business terms**

If the grain isn't clearly defined, the whole design rests on quicksand. If you need to revisit and redefine the grain, then return to this step (step 2) then revisit steps 3 and 4 again.

## Step 3: Identify the Dimensions

Dimensions fall out of the question, "How do business people describe the data resulting from the business process measurement events?" 

You need to decorate fact tables with a robust set of dimensions representing all possible descriptions that take on single values in the context of each measurement.

After defining grain, you can easily identify "who, what, where, when, why, and how" associated with the event.

examples: date, product, customer, employee, facility

With the choice of each dimension, you then list all the discrete, text-like attributes that flesh out each dimension table.

![[images/Pasted image 20260814143525.png|214]]
![[images/Pasted image 20260814143555.png|217]]
![[images/Pasted image 20260814143616.png|220]]

After choosing dimensions like Product or Customer, write down every useful descriptive column (brand, category, city, tier, month name…). Those discrete, text-like fields are the attributes that turn a bare key into a full dimension table.

## Step 4: Identify the Facts

Determined by answering the question, "What is the process measuring?"

All candidate facts in a design must be true to the grain defi ned in step 2. Facts that clearly belong to a different grain must be in a separate fact table.

Typical facts are numeric additive figures, such as quantity ordered or dollar cost amount

We strongly encourage you to resist the temptation to model the data by looking at source data alone. It may be less intimidating to dive into the data rather than interview a business person; however, the data is no substitute for business user input.

# Retail Case Study

Data is collected at several interesting places in a grocery store
- POS system scans product barcodes at the cash register, measuring consumer takeaway at the front door of the grocery store
- Vendors making deliveries is captured at the store's back door

Management is concerned with the logistics of ordering, stocking, and selling products while maximizing profit. 

Profit comes from charging as much as possible for each product, lowering costs for product acquisition and overhead, and at the same time attracting as many customers as possible in a highly competitive environment.

Some of the most significant management decisions have to do with pricing and promotions.

Both store management and headquarters marketing spend a great deal of time tinkering with pricing and promotions. Promotions in a grocery store include temporary price reductions, ads in newspapers and newspaper inserts, displays in the grocery store, and coupons

A 50-cent reduction in the price of paper towels, especially when coupled with an ad and display, can cause the sale of the paper towels to jump by a factor of 10. Unfortunately, such a big price reduction usually is not sustainable because the towels probably are being sold at a loss. ==As a result of these issues, the visibility of all forms of promotion is an important part of analyzing the operations of a grocery store.==

## Step 1

```
NOTE: The first DW/BI project should focus on the business process that is both the most critical to the business users, as well as the most feasible. Feasibility covers a range of considerations, including data availability and quality, as well as organizational readiness.
```

In our retail case study, management wants to better understand customer purchases as captured by the POS system. Thus the business process you’re modeling is POS retail sales transactions. This data enables the business users to analyze which products are selling in which stores on which days under what promotional conditions in which transactions.

## Step 2

```
NOTE: You should develop dimensional models representing the most detailed, atomic information captured by a business process.
```

Less granular model means potentially less detailed dimensions. Not good.

Although aggregated data plays an important role for performance tuning, it is not a substitute for giving users access to the lowest level details; users can easily summarize atomic data, but it’s impossible to create details from summary data

```
NOTE: A DW/BI system almost always demands data expressed at the lowest possible grain, not because queries want to see individual rows but because queries need to cut through the details in very precise ways
```
## Step 3

the POS transaction ticket number is included as a special dimension, as described in the section “Degenerate Dimensions for Transaction Numbers” later in this chapter.

## Step 4

Carefully determine which facts will appear in the fact table, these facts must be true to the grain.

In our example the facts we collect by the POS system include:
- sales quantity
- per unit regular
- discount
- net paid prices
- extended discount
- sales dollar amounts

The extended sales dollar amount equals the sales quantity multiplied by the net unit price.
Likewise, the extended discount dollar amount is the sales quantity multiplied by the unit discount amount.

![[images/Pasted image 20260814151532.png]]

## Derived Facts

You can compute the gross profit by subtracting the extended cost dollar amount from the extended sales dollar amount, or revenue.

==Dimensional modelers sometimes question whether a calculated derived fact should be stored in the database. We generally recommend it be stored physically.==

In this case study, the gross profi t calculation is straightforward, but storing it means it’s computed consistently in the ETL process, eliminating the possibility of user calculation errors. The cost of a user incorrectly representing gross profi t overwhelms the minor incremental storage cost. Storing it also ensures all users and BI reporting applications refer to gross profit consistently

Because gross profit can be calculated from adjacent data within a single fact table row, some would argue that you should perform the calculation in a view that is indistinguishable from the table. This is a reasonable approach if all users access the data via the view and no users with ad hoc query tools can sneak around the view to get at the physical table.

Views are a reasonable way to minimize user error while saving on storage, but the DBA must allow no exceptions to accessing the data through the view. 

However, sometimes non-additive metrics on a report such as percentages or ratios must be computed in the BI application because the calculation cannot be precalculated and stored in a fact table. OLAP cubes excel in these situations.

## Non-Additive Facts

# Date Dimension

## Current and Relative Date Attributes

Most datawarehouses load data daily

## Time of day as a Dimension or Fact

Although date and time are comingled in an operational date/time stamp, time-ofday is typically separated from the date dimension to avoid a row count explosion in the date dimension. As noted earlier, a date dimension with 20 years of history contains approximately 7,300 rows. If you changed the grain of this dimension to one row per minute in a day, you’d end up with over 10 million rows to accommodate the 1,440 minutes per day. If you tracked time to the second, you’d have more than 31 million rows per year! Because the date dimension is likely the most frequently constrained dimension in a schema, it should be kept as small and manageable as possible

==If you want to filter or roll up time periods based on summarized day part groupings, such as activity during 15-minute intervals, hours, shifts, lunch hour, or prime time, time-of-day would be treated as a full-fl edged dimension table with one row per discrete time period, such as one row per minute within a 24-hour period resulting in a dimension with 1,440 rows.==

==If there’s no need to roll up or fi lter on time-of-day groupings, time-of-day should be handled as a simple date/time fact in the fact table.==

# Product Dimension

almost always sourced from the operational product master file.

It is headquarters’ responsibility to defi ne the appropriate product master record (and unique SKU number) for each new product.

## Flatten Many-to-One Hierarchies

Typically, individual SKUs roll up to brands, brands roll up to categories, and categories roll up to departments. Each of these is a many-to-one relationship.

![[images/Pasted image 20260814153439.png]]

For each SKU, all levels of the merchandise hierarchy are well defi ned. Some attributes, such as the SKU description, are unique. In this case, there are 300,000 diff erent values in the SKU description column. At the other extreme, there are only perhaps 50 distinct values of the department attribute. ==Thus, on average, there are 6,000 repetitions of each unique value in the department attribute. This is perfectly acceptable! You do not need to separate these repeated values into a second normalized table to save space.== Remember dimension table space requirements pale in comparison with fact table space considerations.

```
NOTE: Keeping the repeated low cardinality values in the primary dimension table is a fundamental dimensional modeling technique. Normalizing these values into separate tables defeats the primary goals of simplicity and performance, as discussed in “Resisting Normalization Urges” later in this chapter.
```

Product dimension tables typically have more than one explicit hierarchy.

![[images/Pasted image 20260814153805.png]]

## Attributes with Embedded Meaning

==Often operational product codes, identifi ed in the dimension table by the NK notation for natural key, have embedded meaning with diff erent parts of the code representing signifi cant characteristics of the product. In this case, the multipart attribute should be both preserved in its entirety within the dimension table, as well as broken down into its component parts, which are handled as separate attributes. For example, if the fi fth through ninth characters in the operational code identify the manufacturer, the manufacturer’s name should also be included as a dimension table attribute.==

## Numeric Values as Attributes or Facts

You will sometimes encounter ==numeric values that don't clearly fall into either the facto r dimension== attribute categories. A classic example is the standard list price for a product. 

==If a numeric value is used primarily for calculation purposes, it likely belongs in the fact table==

==If the stable numeric value is used predominantly for filtering and grouping, it should be treated as a product dimension attribute==

Sometimes numeric values serve both ==calculation== and ==filtering/grouping functions.== In these cases, ==you should store the value in both the fact and dimension tables.== Perhaps the standard price in the fact table represents the valuation at the time of the sales transaction, whereas the dimension attribute is labeled to indicate it’s the current standard price.

![[images/Pasted image 20260814154918.png]]

![[images/Pasted image 20260814154929.png]]

##### ==Data involved in calculations should be in fact tables and data involved in constraints, groups and labels should be in dimension tables.==
## Drilling Down on Dimension Attributes


