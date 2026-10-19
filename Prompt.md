Build only the behavior described. Leave out extra fields, joins, and paths that the feature does not use.
Keep the code small. Remove anything that is not needed for that behavior.
Give every block one clear purpose, and label it with a short comment that states that purpose.
Write it so a beginner can read it straight through: plain steps, obvious names, no extra layers.

Instructions:



go sport, entire family, go sport +, 

Run the merchant listing sync to live smoke test the new tables, report the results back to me

GO LUX ANC
FLEX OPEN
JBUD SPORT
JBUD SPORT+

Implement a new feature under "new promo", this new feature allows the user to auto approve and apply to amazon. This should approve all these rows in the database, and then apply them to amazon after approving in the database if the approval was successful. If any of this doesn't make sense or should be improved, say so. Be critical of the implementation and where things can and should be improved.


pricer result shouldn't just say "skip, holding the buy box" because it also checks the cap to see if there's a price to move to right? or are we just skipping when we hold buy box?

![[Pasted image 20261012112749.png]]
![[Pasted image 20261012112753.png]]



Build only the behavior described. Leave out extra fields, joins, and paths that the feature does not use.

Keep the code small. Remove anything that is not needed for that behavior.

Give every block one clear purpose, and label it with a short comment that states that purpose.

Write it so a beginner can read it straight through: plain steps, obvious names, no extra layers.



Instructions:

Add an endpoint and UI for the following behavior. Hovering over a sale price shows a strikethrough, and we can click it to wipe the sale price in amazon, make sure to do a browser confirm(), show a toast for a successful 200 response from amazon or if it failed. Change the sync selected button to be "bulk actions" and be a drop down, with the options of Sync selected, and an option for wiping sale price on selected (in amazon). This doesn't touch promo calendar.