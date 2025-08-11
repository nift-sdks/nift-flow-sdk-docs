# Web SDK

## Installation

### CDN (Recommended)
Include the SDK directly in your HTML:

```html
<script src="https://cdn.gonift.com/sdk/latest/nift-web-sdk.umd.min.js"></script>
```

## Quick Start

### 1. Initialize the SDK
Initialize with your Client ID provided by Nift:

```javascript
await NiftWebSDK.init({
  clientId: 'YOUR_CLIENT_ID',
  customer: {
    firstName: 'John', // optional
    lastName: 'Doe', // optional
    email: 'john.doe@example.com' // required
  },
  code: 'REFERRAL_CODE', // required: Your partner referral code (same for all customers)
});
```

### 2. Show Gift Modal
Once the SDK is initialized, you can show the gift activation modal when appropriate in your user flow:

```javascript
NiftWebSDK.showModal({
  onClaim: () => {}, // when the customer clicks the claim button
  onClose: () => {}, // when the customer closes the modal
  onShow: () => {}, // when the modal is shown
  onIneligible: () => {}, // when the customer is ineligible for the gift
});
```

The SDK will automatically construct the activation URL using the referral code pattern, including customer information as query parameters.

## Configuration

### Initialization Options

| Option | Type | Required | Description |
|--------|------|----------|-------------|
| `clientId` | string | Yes | Your unique Client ID |
| `customer` | object | Yes | Customer information |
| `customer.firstName` | string | No | Customer's first name |
| `customer.lastName` | string | No | Customer's last name |
| `customer.email` | string | Yes | Customer's email address |
| `code` | string | Yes | Your partner referral code (same for all customers) |

### Modal Options

| Option | Type | Required | Description |
|--------|------|----------|-------------|
| `onClaim` | function | No | Called when the customer clicks the claim button |
| `onClose` | function | No | Called when the customer closes the modal |
| `onShow` | function | No | Called when the modal is shown |
| `onIneligible` | function | No | Called when the customer is ineligible for the gift |

## Integration Approaches

### Using Referral Codes
1. Obtain a client ID and partner referral code from Nift
2. Use the same referral code for all customers
3. Pass the referral code to SDK when showing modal
4. Customer data is automatically included in the activation URL as query parameters



## Complete Example

```html
<!DOCTYPE html>
<html>
<head>
  <script src="https://cdn.gonift.com/sdk/latest/nift-web-sdk.umd.min.js"></script>
</head>
<body>
  <button id="claimGift">Claim Your Gift</button>

  <script>
    // Initialize SDK
    await NiftWebSDK.init({
      clientId: 'YOUR_CLIENT_ID',
      customer: {
        firstName: 'John',
        lastName: 'Doe',
        email: 'john@example.com'
      },
      code: 'REFERRAL_CODE_123', // Your partner referral code (same for all customers)
    });

    // Show modal on button click
    document.getElementById('claimGift').addEventListener('click', () => {
      NiftWebSDK.showModal({
        onClaim: () => {
          console.log('User clicked claim button');
        },
        onClose: () => {
          console.log('Modal closed');
        },
        onShow: () => {
          console.log('Modal shown');
        },
        onIneligible: () => {
          console.log('User is ineligible');
        }
      });
    });
  </script>
</body>
</html>
```

## Browser Support
- Chrome (latest)
- Firefox (latest)
- Safari (latest)
- Edge (latest)
- Mobile browsers

## Support
For integration support, contact your Nift account manager.