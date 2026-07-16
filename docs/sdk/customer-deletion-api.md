# Partners Customer Deletion API

Our server-side API lets partners submit customer deletion requests on behalf of their customers.
Requests are fulfilled by **anonymization**: Nift scrubs and replaces the customer's personal data
rather than deleting records outright. You can submit one or many customer email addresses per
request; each address is processed individually in the background.

## Token Authorization
From a server, the access token must be received using the given client ID and secret.

`post` to https://www.gonift.com/oauth/token
Form Data Needed:
```
grant_type: client_credentials
client_id: [given by Nift]
client_secret: [given by Nift]
scope: write:customers
```

Posting to this endpoint returns json similar to the following:
```json
{
    "access_token": "12345679hdvjkkg",
    "token_type": "Bearer",
    "expires_in": 31556952,
    "scope": "write:customers",
    "created_at": 1679447338
}
```

> **Note:** Tokens are long-lived (about one year). Store the token server-side and request a new
> one when it expires. Never expose your client secret or access tokens in browsers or mobile apps.

## Submitting Deletion Requests
Once you have a valid (unexpired) access token, submit one or many customer email addresses.

`post` to https://www.gonift.com/api/v2026-07/partners/customers/deletions

Header Needed:
```
Authorization: Bearer [access token]
Content-Type: application/json
```

JSON Body:
```json
{
    "emails": ["jane@example.com", "john@example.com"]
}
```

| Parameter | Location | Required | Description |
|-----------|----------|----------|-------------|
| `emails` | JSON body | Yes | Array of 1–500 customer email addresses (strings). |

## Response
A successful request returns `202 Accepted` acknowledging receipt:

```json
{
    "received_count": 2,
    "queued_count": 2
}
```

| Field | Description |
|-------|-------------|
| `received_count` | Number of email addresses received in this request. |
| `queued_count` | Number of addresses newly queued for deletion. This is `received_count` minus any addresses that were invalid, duplicated within the same request, or already queued by a previous request that has not finished processing yet. |

> **Note:** The response is a receipt only. Deletion happens asynchronously after this
> response. `queued_count` reflects only the de-duplication and validation of your own submission
> — it never discloses whether an email address belongs to a Nift customer, and there is no
> per-address result.

## Errors

| Status | Error | When |
|--------|-------|------|
| `400` | `invalid_request` | `emails` is missing, not an array, empty, longer than 500 entries, or contains non-string values. |
| `401` | `invalid_token` | The access token is missing, invalid, or expired. |
| `403` | `insufficient_scope` | The token does not have the `write:customers` scope. |
| `403` | `integration_not_configured` | Your application is not set up for deletion requests — contact Nift. |

Authentication failures (`invalid_token`, `insufficient_scope`) return an **empty body**; the
error code and description are carried in the `WWW-Authenticate` response header, per the OAuth2
Bearer Token spec:

```
WWW-Authenticate: Bearer realm="Doorkeeper", error="invalid_token", error_description="The access token expired"
```

The other errors return a json body with an `error` code and a human-readable
`error_description`:

```json
{
    "error": "invalid_request",
    "error_description": "emails must be a JSON array of 1..500 email strings"
}
```

## Behavior
- Deletion runs asynchronously in the background; the `202` response does not wait for
  processing to finish.
- One malformed address never rejects the batch — it is ignored, and the valid addresses in the
  same request are still queued.
- Email addresses are normalized before matching: surrounding whitespace is removed and the
  address is lowercased.
- The endpoint is **idempotent**: re-submitting an address that is already queued does not
  create duplicate work — it is skipped and not counted in `queued_count`. Retrying a request
  after a network failure is safe.
- If a queued address does not match a Nift customer, the request is resolved internally;
  there is nothing further for you to do.

## Important Notes
- Maximum 500 email addresses per request. Split larger lists across multiple requests.
- Call this API from your server only. Never expose your client secret or access tokens in
  client-side code.
