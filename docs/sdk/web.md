# Web SDK

## Installation

### CDN (Recommended)
Include the SDK directly in your HTML:

```html
<script src="https://cdn.nift.me/sdk/webembed/latest/nift-web-sdk.umd.min.js"></script>
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
    phoneNumber: '+16179876543', // optional
    email: 'john.doe@example.com' // required
  },
  code: 'REFERRAL_CODE', // required: Your partner referral code (same for all customers)
});
```

The SDK will automatically construct the activation URL using the referral code pattern, including customer information as query parameters.

### 2. Show Embedded Experience
For a seamless in-page experience, you can show the embedded modal which keeps users on your site:

```javascript
NiftWebSDK.showEmbeddedModal({
  onClose: () => {}, // when the customer closes the modal
  onShow: () => {}, // when the modal is shown
  onIneligible: () => {}, // when the customer is ineligible for the gift
});
```

The embedded modal provides a two-step flow:
1. **Offer Screen**: Displays the gift offer with a "Claim your gift" button
2. **SDK Flow**: After clicking claim, loads the full Nift redemption experience in an iframe

This approach keeps customers engaged within your application while still providing the complete Nift gift redemption experience.

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

#### showEmbeddedModal() Options

| Option | Type | Required | Description |
|--------|------|----------|-------------|
| `onClose` | function | No | Called when the customer closes the modal |
| `onShow` | function | No | Called when the modal is shown |
| `onIneligible` | function | No | Called when the customer is ineligible for the gift |

**Note:** The embedded modal does not have an `onClaim` callback since the claim action transitions to the SDK flow within the same modal.

## Integration Approaches

### Choosing Between Modal Types

**Embedded Modal (`showEmbeddedModal()`)**
- Keeps users on your site with an in-page iframe
- Best for: Seamless user experience without leaving your application
- User flow: View offer → Click claim → Iframe loads within modal
- Use case: Premium integrations requiring consistent branding and user retention

### Using Referral Codes
1. Obtain a client ID and partner referral code from Nift
2. Use the same referral code for all customers
3. Pass the referral code to SDK when showing modal
4. Customer data is automatically included in the activation URL as query parameters

## Complete Examples


### Example: Embedded Modal (In-Page Flow)

```html
<!DOCTYPE html>
<html>
<head>
  <script src="https://cdn.nift.me/sdk/webembed/latest/nift-web-sdk.umd.min.js"></script>
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

    // Show embedded modal on button click
    document.getElementById('claimGift').addEventListener('click', () => {
      NiftWebSDK.showEmbeddedModal({
        onShow: () => {
          console.log('Embedded modal shown');
        },
        onClose: () => {
          console.log('Embedded modal closed');
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

## Modal Behavior

### Embedded Modal
- **Offer Screen**: Shows gift offer with "Claim your gift" button
  - Responsive design adapts to screen size
  - Theme styling from backend (colors, fonts, button styles)
  - "Powered by Nift" footer with privacy policy link
  - Close button in top-right corner
- **SDK Flow Screen**: After clicking claim, transitions to iframe view
  - Desktop: 90% viewport height, max 900px
  - Mobile: Nearly full-screen with responsive padding
  - Complete Nift redemption experience embedded
  - Maintains same footer and close button

### Theme Customization
The modal supports theme customization:
- Primary colors
- Button styles and text
- Font family
- Modal dimensions
- Custom messaging (title, description, button text)

## Browser Support
- Chrome (latest)
- Firefox (latest)
- Safari (latest)
- Edge (latest)
- Mobile browsers

## Support
For integration support, contact your Nift account manager.
