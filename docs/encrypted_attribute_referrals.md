# Partner Integration Guide: AES-256-GCM Email Encryption

## Overview

To protect customer email addresses in transit, we support encrypting the `email` field using **AES-256-GCM** symmetric encryption. Encrypted email can be passed through two integration paths:

- **Gift Card Web Flow** — the `GET /nift_cards/<referral_code>/start` URL with query parameters
- **Partner API** — the `POST /api/v{version}/partners/nift_cards/:code/activations` endpoint

Both paths use the same encryption format and shared key.

## What We Provide

We will generate and securely share with you:

*   **Shared Secret Key**: A 32-byte (256-bit) key, provided as a Base64-encoded string
*   **Algorithm**: `AES-256-GCM`

Example key (for illustration only — your actual key will be different):

    k3LmR9xQ2vN8pY4wT6jB1aF5hG7dC0eI+sU3zX9oKmA=

**Important**: Store this key securely. It must never be exposed in client-side code, logs, or version control. Treat it with the same care as a private key or API secret.

## How to Encrypt the Email

### Step-by-step process:

1.  **Generate a random 12-byte IV (nonce)** — this MUST be unique for every encryption. Never reuse an IV with the same key.

2.  **Encrypt the email** using AES-256-GCM with:
    *   Key: the shared secret (Base64-decode it first to get the raw 32 bytes)
    *   IV: the random 12 bytes from step 1
    *   Plaintext: the email address (e.g., `user@example.com`)
    *   No additional authenticated data (AAD)

3.  **Collect the outputs**:
    *   `ciphertext`: the encrypted data
    *   `tag`: the 16-byte GCM authentication tag

4.  **Build a JSON object** with each value Base64url-encoded (RFC 4648 §5 — uses `-` and `_` instead of `+` and `/`, no padding `=`):

    ```json
    {
      "iv": "<base64url-encoded 12-byte IV>",
      "ciphertext": "<base64url-encoded ciphertext>",
      "tag": "<base64url-encoded 16-byte tag>"
    }
    ```

5.  **Base64url-encode the entire JSON string** to produce the final payload.

6.  **Send this payload** as the `email` parameter value in either integration path.

### Example

Given:

*   Email: `user@example.com`
*   Key (base64): `k3LmR9xQ2vN8pY4wT6jB1aF5hG7dC0eI+sU3zX9oKmA=`

After encryption, the JSON (before final encoding) might look like:

```json
{
  "iv": "dGhpcyBpcyBhIG5v",
  "ciphertext": "c29tZSBlbmNyeXB0ZWQgZGF0YQ",
  "tag": "YXV0aGVudGljYXRpb250YWc"
}
```

The final value sent in the `email` field would be:

    eyJpdiI6ImRHaHBjeUJwY3lCaElHNXYiLCJjaXBoZXJ0ZXh0IjoiYzI5dFpTQmxibU55ZVhCMFpXUWdaR0YwWVEiLCJ0YWciOiJZWFYwYUdWdWRHbGpZWFJwYjI1MFlXYyJ9

_(These are illustrative values — actual encrypted output will differ every time due to the random IV.)_

## Code Examples

<details>
<summary>JavaScript (Web Crypto API)</summary>

```javascript
async function encryptEmail(email, base64Key) {
  const key = await crypto.subtle.importKey(
    'raw',
    Uint8Array.from(atob(base64Key), c => c.charCodeAt(0)),
    { name: 'AES-GCM' },
    false,
    ['encrypt']
  );

  const iv = crypto.getRandomValues(new Uint8Array(12));
  const encoded = new TextEncoder().encode(email);
  const encrypted = await crypto.subtle.encrypt(
    { name: 'AES-GCM', iv, tagLength: 128 },
    key,
    encoded
  );

  // Web Crypto appends the tag to the ciphertext
  const encryptedBytes = new Uint8Array(encrypted);
  const ciphertext = encryptedBytes.slice(0, -16);
  const tag = encryptedBytes.slice(-16);

  const payload = JSON.stringify({
    iv: base64urlEncode(iv),
    ciphertext: base64urlEncode(ciphertext),
    tag: base64urlEncode(tag)
  });

  return base64urlEncode(new TextEncoder().encode(payload));
}

function base64urlEncode(bytes) {
  const base64 = btoa(String.fromCharCode(...bytes));
  return base64.replace(/\+/g, '-').replace(/\//g, '_').replace(/\=+$/, '');
}
```

