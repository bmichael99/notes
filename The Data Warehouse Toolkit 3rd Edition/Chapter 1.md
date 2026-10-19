Information
- Operation record keeping
- analytical decision making

Data in -> **Operational systems**
- Turn the wheels of an organization
- Typically do not maintain history, but rather update data to reflect the most current state.
- E.g. take orders, sign up new customers, monitor status of operational activities, log complaints, etc.
- Shopify/Logiwa/Netsuite for our case at Jlab
Data out -> **DW/BI System**
- Watch the wheels of the organization turn to evaluate performance.
- e.g. count new orders and compare them with last week's orders, ask why the new customers signed up, what the customer's complained about, etc.
- Take a large set of data over a set of time and compress that data into an answer set.

## Goals of Data Warehousing and Business Intelligence 

The DW/BI system must make information easily accessible
- The data must be intuitive and obvious to the business user, not merely the developer.
- Data's structures/labels should mimic the business users' thought process and vocabulary
- simple and fast

The DW/BI system must present information consistently.
- Data must be carefully assembled from a variety of sources, cleansed, quality assured, and released only when it is fit for user consumption
- Consistency implies that common labels and definitions for the DW/BI system's contents are used across data sources. If two performance measure have the same name, they must mean the same thing

The DW/BI system must adapt to change
- Existing data and applications should not be changed when the business community asks new questions or new data is added to the warehouse.
- If descriptive data in the DW/BI system must be modified, you should account for the changes and make these changes transparent to the users.

## Dimensional Modeling Introduction

Dimensional modeling is widely accepted as the preferred technique for presenting analytic data because it addresses two simultaneous requirements:
- Delivers data that is understandable to the business users
- Deliver fast query performance

Dimensional modeling is a longstanding technique for making databases simple.
- Simplicity is critical because it ensures that users can easily understand the data, as well as allows software to navigate and deliver results quickly and efficiently.

## Normalized Models vs Dimensional Models

We don't use normalized 3NF structures in a dimensional model. Normalizing to this degree will lead to a spider web of tables that is difficult to navigate and will be disastrous for query performance. The use of normalized modeling in the DW/BI presentation area defeats the intuitive and high-performance retrieval of data.

A dimensional model contains the same information as a normalized model, but packages the data in a format that delivers user understandability, query performance, and resilience to change. 

## Star Schemas vs OLAP Cubes

Dimensional models implemented in relational database management system (RDBMS) are referred to as **star schemas** because of their resemblance to a star-like structure.

Dimensional models implemented in multidimensional database environments are referred to as **online analytical processing (OLAP) cubes** 

OLAP Cube
- When data is loaded into an OLAP cube, it is stored and indexed using formats and techniques that are designed for dimensional data.
- Performance aggregations or precalculated summary tables are often created and managed by the OLAP cube engine.
- Cubes deliver superior query performance because of the precalculations, indexing strategies, and other optimizations.

Recommended that detailed atomic information be loaded into a star schema, optional OLAP cubes are then populated from the star schema. 

A star schema hosted in a relational database is a good physical foundation for building an OLAP cube, and is generally regarded as a more stable basis for backup and recovery.

It can be difficult to port BI applications between different OLAP tools than to port BI applications across different relational databases.

## Fact Tables for Measurements

A **fact table** in dimensional model stores the performance measurements resulting from an organizations business process events. 
- You should strive to store the low-level measurement data resulting form a business process in a single dimensional model.
- Measurements are like amount, quantity, duration, etc.
	- Provide foreign keys to dimension tables that explain who, what, when, where (customer_id, product_id, date_id, etc)
![[images/Pasted image 20260805142151.png]]
![[images/Pasted image 20260805142216.png]]

Measurement data should not be replicated in multiple places for multiple organizational functions around the enterprise.
- All users should reference the same source of truth, our fact table

**fact** represents a business measure. Each row in a fact table corresponds to a measurement event. The data on each row is at a specific level of detail, referred to as the **grain**
- e.g. one row per product sold on a sales transaction.

All measurement rows in a fact table must be at the same grain, this ensures that measurements are inappropriately double-counted.

![[images/Pasted image 20260805142538.png]]

A **textual measurement** is a description of something and is drawn from a discrete list of values.
- The designer should make every effort to put textual data into dimensions where they can be correlated more effectively with the other textual dimension attributes and consume much less space. ==You should not store redundant textual information in fact tables.==

A true **text fact**, that is a textual piece of data that is placed into a fact table is rare because the unpredictable content of a text fact, like a freeform text comment, makes it nearly impossible to analyze.
![[images/Pasted image 20260805143459.png|216]]

Fact tables normally fall into grains of three categories:
1. Transaction
	1. most common
