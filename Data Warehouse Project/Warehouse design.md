Before ETL we should land source data as-is from that source, do not perform ETL on the raw data.

During the transformation phase we should match rules if necessary to shape the data to the dimension table we're inputting to, so in the case of customers we should match rules like the following:

![[Pasted image 20260806115209.png]]

![[Pasted image 20260806115224.png]]

![[Pasted image 20260806115407.png]]greensocks
gsap



SCD type 2: slow changing dimension tables should keep track of the row effective date, the row expiration date, and whether or not it's current.
- Case pack / address / status → often Type 2.

![[Pasted image 20260806124001.png]]

Working with business leadership partners, rank each business process on business value and feasibility, then tackle processes with the highest impact and feasibility scores fi rst. Although prioritization is a joint activity with the business, your underlying understanding of the organization’s business processes is essential to its eff ectiveness and subsequent actionability.

How do promotions get logged? Where are we saving that information? is there an operational system for storing this information is just stored in a spreadsheet?

