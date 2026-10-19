
# Fundamental Concepts

These techniques must be considered during every dimensional design.

## Gather Business Requirements and Data Realities

Before launching a dimensional modeling effort, the team needs to understand the needs of the business, as well as the realities of the underlying source data. You uncover the requirements via sessions with business representatives to understand their objectives based on key performance indicators, compelling business issues, decision-making processes, and supporting analytic needs. At the same time, data realities are uncovered by meeting with source system experts and doing high-level data profi ling to assess data feasibilities.

## Collaborative Dimensional Modeling Workshops

Dimensional models should be designed in collaboration with subject matter experts and data governance representatives from the business.

## Four-Step Dimensional Design Process

1. Select the business process. 
2. Declare the grain. 
3. Identify the dimensions. 
4. Identify the facts.

- After these steps: Determine the table and column names, sample domain values, and business rules.

## Business Processes

**Business processes** are the operational activities performed by your organization, such as taking an order, processing an insurance claim, registering students for a class, or snapshotting every account each month. 

Business process events generate or capture performance metrics that translate into facts in a fact table.

Choosing the process is important because it defines a specific design target and allows the grain, dimensions, and facts to be declared.

## Grain

The **grain** establishes exactly what a single fact table row represents. 

==The grain must be declared before choosing dimensions or facts because every candidate dimension or fact must be consistent with the grain.==

**Atomic grain** refers to the lowest level at which data is captured by a given business process.
- strongly encouraged to start by focusing on atomic-grained data because it withstands the assault of unpredictable user queries; rolled-up summary grains are important for performance tuning but they pre-suppose the business's common questions.
- Each proposed fact table grain results in a separate physical table; different grains must not be mixed in the same fact table.

## Dimensions for Descriptive Context

**Dimensions** provide the "who, what, where, when, why, and how" context surrounding a business process event.

Dimension tables contain the descriptive attributes used by BI applications for filtering and grouping the facts. 

Dimension tables are sometimes called the "soul" of the data warehouse because they contain the entry points and descriptive labels that enable the DW/BI system to be leveraged for business analysis. 

## Facts for Measurements

Facts are the measurements that result from a business process event and are almost always numeric. A single fact table row has a one-to-one relationship to a measurement event as described by the fact table’s grain. Thus a fact table corresponds to a physical observable event, and not to the demands of a particular report. Within a fact table, only facts consistent with the declared grain are allowed. For example, in a retail sales transaction, the quantity of a product sold and its extended price are good facts, whereas the store manager’s salary is disallowed.

## Star Schemas and OLAP Cubes

**Star Schemas* are dimensional structures deployed in a RDBMS.

## Graceful Extensions to DImensional Models

all the following changes can be implemented without altering any BI queries or applications:

- Facts consistent with the grain of an existing fact table can be added by creating new columns
- Dimensions can be added to an existing fact table by creating new foreign key columns, presuming they don't alter the fact table's grain.
- Attributes can be added to an existing dimension table by creating new columns
- The grain of a fact table can be made more atomic by adding attributes to an existing dimension table, and then restating the fact table at the lower grain, being careful to preserve the existing column names in the fact and dimension tables.

# Basic Fact Table Techniques

## Fact Table Structures

In addition to numeric measures, a fact table always contains foreign keys for each of its associated dimensions, as well as optional degenerate dimension keys and date/time stamps. Fact tables are the primary target of computations and dynamic aggregations arising from queries.

## Additive, Semi-Additive, Non-Additive Facts

Numeric Measures in a fact table fall into three categories:
1. Fully Additive
		Can be summed across any of the dimensions associated with the fact table
2. Semi Additive
		Measures that can be summed across some dimensions, but not all; balance amounts are common semi-additive facts because they are additive across all dimensions except time.
3. Non Additive
		Such as ratios
		Should store the fully additive components of the non-additive measure and sum these components into the final answer set before calculating the final non-additive fact

## Nulls in Fact Tables

Null-valued measurements behave gracefully in fact tables
- Aggregate functions (SUM,COUNT, MIN, MAX, AVG) all behave correctly with NULL facts.

Nulls must be avoided in the fact table's foreign keys because these nulls would automatically cause a referential integrity violation.

## Conformed Facts

If the same measurement appears in separate fact tables, care must be taken to make sure the technical definitions of the facts are identical if they are to be compared or computed together. If the fact definitions are consistent the conformed facts should be identically named; but if they are incompatible, they should be differently named to alert the business users and BI applications.

## Transaction Fact Tables

==A row in a **transaction fact table** corresponds to a measurement event at a point in space and time==

Atomic transaction grain fact tables are the most dimensional and expressive fact tables.

optionally contain time stamps and degenerate dimension keys

## Periodic Snapshot Fact Tables

Summarizes many measurement events occurring over a standard period, such as a day, a week, or a month. ==The grain is the period,== not the individual transaction.

## Accumulating Snapshot Fact Tables

Summarizes the measurement events occurring at predictable steps between the beginning and end of a process. Pipeline or workflow processes, such as order fulfillment or claim processing, that have a dignified start point, standard intermediate steps, and defined endpoint can be modeled with this type of fact table.

pg 44 for more info

## Factless Fact Tables

It's possible that the event merely records a set of dimensional entities coming toegether at a moment in time.

## Aggregate Fact Tables or OLAP Cubes

Aggregate fact tables are simple numeric rollups of atomic fact table data built solely to accelerate query performance. These aggregate fact tables should be available to the BI layer at the same time as the atomic fact tables so that BI tools smoothly choose the appropriate aggregate level at query time.

Aggregate fact tables contain foreign keys to shrunken conformed dimensions, as well as aggregated facts created by summing measures from more atomic fact tables.