2. Periodic Snapshot
3. Accumulating Snapshot

**Referential Integrity** means that keys in the fact table correctly match their respective primary keys in the corresponding dimension tables.
- You access the fact table via the dimension tables joined to it

Fact tables generally have a primary key composed of a subset of the foreign keys (**Composite Key**), every table that has a composite key is a fact table. **Fact tables express many-to-many relationships**, all others are dimension tables.

## Dimension Tables for Descriptive Context

Contain the textual context associated with a business process measurement event.

Describe the "who, what, where, when, how, and why" associated with the event

Single primary key only

**Dimension attributes** serve as the primary source of query constraints, groupings, and report labels. 
- vital to making the DW/BI system usable and understandable.

Attributes should consist of real words and not cryptic abbreviations
- minimize the use of codes, replace them with more verbose textual attributes

In many ways, the data warehouse is only as good as the dimension attributes; ==the analytic power of the DW/BI environment is directly proportional to the quality and depth of the dimension attributes.==
All highly beneficial to improve the power of the DW/BI environment:
- Time spent provided attributes with verbose business terminology
- Time spent populating the domain values in an attribute column
- Time spent ensuring the quality of the values in an attribute column

![[images/Pasted image 20260805145703.png]]

To determine whether a numeric data element is a fact or dimension attribute ask whether the column is a measurement that takes on lots of values and participates in calculations (making it a fact) or is a discretely valued description that is more or less constant and participates in constraints and row labels (making it a dimensional attribute). 
- e.g. the cost of a standard cost for a product seems like a constant attribute of the product but may be changed so often that you decide it is more like a measured fact.

Dimension tables often represent hierarchical relationships. For example, products roll up into brands and then into categories.
![[images/Pasted image 20260805150326.png]]

You should resist the urge to normalize data by storing only the brand code in the product dimension and creating a separate brand lookup table.

## Facts and Dimensions Joined in a Star Schema

![[images/Pasted image 20260805150545.png]]

First constrain the heavily indexed dimension tables, and then attack the fact table all at once with the Cartesian product of the dimension table keys satisfying the user's constraints.
	The optimizer can evaluate arbitrary n-way joins to a fact table in a single pass through the fact table's index using this aproach

![[images/Pasted image 20260805151352.png|145]]

# Kimball's DW/BI Architecture

Four separate and distinct components to consider in the DW/BI environment:
1. Operational source systems
2. ETL system
3. Data presentation Area
4. Business Intelligence Applications

## Operational Source Systems

These are the operational systems of record that capture the business's transactions. Think of the source systems as outside the data warehouse because presumably you have little or no control over the content and format of the data in these operational systems.

Data that you access from these systems is normally done in a standardized way every time, unlike the way your data warehouse will be interacted with.

A good data warehouse should relieve the source systems of much of the responsibility for representing the past.

![[images/Pasted image 20260805152353.png]]

## Extract, Transformation, and Load System

The ETL system of the DW/BI environment consists of a:
1. Work area
2. Instantiated data structures
3. Set of processes

ETL is everything between the Operational Source systems and the DW/BI presentation area.

ETL systems and techniques are expanded upon in [[Chapter 19]]: ETL subSystems and Techniques

**Extraction** is the first step of getting data into the data warehouse environment. 
- reading and understanding the source data and copying the data needed into the ETL system for further manipulation. At this point the data belongs to the data warehouse.

**Transforming**: Cleansing the data
- correcting misspellings, resolving domain conflicts, dealing with missing elements, or parsing into standard formats, combining data from multiple sources, and de-duplicating data.
	- These activities can be architected to create diagnostic metadata, to improve data quality in the source systems over time.

**Loading**: 
Have to make the decision between loading the data into physical normalized structures prior to loading into the presentation area's dimensional structures for querying and reporting. The ETL system is typically dominated by the simple activities of sorting and sequential processing. The ETL system may not be based on relational technology but may rely on a system of float files. After validating the data for conformance with the defined one-to-one and many-to-one business rules, it may be pointless to take a final step of building a 3NF physical database before transforming the data once again into denormalized structures for the BI presentation area.

The creation of both normalized structures for the ETL and dimensional structures for presentation means that the data is potentially extracted, transformed, and loaded twice. 
- This two-step process requires more time and investment for the development, more time for the periodic loading or updating of data, and more capacity to store the multiple copies of data.

The focus should be on developing a dimensional presentation area that supports improved business decision making. Although enterprise-wide data consistency is a fundamental goal of the DW/BI environment, there may be effective and less costly approaches than physically creating normalized tables in the ETL system, if these structures don't already exist.

![[images/Pasted image 20260805153859.png]]