</details>

<details>
<summary>Python</summary>

```python
import json, base64, os
from cryptography.hazmat.primitives.ciphers.aead import AESGCM

def encrypt_email(email: str, base64_key: str) -> str:
    key = base64.b64decode(base64_key)
    iv = os.urandom(12)

    aesgcm = AESGCM(key)
    # encrypt returns ciphertext + tag concatenated
    ct_and_tag = aesgcm.encrypt(iv, email.encode(), None)
    ciphertext = ct_and_tag[:-16]
    tag = ct_and_tag[-16:]

    payload = json.dumps({
        "iv": base64url_encode(iv),
        "ciphertext": base64url_encode(ciphertext),
        "tag": base64url_encode(tag),
    })

    return base64url_encode(payload.encode())

def base64url_encode(data: bytes) -> str:
    return base64.urlsafe_b64encode(data).rstrip(b"=").decode()
```

</details>

<details>
<summary>Ruby</summary>

```ruby
require 'openssl'
require 'json'
require 'base64'

def encrypt_email(email, base64_key)
  key = Base64.decode64(base64_key)
  cipher = OpenSSL::Cipher.new('aes-256-gcm')
  cipher.encrypt
  cipher.key = key
  iv = cipher.random_iv
  cipher.auth_data = ""

  ciphertext = cipher.update(email) + cipher.final
  tag = cipher.auth_tag

  payload = {
    iv: Base64.urlsafe_encode64(iv, padding: false),
    ciphertext: Base64.urlsafe_encode64(ciphertext, padding: false),
    tag: Base64.urlsafe_encode64(tag, padding: false)
  }.to_json

  Base64.urlsafe_encode64(payload, padding: false)
end
```

</details>

<details>
<summary>Java</summary>

```java
import javax.crypto.Cipher;
import javax.crypto.spec.GCMParameterSpec;
import javax.crypto.spec.SecretKeySpec;
import java.security.SecureRandom;
import java.util.Base64;

public String encryptEmail(String email, String base64Key) throws Exception {
    byte[] key = Base64.getDecoder().decode(base64Key);
    byte[] iv = new byte[12];
    new SecureRandom().nextBytes(iv);

    Cipher cipher = Cipher.getInstance("AES/GCM/NoPadding");
    cipher.init(Cipher.ENCRYPT_MODE,
        new SecretKeySpec(key, "AES"),
        new GCMParameterSpec(128, iv));

    byte[] encrypted = cipher.doFinal(email.getBytes("UTF-8"));
    // Java appends tag to ciphertext
    byte[] ciphertext = Arrays.copyOfRange(encrypted, 0, encrypted.length - 16);
    byte[] tag = Arrays.copyOfRange(encrypted, encrypted.length - 16, encrypted.length);

    Base64.Encoder encoder = Base64.getUrlEncoder().withoutPadding();
    String payload = String.format(
        "{\"iv\":\"%s\",\"ciphertext\":\"%s\",\"tag\":\"%s\"}",
        encoder.encodeToString(iv),
        encoder.encodeToString(ciphertext),
        encoder.encodeToString(tag));

    return encoder.encodeToString(payload.getBytes("UTF-8"));
}
```

</details>

## Integration Paths

Nift supports two ways to pass an encrypted email address. Both use the same encryption format described above.

### Gift Card Web Flow

Use this path when you want to link customers directly to the gift card experience in a browser. This is commonly used with **skip-details referral codes** that auto-activate the nift card without requiring the customer to fill out a form.

<details>
<summary>more information</summary>

#### URL Format

```
GET /nift_cards/{referral_code}/start?email={encrypted_email}&first_name={name}&last_name={name}&zipcode={zip}
```

