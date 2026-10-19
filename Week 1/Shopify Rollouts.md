A/B testing, existing platforms or if shopify has native solution


Shopify rollouts

**Experiment** is a type of rollout
- Can adjust a subset of eligible visitors that changes display to
	- Treatment
	- Control

Can only add one of each of the following types of changes to a rollout
- replace main theme
	- Add changes to your theme content
- replace checkout and accounts configuration

Shopify rollouts is purely server side so a workaround is to have a hard coded html data variable within the banner itself that can be attached to requests that get sent to the backend, so attach a data variable that says either A or B. And we can measure the clickthrough rates based on their variant, just need to log and aggregate based on variant.







