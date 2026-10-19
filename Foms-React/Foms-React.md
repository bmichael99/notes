
##Libraries Used:

- **Tanstack Tables**
	- Reason: 
- **Tanstack Query**
https://react.dev/learn/build-a-react-app-from-scratch#data-fetching
	- Reason: Standardizes data fetching behavior to help maintain clean code. This is not a "necessary" library, however it does help with caching and maintaining best practices for loading and error states.
- **React Router** - Migrate to Tanstack Router? 
https://react.dev/learn/build-a-react-app-from-scratch#routing
	- Reason: Easy to use router library, at least when just working with the "Data" and "Declarative" modes as defined in the react router documentation. See https://reactrouter.com/start/modes#data for more information. 
- **Zod**
	- Schema validation for API's and type definition derived from schema's
	- Catches misalignments between front and back-end
- **React Hook Form**
	- 
	

include the shipment ID in the URL so that you can determine which shipment you're referring to from a fulfillment order, since fulfillment orders return an array of shipments from the backend

React Frontend and hit PHP backend endpoints
- replicate FOMS


requirements:
- active orders
- open order statistics
-  auth?
	- sso

TODO:
- ~~Hit endpoint for only active orders, not all orders
- Migrate to tanstack query
	- just let AI do this
	- Mostly done just need to migrate client.tsx to use tanstack query as well.
- Audit and refactor previously written AI code, rewrite API Client
- Fix formatting issues when going through table, set a fixed length for each column and cut off extra characters
- Add sorting to table
	- partially done
- ~~Add search to table
- ~~Add extra pagination features for how many rows to display per table page
- show information specific to each fulfillment ID when clicking on a specific order
- Improve docs of react-frontend
	- inline docs
- Build out per order functionality to mimic that of the playground
	- Do this step by step to improve maintainability and get a better understanding of the FOM's system, letting AI one-shot this will lead to tech debt and little understanding of the underlying systems.
- Make Source, Destination, Address card components abstracted into one module with things that need to be different as inputs
- Dedicated error page for redirecting back to main dashboard page
- Implement Zod to handle schema validation from incoming rest API data,
	- Just let AI handle updating the schemas
- The edit page for address, Line Items, and Shipments should just be a form template with information passed in such as the put endpoint, the title, and what fields to include.
	- Needs to have the ability to insert children as well as there are custom children in some of these components as well.
	- Implement Zod before this
- Add back delete shipment button
- Currently when you load the edit shipments page you would need to pass in the shipment data from fullment_order/${id}
- Make a duplicate of the database and run a dev server so I can test update endpoints



- Update Timeline Comment section to include an image banner based on the current auth'd user, currently hard coded as a question mark.
- Refactor whatever the fuck this is in fetchCurrentUser and other fetch functions
```
const apiBase = import.meta.env.VITE_API_BASE ?? '/foms/api/'
const url = `${apiBase.replace(/\/?$/, '/')}user/me`
```


**Instructions for AI**:
Add simple in-line documentation to the selected area that explains in a very short and readable format what each code-block does for long term maintainability.


find cheaper token provider
freight forwarder address
fraud detection
order for anything less than $5 def fraud
Route fraud , does this look like a freight forwarder address
Buying from out of country like columbia then u order through a freight forwarder, from the freight forwarder to columbia
Build out better analysis for detecting fraud, we rarely find things that aren't already recognized by shopify

An obvious sign of fraud is more than 1 order where a customer is returning their items

Letting n8n handle classification of fraud detection, give better updates into the teams IT channel.

We can route all shipments through this and determine whether or a shipment should be put on hold or show a little warning on foms about a possible fraudulent order.

Already have existing n8n setup doing this

CS uses gorgeous for all their comms, emails, CS related stuff.