## Consolidated Fact Tables

It is often convenient to combine facts from multiple processes together into a single consolidated fact table ==if they can be expressed at the same grain.==

# Basic Dimension Table Techniques

## Dimension Table Structures

Has a single primary key column that is embedded as a foreign key in any associated fact table where the dimension rows' descriptive context is exactly correct for that fact table row.

Denormalized tables with low-cardinality text attributes.

Operational codes and indicators can be treated as attributes, but verbose descriptions are best.

# Dimension Surrogate Keys

The primary key of a dimension table cannot be the operational system's natural key because there will be multiple dimension rows for that natural key when changes are tracked over time.

Create anonymous integer primary keys for every dimension. Use incrementing integer keys here.

## Natural, Durable, and Supernatural Keys

A durable key is a key that does not change to reflect something that is functionally the same bust has changed over time and thus must have a unique identifier for each row of that "same thing" and a durable key correlating the two rows as one/

## Drilling Down

The fundamental way data is analyzed by business users, it's done by adding a row header to an existing query, it's a dimension attribute appended to the GROUP BY expression in a sql query.

## Degenerate Dimensions

This degenerate dimension is placed in the fact table with the explicit acknowledgment that there is no associated dimension table. Degenerate dimensions are most common with transaction and accumulating snapshot fact tables.

## Denormalized Flattened Dimensions

In general, dimensional designers must resist the normalization urges caused by years of operational database designs and instead denormalize the many-to-one fixed depth hierarchies into separate attributes on a fl attened dimension row.

## Multiple Hierarchies in Dimensions

Seperate hierarchies can gracefully coexist in the same Dimension table, for example calendar date dimensions can have a day to week to fiscal period hierarchy.

## Flags and Indicators as Textual Attributes

Cryptic abbreviations, true/false flags, and operational indicators should be supplemented in dimension tables with full text words that have meaning when independently viewed.

## NULL Attributes in Dimensions

Recommended substituting a descriptive string such as unknown or Not Applicable in place of the null value. 

Nulls in dimension attributes should be avoided because different databases handle grouping and constraining on nulls inconsistently

## Calendar Date Dimensions

pg 49

## Role-Playing Dimensions

When you reference the same physical dimension table multiple times within a fact table, give the dimension table an alias that references the use of each reference of that table. For example 2 dates one can be aliased as ship date and one is the purchase date

## Junk Dimensions


## Snowflaked Dimensions

A flattened denormalized dimension table contains exactly the same information as a snowflaked dimension. Do not normalize dimensions, just use hierarchies. Snowflakes bad

## Outrigger Dimensions

When a dimension table references another dimension table, such as a back account dimension table that has a reference to the date when the bank account was opened. These should be used sparingly, and in most cases should just be part of the fact table.

# Integration via Conformed Dimensions

Defining a simple but powerful recipe for integrating data from different business processes

## Conformed Dimensions

Conformed dimensions are shared, identically defined attributes (like `brand` or `month_name`) reused across fact tables so you can build one report that lines up metrics from different processes on the same rows.

## Shrunken Dimensions

These are conformed dimensions that are a subset of rows and or columns of a base dimension. Shrunken Rollup dimensions are required when constructing aggregate fact tables. Also necessary for business processes that naturally capture data at a higher level of granularity.

## Drilling Across

Making separate queries against two or more fact tables where the row headers of each query consist of identical conformed attributes. The answer sets from the two queries are aligned by performing a sort-merge operation on the common dimension attribute row headers.

## Value Chain

A **value chain** identifies the natural flow of an organization's primary business process.

## Enterprise Data Warehouse Bus Architecture

Decomposes the DWI/BI planning process into manageable pieces by focusing on business processes, while delivering integration via standardized conformed dimensions that are reused across processes.

Agile implementations corresponding to the rows of the bus matrix.


## Enterprise Data Warehouse Bus Matrix

The main deliverable of the data warehouse as described above. The rows of the matrix are business processes and the columns are dimensions.

Implement one row of the matrix at a time.

## Detailed Implementation Bus Matrix

A more granular bus matrix where each business process row has been expanded to show specific fact tables. At this level of detail, the precise grain statement and list of facts can be documented.

## Opportunity/Stakeholder Matrix

The second deliverable of the data warehouse.

After the enterprise data warehouse bus matrix rows have been identified, you can draft a different matrix by replacing the dimension columns with business functions, such as marketing, sales, and fi nance, and then shading the matrix cells to indicate which business functions are interested in which business process rows. The opportunity/stakeholder matrix helps identify which business groups should be invited to the collaborative design sessions for each process-centric row.

# Dealing with Slowly Changing Dimensions Attributes

How to deal with SCD attributes. It is common to have attributes in the same dimension table th are handled with different change tracking techniques.

## Type 0: Retain Original

With **type 0** the dimension attribute value never changes, so facts are always grouped by this original value. Appropriate for any attribute labeled "original", e.g. a customer's original credit score or a durable identifier. Applies to most attributes in a date dimension

## Type 1: Overwrite

This technique destroys history, the old attribute value in the dimension row is overwritten with the new value.

## Type 2: Add New Row

Required generalizing the primary key of the dimension beyond the natural or durable key because there will potentially be multiple rows describing each member

A primary surrogate key must be assigned to new rows created for a dimension member and be used as a foreign key in all fact tables from the moment of the update until a subsequent change creates a new dimension key and updated dimension row.

Required columns if using type 2:
1. Row effective date or date/time stamp
2. Row expiration date or date/time stamp
3. current row indicator (aka is this the currently used row)

## Type 3: Add New Attribute

# Dealing with Dimensions Hierarchies

## Fixed Depth Positional Hierarchies

# Advanced Fact Table Techniques

