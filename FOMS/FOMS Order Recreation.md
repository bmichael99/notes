This page allows the user to take a [[JLab OMS (FOMS)]] order and recreate it in an [[FOMS OrderDestination]] (just Shopify for now).

The page will take in a [[FOMS FulfillmentOrder|FulfillmentOrder]] by id and load its data. It will then show the [[FOMS ShippingInfo|ShippingInfo]] and [[FOMS FulfillmentOrderLineItem|LineItems]] and allow the user to confirm or edit the data.

It will list all Shopify [[FOMS OrderDestination|OrderDestinations]], allowing the user to select which Shopify storefront they want to send the order to.
# Line items
This section will allow the user to select a new line item. It should open a modal that shows SKUs in an autocomplete and also shows the available inventory.