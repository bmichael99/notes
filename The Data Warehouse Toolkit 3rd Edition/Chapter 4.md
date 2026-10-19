# Enterprise Data Warehouse Bus Architecture

Incremental approach to build the enterprise's warehouse.
## Understanding the Bus Architecture

The way in which everything connects from and which everything derives power

![[images/Pasted image 20260806135400.png]]

All the processes of an organization's value chain create a family of dimensional models that share a comprehensive set of common, conformed dimensions [[Chapter 1#Conformed-Dimensions]] . 

![[images/Pasted image 20260806135753.png]]

The bus architecture enables DW/BI managers to get the best of both worlds. They have an architectural framework guiding the overall design, but the problem has been divided into bite-sized business process chunks that can be implemented in realistic time frames. ==Separate development teams follow the architecture while working fairly independently and asynchronously.==

## Enterprise DataWarehouse Bus Matrix

Deliverable:
	Enterprise data warehouse bus matrix
		it is often helpful to create a list of core dimensions before filling in the matrix to assess whether a given dimension should be associated with a business process. The number of bus matrix rows and columns varies by organization.
			is a hybrid resource that serves multiple purposes, including architecture planning, database design, data governance coordination, project estimating, and organizational communication.
				defines the overall data architecture for the DW/BI system.

![[images/Pasted image 20260806143538.png]]

The organization's business processes are represented as matrix rows.
Remember: ==You are identifying business processes, not the organization's business departments.==

The matrix rows translate into dimensional models representing the organization’s primary activities and events, which are often recognizable by their operational source.

When it’s time to tackle a DW/BI development project, start with a single business process matrix row because that minimizes the risk of signing up for an overly ambitious implementation.

Focusing on the results of a single process, often captured by a single underlying source system, reduces the ETL development Risk
- Risk: biting off too much ETL system design and development.

Business processes are recognizable by their operational source.

Although dimensional models that cross processes can be immensely beneficial in terms of both query performance and ease of use, they are typically more difficult to implement because the ETL effort grows with each additional major source integrated into a single dimensional model.

It is prudent to focus on the individual processes as building blocks before tackling the task of consolidating.

The columns of the bus matrix represent the common dimensions used across the enterprise. It is often helpful to create a list of core dimensions before filling in the matrix to assess whether a given dimension should be associated with a business process. The number of bus matrix rows and columns varies by organization. For many, the matrix is surprisingly square with approximately 25 to 50 rows and a comparable number of columns. In other industries, like insurance, there tend to be more columns than rows

### Opportunity/Stakeholder matrix

Leverages the same business process rows, but replaces the dimension columns with business functions, such as merchandising, marketing, store operations, and finance. 

This matrix indicates which business functions are interested in which business processes (and projects)
- Also identifies which groups need to be invited to the detailed requirements, dimensional modeling, and BI application specification meetings after a process-centric row is queued up as a project.

![[images/Pasted image 20260806151408.png]]
### Common Bus Matrix Mistakes

Struggle with the level of detail expressed by each row, resulting in the following missteps:

- Departmental or overly encompassing rows.
	- Matrix Rows shouldn't just match one department, one department probably represents many rows
- Report-centric or too narrowly defined rows
	- Shouldn't be a laundry list of requested reports, a single business process supports numerous analyses; the matrix row should reference the business process, not the derivative reports or analytics.

Architects also naturally fall into the similar traps of defining columns that either too broad or too narrow:

- Overly generalized columns
	- A "person" column is too general, from internal employees to external suppliers and customer contacts. Virtually zero overlap between these populations. Don't make a single dimension for this.
	- Similarly, it’s not beneficial to put internal and external addresses referring to corporate facilities, employee addresses, and customer sites into a generic location column in the matrix.
- Separate Columns for each level of a hierarchy
	- Columns of the bus matrix should refer to dimensions at their most granular level.
		- e.g. use a single column for dates even if different processes want their data is groups of weeks vs days. To express levels of detail above a daily grain, you can denote the granularity within the matrix cell, you could also subdivide the date column to indicate the hierarchical level associated with each business process row.

### Retrofitting Existing Models to a Bus Matrix

Do not build separate dimensional models that ignore a framework tying them together. 

It is unacceptable to build separate dimensional models that ignore a framework tying them together. Isolated, independent dimensional models are worse than simply a lost opportunity for analysis. 

![[images/Pasted image 20260806152837.png|418]]

# Conformed Dimensions

==Conformed dimensions should be built once in the ETL system and then replicated either logically or physically throughout the enterprise DW/BI environment.== When built, it’s extremely important that the DW/BI development teams take the pledge to use these dimensions. It’s a policy decision that is critical to making the enterprise DW/BI system function; their usage should be mandated by the organization’s CIO.

## Drilling Across Fact Tables

Revisit this for combining performance measurements from different business processes in a single report

## Identical Conformed Dimensions

Conformed Dimensions mean the same thing with every possible fact table to which they are joined. A date dimension table connected to the sale facts is identical to the date dimension table connected ot the inventory facts.
- have consistent dimension keys, attribute column names, attribute definitions, attribute values (which translate into consistent report labels and groupings).

Dimension attributers don't conform if they're called Month in one dimension and Month Name in another

Identical conformed dimensions in two dimensional models may be the same physical table within the database. it is more likely that the dimension is built once in the ETL system and then duplicated synchronously outward to each dimensional model. 

In either case, the conformed date dimensions in both dimensional models have the same number of rows, same key values, same attribute labels, same attribute data defi nitions, and same attribute values. Attribute column names should be uniquely labeled across dimensions.

## Shrunken Rollup Conformed Dimension with Attribute Subset

Use shrunken rollup conformed dimensions when two facts don't use the same level of granularity or where granularity differs 

Attributes that are common to both the detailed and rolled-up dimension tables, such as the brand and category descriptions, should be labeled, defi ned, and identically valued in both tables, as illustrated in Figure 4-13. However, the primary keys of the detailed and rollup dimension tables are separate.

![[images/Pasted image 20260806164038.png]]

