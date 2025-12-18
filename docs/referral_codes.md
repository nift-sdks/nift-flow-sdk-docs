# Referral Codes
## Overview
Nift Partner Referral Link, lets Nift partners provide $30 Nift Gift Cards to their customers by referring customers to the Nift service with partner specific code.

### Referral Link
To give a $30 Nift Gift Card to customers, the Nift partner should refer the customer to the dedicated link Nift provided to you.

The customer will be referred to a page that will ask the customers to enter their name, email and possibly zipcode.

The partner can include in the referral link the customer name and email. Nift will store that information and will pre-fill these fields in the landing page.

Here is how you can include the name and email in the referral link:
first_name={first_name}&last_name={last_name}&email={email}

You can include all or some of the fields.

You can also append customer id or other id by adding it as a parameter to the URL request. We support any parameter name you prefer to use. Just let us know in advance the parameter name. For example: client_id={client_id} 

## Braze Integration to Track Conversion

Partners can integrate their Braze account and Nift will push daily events to customers that selected. The partner will need to include in the referral URL the customer external ID or any other unique identifier in Braze that will let Nift be able to find the customer record and create the event.

Based on the given External ID, Nift will look in Braze for the customer record and create an event with: {name: “nift_processed”, time: , referral_code: }

Nift will also add “nift_processed”: true attribute to the customer record.

## Iterable Integration to Track Conversion

Partners can integrate their Iterable account and set which iterable list Nift should push the users info to.
Nift will push every morning the user info of the customers that came from the partner and selected a gift in the past 24 hours.
The user info will be: given customer_external_id, customer_selected time, referral_code
