## Before
- I need a verified list of columns that Walmart would need for billed overage invoice
- I also need to verify that data is being extracted from the correct sources + fields.
	- Who is a good point of reference for verifying that data is being extracted from the correct sources + fields.
- Make sure we get a response/remittance back from Walmart so that we know what invoiced successfully.

What's an 810? What does a 2 hr billable SOW (statement of work?) entail, is that for SPS?


## During
Nathan getting a list of what walmart requires
- nvm

we need to be able to support target as well?
- if no api then we'll find a way to generate an upload/download o the overage report

high radius download for getting what was paid

Amit is our implementation guy on the SPS side

Don't want to do this in netsuite because it creates an additional invoice

go with JSON or XML file, we will drop into an FTP or API connection, then amit will grab that and submit it to the retailer

format of the file type

need a list of the mapping for that customer

walmart first

walmart's required fields, map that to the JSON

will we be able to link to the PO?
- yes we do link it to the PO it's missing from
- they process it within their system

will get json format and required fields

API is preferred, we would need SPS credentials tokens and folder to drop the files into. They will run the automation for files to get dropped in there and send to walmart. Response for success

2 hour SOW per retailer

scott said: maybe ask jason, if we have no PO, then we create it by going through the data we received from walmart. 
## After