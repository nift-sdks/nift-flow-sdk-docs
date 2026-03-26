# Encrypting Values with AES-256-GCM

## Overview

To protect sensitive customer data in transit, we support encrypting fields using **AES-256-GCM** symmetric encryption. You can encrypt:

- A **single value** (e.g., an email address) — passed as the `email` query parameter or SDK field
- **Multiple customer fields** bundled into a single JSON blob — passed as the `customer_data` query parameter

This encryption format is used across all Nift integration paths:

- [Nift Web Flow — Encrypted Email](encrypted_email_web_flow.md) — single encrypted email via referral links
- [Nift Web Flow — Encrypted Customer Data](encrypted_customer_data_web_flow.md) — multiple encrypted fields via referral links
- [Nift SDK](encrypted_email_sdk.md) — SDK initialization

## What We Provide

We will generate and securely share with you:

*   **Shared Secret Key**: A 32-byte (256-bit) key, provided as a Base64-encoded string
*   **Algorithm**: `AES-256-GCM`

Example key (for illustration only — your actual key will be different):

    k3LmR9xQ2vN8pY4wT6jB1aF5hG7dC0eI+sU3zX9oKmA=

**Important**: Store this key securely. It must never be exposed in client-side code, logs, or version control. Treat it with the same care as a private key or API secret.

## How to Encrypt

### Step-by-step process:

1.  **Generate a random 12-byte IV (nonce)** — this MUST be unique for every encryption. Never reuse an IV with the same key.

2.  **Encrypt the value** using AES-256-GCM with:
    *   Key: the shared secret (Base64-decode it first to get the raw 32 bytes)
    *   IV: the random 12 bytes from step 1
    *   Plaintext: the value to encrypt (e.g., `user@example.com`)
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

    The key for the encrypted data can be `"ciphertext"` **or** the name of the URL parameter carrying the value. For example, if the encrypted payload is sent via an `email` parameter, the following is also accepted:

    ```json
    {
      "iv": "<base64url-encoded 12-byte IV>",
      "email": "<base64url-encoded ciphertext>",
      "tag": "<base64url-encoded 16-byte tag>"
    }
    ```

    When both keys are present, `"ciphertext"` takes precedence.

5.  **Base64url-encode the entire JSON string** to produce the final payload.

6.  **Send this payload** as the parameter value in your chosen integration path.

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
async function encryptValue(value, base64Key) {
  const key = await crypto.subtle.importKey(
    'raw',
    Uint8Array.from(atob(base64Key), c => c.charCodeAt(0)),
    { name: 'AES-GCM' },
    false,
    ['encrypt']
  );

  const iv = crypto.getRandomValues(new Uint8Array(12));
  const encoded = new TextEncoder().encode(value);
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

def encrypt_value(value: str, base64_key: str) -> str:
    key = base64.b64decode(base64_key)
    iv = os.urandom(12)

    aesgcm = AESGCM(key)
    # encrypt returns ciphertext + tag concatenated
    ct_and_tag = aesgcm.encrypt(iv, value.encode(), None)
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

def encrypt_value(value, base64_key)
  key = Base64.decode64(base64_key)
  cipher = OpenSSL::Cipher.new('aes-256-gcm')
  cipher.encrypt
  cipher.key = key
  iv = cipher.random_iv
  cipher.auth_data = ""

  ciphertext = cipher.update(value) + cipher.final
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

public String encryptValue(String value, String base64Key) throws Exception {
    byte[] key = Base64.getDecoder().decode(base64Key);
    byte[] iv = new byte[12];
    new SecureRandom().nextBytes(iv);

    Cipher cipher = Cipher.getInstance("AES/GCM/NoPadding");
    cipher.init(Cipher.ENCRYPT_MODE,
        new SecretKeySpec(key, "AES"),
        new GCMParameterSpec(128, iv));

    byte[] encrypted = cipher.doFinal(value.getBytes("UTF-8"));
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

## Encrypting Multiple Fields (`customer_data`)

The `customer_data` parameter lets you encrypt multiple customer fields in a single blob. The process is the same as above, except the **plaintext** you encrypt is a JSON string instead of a raw value.

### Supported fields

| Field | Description |
|-------|-------------|
| `email` | Customer's email address |
| `first_name` | Customer's first name |
| `last_name` | Customer's last name |
| `postal_code` | Customer's postal/zip code |

### Steps

1. Build a JSON object with the customer fields you want to include (all fields are optional):

    ```json
    {
      "email": "user@example.com",
      "first_name": "Jane",
      "last_name": "Doe",
      "postal_code": "02116"
    }
    ```

2. Treat this JSON **string** as the plaintext and follow the same [encryption steps above](#how-to-encrypt).

3. Pass the result as the `customer_data` query parameter.

For example, using the Python function from the code examples:

```python
import json

customer_fields = json.dumps({
    "email": "user@example.com",
    "first_name": "Jane",
    "last_name": "Doe",
    "postal_code": "02116",
})

encrypted = encrypt_value(customer_fields, base64_key)
# Use as: ?customer_data={encrypted}
```

For full usage details, see [Encrypted Customer Data in the Nift Web Flow](encrypted_customer_data_web_flow.md).

## Important Notes

*   **Never reuse an IV** with the same key. Generate a fresh random 12-byte IV for every encryption.
*   **Plaintext fallback (email only)**: If you send an unencrypted email (containing `@`) via the `email` parameter, it will be accepted as-is. This allows gradual rollout. This does **not** apply to `customer_data`, which is always treated as encrypted.
*   **Plaintext detection**: The `@` character is not part of Base64 or Base64url, so the system reliably distinguishes plaintext email addresses from encrypted blobs by checking for the presence of `@`.
*   **Tag length**: Always use a 128-bit (16-byte) authentication tag. This is the GCM default and maximum.
*   **No padding characters**: Base64url encoding should NOT include `=` padding characters.
*   **URL safety**: The encrypted output uses only `A-Z`, `a-z`, `0-9`, `-`, and `_` — all URL-safe characters. No percent-encoding is needed when placing it in a query string.
