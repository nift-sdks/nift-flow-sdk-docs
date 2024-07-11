# Partners Customer Status Server API

Our server-side API gives access to the customer status.
Using, oauth token authorization, the customer's email, and the referral code, the endpoint returns the customer status (eligibility to receive a Nift gift),
the last time they activated a nift card, and the last time they selected a gift.

## Token Authorisation
From a server, the access token must be received using the given cliend ID and secret.

`post` to https://www.gonift.com/oauth/token
Form Data Needed:
```
grant_type: client_credentials
client_id: [given by Nift]
client_secret: [given by Nift]
scope: read:customers
```

Posting to this endpoint returns json similar the following:
```json
{
    "access_token": "12345679hdvjkkg",
    "token_type": "Bearer",
    "expires_in": 31556952,
    "scope": "read:customers",
    "created_at": 1679447338
}
```

## Customer Status
Once you have a valid (unexpired) access token, the customer's status can be received.
`post` to https://www.gonift.com/api/v:2023-03/partners/customers/status
Form Data Needded:
```
email: "person@email.com"
code: [referral code]
```
Header Needed:
```
Authorization: Bearer [access token]
```

This will give you json similar to the following:
```json
{
    "status": "available",
    "last_activated_at": "2022-10-26T17:30:00.000-04:00",
    "last_selected_at": "2022-11-01T12:21:00.000-04:00"
}
```