TODO:
Make the decision between having a normalized database for supporting the ETL process or whether the data should be transformed straight into the dimensional structures.
- it is often better to have a normalized database that reflects the state of the raw data and then transform that data (ETL) into the star schema.
## Presentation Area to Support Business Intelligence

The DW/BI presentation area is where data is organized, stored, and made available for direct querying by users, report writers, and other analytical BI applications.

The back room ETL system is off-limits. The presentation area is the DW/BI environment as far as the business community is concerned. It is all the business sees and touches via their access tools and BI applications.

Although the presentation area also may contain performance-enhancing aggregated data, it is not sufficient to deliver these summaries without the underlying granular data in a dimensional form. 

The most finely grained data must be available in the presentation area so that users can ask the most precise questions possible. 

![[images/Pasted image 20260805155825.png]]

Dimensional models should correspond to physical data capture events; they should not be designed to deliver the report-of-the-day. 

In other words, you should construct a single fact table for atomic sales metrics rather than populating separate similar, but slightly different, databases containing sales metrics for sales, marking, logistics, and finance teams.

All the dimensional structures must be built using common, conformed dimensions. See [[Chapter 4]] for more details.
- When dimensional models have been designed with conformed dimensions, they can be readily combined and used together. ==The presentation area in a large enterprise DW/BI solution ultimately consists of dozens of dimensional models with many of the associated dimension tables shared across fact tables.==

Using the Bus architecture [[Chapter 4]] is the secret to building distributed DW/BI systems. When the bus architecture is used as a framework, you can develop the enterprise data warehouse in an agile, decentralized, realistically scoped, iterative manner.

![[images/Pasted image 20260805161311.png]]

## Business Intelligence Applications

BI application loosely refers to the range of capabilities provided to business users to leverage the presentation area for analytic decision making.
- query the data in the DW?BI presentation area

## Restaurant Metaphor for the Kimball Architecture

The backroom ETL system must be laid out and architected long before any data is extracted from the source. The ETL system is also highly concerned about data quality, integrity, and consistency.
- Incoming data is checked for reasonable quality as it enters
- Conditions are continually monitored to ensure ETL outputs are of high integrity.
- Business rules to consistently derive value-add metrics and attributes are applied once by skilled professionals in the ETL system rather than relying on each patron to develop them independently.

![[images/Pasted image 20260805162116.png]]

## Data Presentation and BI in the Front Dining Room

==Like the restaurant, the DW/BI system provides "menus" to describe what's available via metadata, published reports, and parametrized analytic applications.== The DW/BI patrons expect consistency and high quality. The presentation area's data must be properly prepared and safe to consume.

Data must be delivered in a form that is appealing to the business user or BI application developer.

You must prevent an unhappy ending by managing the restaurant proactively, make sure the kitchen is properly organized and utilized to deliver as needed to presentation area's food, decor, service, and cost.

# Alternative DW/BI Architectures

DW/BI deliverables should be broadly accepted by the business to make better more informed decisions. The architecture should merely be a means to this objective. 

## Independant Data Mart Architecture

Analytic data is deployed on a departmental basis without concern to sharing and integrating information across the enterprise. Typically a single department identifies requirements for data from an operational source system, then the department works with IT staff to construct a database that satisfies their departmental needs, reflecting their business rules and preferred labeling. Working in isolation, this departmental data mart addresses the department's analytic requirements.

This is not ideal, due to possible data mis-alignment, and mis-match of business rules and labeling.

## Hub-and-Spoke Corporate Information Factory Inmon Architecture

![[images/Pasted image 20260805163655.png]]![[images/Pasted image 20260805163813.png]]

## Hybrid Hub-and-Spoke and Kimball Architecture

don't care

# Dimensional Modeling Myths

The secret to query flexibility is building fact tables at the most granular level. Dimension models that deliver only summary data are bound to be problematic.
- The correct starting point for your dimensional models is to express data at the lowest detail possible for maximum flexibility and extensibility.

## Conformed-Dimensions
==Conformed-dimensions are built and maintained as centralized, persistent master data in the ETL system and then reused across dimensional models to enable data integration and ensure semantic consistency.== 

==Data integration depends on standardized labels, values, and definitions. It is hard work to reach organizational consensus and then implement the corresponding ETL rules, but you can’t dodge the effort, regardless of whether you’re populating normalized or dimensional models.== 

==Presentation area databases that don’t adhere to the bus architecture with shared conformed dimensions lead to standalone solutions. You can’t hold dimensional modeling responsible for organizations’ failure to embrace one of its fundamental tenets.==

![[images/Pasted image 20260806125146.png]]

# More reasons to think dimensionally


# Agile Considerations

Incremental development work can produce components of the framework until sufficient functionality is available and then released to the business community.

