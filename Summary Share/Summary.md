A Star Schema is a multi-dimensional data model used to organize data in a database so that it is easy to understand and analyze. It is made of Fact and Dimension tables.

![Example 1](images/Pasted%20image%2020260805150545.png)

Each business process will be a fact table that can be identified by their operational source. An operational source is just a data source, like netsuite, logiwa, etc - wherever the data originates. 

A fact table is data from an operational source, it stores the performance measurements resulting from an organizations business process events. Ideally you store this at the lowest level of granularity and then you can build summary tables later if needed. Measurements are like amount, quantity, duration etc. Examples of fact tables just for website analytics would be fact_clicks, fact_page_view, fact_session_seconds. 

A dimension table describes the who, what, when, where (customer_id, product_id, date_id, etc). Many dimensions will be shared conformed dimensions meaning its a single dimension table that is reused across fact tables/business processes while maintaining the exact same structure, keys, and business meaning. A dimension is referenced in a fact table by key. This makes it easy to reference data across tables, such as referencing the same customer or product across fact tables.

Good summary: https://www.databricks.com/blog/what-is-star-schema


![Example 2](images/Pasted%20image%2020260805151352.png)

## Enterprise DataWarehouse Bus Matrix

Deliverable:
	Enterprise data warehouse bus matrix
		it is often helpful to create a list of core dimensions before filling in the matrix to assess whether a given dimension should be associated with a business process. The number of bus matrix rows and columns varies by organization.
			is a hybrid resource that serves multiple purposes, including architecture planning, database design, data governance coordination, project estimating, and organizational communication.
				defines the overall data architecture for the DW/BI system.

![Enterprise data warehouse bus matrix](images/Pasted%20image%2020260806143538.png)

![Star schema detail](images/Pasted%20image%2020260909132636.png)

The organization's business processes are represented as matrix rows.
Remember: **You are identifying business processes, not the organization's business departments.**

The matrix rows translate into dimensional models representing the organization’s primary activities and events, which are often recognizable by their operational source.

When it’s time to tackle a DW/BI development project, start with a single business process matrix row because that minimizes the risk of signing up for an overly ambitious implementation.

Focusing on the results of a single process, often captured by a single underlying source system, reduces the ETL development Risk
- Risk: biting off too much ETL system design and development.

Business processes are recognizable by their operational source.

Although dimensional models that cross processes can be immensely beneficial in terms of both query performance and ease of use, they are typically more difficult to implement because the ETL effort grows with each additional major source integrated into a single dimensional model.

It is prudent to focus on the individual processes as building blocks before tackling the task of consolidating.

The columns of the bus matrix represent the common dimensions used across the enterprise. It is often helpful to create a list of core dimensions before filling in the matrix to assess whether a given dimension should be associated with a business process. The number of bus matrix rows and columns varies by organization. For many, the matrix is surprisingly square with approximately 25 to 50 rows and a comparable number of columns. In other industries, like insurance, there tend to be more columns than rows


