# Encrypted Email in the Nift SDK

## Overview

Partners can pass an encrypted email address when activating nift cards through the Nift SDKs. The encrypted value is passed in place of a plaintext email — no other changes to the integration are needed.

For how to encrypt the email value, see [Encrypting Values with AES-256-GCM](encrypting_values.md).

## Usage

When initializing a Nift SDK, pass the encrypted email as the `email` field in the customer object. The SDK handles decryption transparently.

> **Note:** Encryption must happen on your server, since the shared secret key must never be exposed in client-side code. Your backend should encrypt the email and pass the encrypted value to the client, which then provides it to the SDK.

### Web SDK

```javascript
await NiftWebSDK.init({
  clientId: 'YOUR_CLIENT_ID',
  customer: {
    firstName: 'Jane',
    lastName: 'Doe',
    email: 'eyJpdiI6Ii4uLiIsImNpcGhlcnRleHQiOiIuLi4iLCJ0YWciOiIuLi4ifQ' // encrypted email
  },
  code: 'REFERRAL_CODE',
});
```

### Android SDK

```kotlin
val customerInfo = CustomerInfo(
    customer = Customer(
        firstName = "Jane",
        lastName = "Doe",
        email = encryptedEmail // encrypted email from your backend
    )
)
val config = NiftCardFlowConfig(context, customerInfo, "REFERRAL_CODE", "YOUR_CLIENT_ID")
```

### iOS SDK

```swift
let customer = Customer(
    firstName: "Jane",
    lastName: "Doe",
    email: encryptedEmail // encrypted email from your backend
)
let customerInfo = CustomerInfo(customer: customer)
let config = NiftCardFlowConfig(customerInfo: customerInfo, code: "REFERRAL_CODE", clientId: "YOUR_CLIENT_ID")
```

### React Native SDK

```javascript
NiftCardFlow.initialize({
  customer: {
    firstName: 'Jane',
    lastName: 'Doe',
    email: encryptedEmail, // encrypted email from your backend
  },
  code: 'REFERRAL_CODE',
  clientId: 'YOUR_CLIENT_ID',
});
```

## How It Works

1. Your server encrypts the customer's email using the shared key
2. Your client passes the encrypted value to the SDK as the `email` field
3. The SDK sends the value to Nift, which detects whether it is plaintext (contains `@`) or encrypted (no `@`)
4. If encrypted, Nift decrypts it server-side using the shared key
5. The nift card is activated normally
