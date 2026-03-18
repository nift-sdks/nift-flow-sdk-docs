# Encrypted Email in the Nift Web Flow

## Overview

Partners can pass an encrypted email address (along with other customer details) as query parameters in the Nift card referral link. This is the browser-based flow where customers are redirected to the Nift website to claim their gift.

For how to encrypt the email value, see [Encrypting Values with AES-256-GCM](encrypting_values.md).

## URL Format

```
https://www.gonift.com/nift_cards/{referral_code}/start?email={encrypted_email}&first_name={name}&last_name={name}&zipcode={zip}
```

All customer fields are **top-level query parameters**.

## Parameters

| Parameter | Location | Required | Description |
|-----------|----------|----------|-------------|
| `referral_code` | URL path | Yes | The referral code provided by Nift |
| `email` | Query string | Yes | Encrypted email (see [Encrypting Values](encrypting_values.md)) or plaintext email |
| `first_name` | Query string | No | Customer's first name |
| `last_name` | Query string | No | Customer's last name |
| `zipcode` | Query string | No | Customer's zip code |

## Examples

### With plaintext email

```
https://www.gonift.com/nift_cards/MYCODE123/start?first_name=Jane&last_name=Doe&email=jane@example.com&zipcode=02116
```

### With encrypted email

```
https://www.gonift.com/nift_cards/MYCODE123/start?first_name=Jane&last_name=Doe&email=eyJpdiI6Ii4uLiIsImNpcGhlcnRleHQiOiIuLi4iLCJ0YWciOiIuLi4ifQ&zipcode=02116
```

The encrypted `email` value is produced using the process described in [Encrypting Values](encrypting_values.md). Since the output is Base64url-encoded (using only `A-Z`, `a-z`, `0-9`, `-`, `_`), it is safe to include directly in a URL query string without additional percent-encoding.

> **Note:** Because the encrypted value is passed via the `email` query parameter, the JSON payload inside can use either `"ciphertext"` or `"email"` as the key for the encrypted data. See [Encrypting Values — step 4](encrypting_values.md#how-to-encrypt) for details.

## Behavior

The behavior when the customer lands on this URL depends on how the referral code is configured:

| Referral Code Type | What Happens |
|--------------------|-------------|
| **skip-details enabled** | The nift card is **automatically activated** using the provided parameters. The customer skips the details form entirely and goes straight to browsing gift categories. |
| **standard (skip-details disabled)** | The provided parameters **pre-fill the details form**. The customer still reviews and submits the form before the nift card is activated. |
