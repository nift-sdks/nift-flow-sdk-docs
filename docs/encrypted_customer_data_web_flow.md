# Encrypted Customer Data in the Nift Web Flow

## Overview

Partners can pass multiple customer fields in a single encrypted `customer_data` parameter in the Nift card referral link. This is an alternative to passing `email`, `first_name`, `last_name`, and `zipcode` as separate query parameters — instead, all fields are bundled into one encrypted blob.

For how to encrypt the value, see [Encrypting Values with AES-256-GCM](encrypting_values.md).

## URL Format

```
https://www.gonift.com/nift_cards/{referral_code}/start?customer_data={encrypted_blob}
```

The `customer_data` parameter replaces the individual query parameters (`email`, `first_name`, `last_name`, `zipcode`) with a single encrypted value.

## Supported Fields

The plaintext JSON inside the encrypted `customer_data` blob supports these fields:

| Field | Required | Description |
|-------|----------|-------------|
| `email` | No | Customer's email address |
| `first_name` | No | Customer's first name |
| `last_name` | No | Customer's last name |
| `postal_code` | No | Customer's postal/zip code |

All fields are optional — include whichever fields you have available. Unknown fields are ignored.

## How It Works

1. Build a JSON object containing the customer fields you want to send
2. Encrypt the JSON string using the process described in [Encrypting Values](encrypting_values.md)
3. Pass the encrypted value as the `customer_data` query parameter

### Step 1: Build the plaintext JSON

```json
{
  "email": "jane@example.com",
  "first_name": "Jane",
  "last_name": "Doe",
  "postal_code": "02116"
}
```

### Step 2: Encrypt the JSON string

Use the same AES-256-GCM encryption process described in [Encrypting Values](encrypting_values.md). The **plaintext** input is the JSON string from step 1 (e.g., `"{\"email\":\"jane@example.com\",\"first_name\":\"Jane\",\"last_name\":\"Doe\",\"postal_code\":\"02116\"}"`).

### Step 3: Pass the encrypted value in the URL

```
https://www.gonift.com/nift_cards/MYCODE123/start?customer_data=eyJpdiI6Ii4uLiIsImNpcGhlcnRleHQiOiIuLi4iLCJ0YWciOiIuLi4ifQ
```

Since the output is Base64url-encoded (using only `A-Z`, `a-z`, `0-9`, `-`, `_`), it is safe to include directly in a URL query string without additional percent-encoding.

## Parameter Precedence

Individual query parameters take precedence over fields in `customer_data`. If you pass both `customer_data` and a standalone parameter, the standalone parameter wins:

```
?customer_data={blob containing first_name: "Jane"}&first_name=Janet
```

In this example, `first_name` will be `"Janet"`, not `"Jane"`.

This allows you to encrypt most fields while still overriding specific values as needed.

## Examples

### All fields in customer_data

```
https://www.gonift.com/nift_cards/MYCODE123/start?customer_data=eyJpdiI6Ii4uLiIsImNpcGhlcnRleHQiOiIuLi4iLCJ0YWciOiIuLi4ifQ
```

### Partial fields (email only)

If you only have the customer's email, the plaintext JSON would be:

```json
{
  "email": "jane@example.com"
}
```

### Mixed: customer_data with individual overrides

```
https://www.gonift.com/nift_cards/MYCODE123/start?customer_data=eyJpdiI6Ii4uLiIsImNpcGhlcnRleHQiOiIuLi4iLCJ0YWciOiIuLi4ifQ&first_name=Janet
```

Here, `first_name` from the query string overrides whatever `first_name` was inside the encrypted blob.

## Behavior

The behavior when the customer lands on this URL depends on how the referral code is configured:

| Referral Code Type | What Happens |
|--------------------|-------------|
| **skip-details enabled** | The nift card is **automatically activated** using the provided parameters. The customer skips the details form entirely and goes straight to browsing gift categories. |
| **standard (skip-details disabled)** | The provided parameters **pre-fill the details form**. The customer still reviews and submits the form before the nift card is activated. |

## Comparison with Individual Encrypted Parameters

| | `customer_data` | Individual `email` param |
|--|-----------------|--------------------------|
| **What's encrypted** | JSON object with multiple fields | Single email value |
| **Plaintext detection** | Not supported — value is always treated as encrypted | If value contains `@`, treated as plaintext email |
| **Number of encrypted params** | One blob for all fields | One per encrypted field |
| **Supported fields** | `email`, `first_name`, `last_name`, `postal_code` | `email` only |

Both approaches can be used alongside each other. See [Encrypted Email in the Nift Web Flow](encrypted_email_web_flow.md) for the individual parameter approach.