All customer fields are **top-level query parameters** (not nested under `loyal_customer` like the API path).

#### Parameters

| Parameter | Location | Required | Description |
|-----------|----------|----------|-------------|
| `referral_code` | URL path | Yes | The referral code |
| `email` | Query string | Yes | Encrypted email (same format as API) or plaintext |
| `first_name` | Query string | No | Customer's first name |
| `last_name` | Query string | No | Customer's last name |
| `zipcode` | Query string | No | Customer's zip code |

#### Example with plaintext email

```
/nift_cards/MYCODE123/start?first_name=Jane&last_name=Doe&email=jane@example.com&zipcode=02116
```

#### Example with encrypted email

```
/nift_cards/MYCODE123/start?first_name=Jane&last_name=Doe&email=eyJpdiI6Ii4uLiIsImNpcGhlcnRleHQiOiIuLi4iLCJ0YWciOiIuLi4ifQ&zipcode=02116
```

The encrypted `email` value is produced using the exact same encryption process described above. Since the output is Base64url-encoded (using only `A-Z`, `a-z`, `0-9`, `-`, `_`), it is safe to include directly in a URL query string without additional percent-encoding.

#### Behavior: Skip-Details vs. Standard Referral Codes

The behavior when the customer lands on this URL depends on how the referral code is configured:

| Referral Code Type | What Happens |
|--------------------|-------------|
| **skip-details enabled** | The nift card is **automatically activated** using the provided parameters. The customer skips the details form entirely and goes straight to browsing gift categories. |
| **standard (skip-details disabled)** | The provided parameters **pre-fill the details form**. The customer still reviews and submits the form before the nift card is activated. |

</details>

---

### Partner API

Use this path when your backend activates nift cards programmatically on behalf of customers.

<details>
<summary>more information</summary>

#### Endpoint

```
POST /api/v{version}/partners/nift_cards/{code}/activations
Content-Type: application/json
Authorization: Bearer <oauth_token>
```

#### Request Body

```json
{
  "loyal_customer": {
    "email": "<encrypted_email_or_plaintext>",
    "first_name": "Jane",
    "last_name": "Doe",
    "zipcode": "02116"
  }
}
```

| Field | Required | Description |
|-------|----------|-------------|
| `loyal_customer.email` | Yes | Encrypted email (output of the encryption process above) or plaintext email |
| `loyal_customer.first_name` | Yes | Customer's first name |
| `loyal_customer.last_name` | Yes | Customer's last name |
| `loyal_customer.zipcode` | Yes | Customer's zip code |

#### Example with encrypted email

```json
{
  "loyal_customer": {
    "email": "eyJpdiI6Ii4uLiIsImNpcGhlcnRleHQiOiIuLi4iLCJ0YWciOiIuLi4ifQ",
    "first_name": "Jane",
    "last_name": "Doe",
    "zipcode": "02116"
  }
}
```

#### How It Works

1. Your server encrypts the customer's email using the shared key
2. Your server sends a `POST` request with the encrypted email in the `loyal_customer` object
3. Nift detects whether the email is plaintext (contains `@`) or encrypted (no `@`)
4. If encrypted, Nift decrypts it server-side using the shared key
5. The nift card is activated and a response is returned with the card details

</details>

## Important Notes

*   **Never reuse an IV** with the same key. Generate a fresh random 12-byte IV for every encryption.
*   **Plaintext fallback**: If you send an unencrypted email (containing `@`), it will be accepted as-is. This allows gradual rollout.
*   **Plaintext detection**: The `@` character is not part of Base64 or Base64url, so the system reliably distinguishes plaintext email addresses from encrypted blobs by checking for the presence of `@`.
*   **Tag length**: Always use a 128-bit (16-byte) authentication tag. This is the GCM default and maximum.
*   **No padding characters**: Base64url encoding should NOT include `=` padding characters.
*   **URL safety**: The encrypted output uses only `A-Z`, `a-z`, `0-9`, `-`, and `_` — all URL-safe characters. No percent-encoding is needed when placing it in a query string.
