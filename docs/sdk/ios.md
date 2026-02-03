Nift's iOS SDK for the Nift Card Flow

## Using the SDK
### Main Use

Our iOS SDKs gives access to a few classes and structs to support instantiating and using the nift card flow (Swift UI).

To show the view, call `NiftCardFlowView`. 
```swift
NiftCardFlowView(config: niftCardFlowConfig)
```

To use with UIKit, pass the SwiftUI view to a `UIHostingController`.
```swift
let view = NiftCardFlowView(config: niftCardFlowConfig)
UIHostingController(rootView: view)
```

The `NiftCardFlowConfig` uses a few things for initialization: a `CustomerInfo` instance, `referralCode` (String), `clientID` (String),  and `isDarkTheme` (optional Bool).

The `CustomerInfo` type contains the customer (first name, last name, email, and an optional zipcode) and optional fields country code, preferred language, whether the user has passed minimum drinking age or not, and a profile has for any extra info that will help select better gifts for the customer.

```swift
NiftCardFlowConfig(customerInfo: CustomerInfo, referralCode: String, clientId: String, isDarkTheme: Bool = false)
```

* Detailed type info is listed below.

All together, creating the view looks like the following:
```swift
import NiftCardFlowSource

struct ContentView: View {
    var config = NiftCardFlowConfig(
        customerInfo: CustomerInfo(
            customer: Customer(
                email: "person@email.com",
                firstName: "Person",
                lastName: "Surname"
            )
        ),
        referralCode: "code",
        clientId: "xxxxxxxxxx")

    var body: some View {
        VStack {
            NiftCardFlowView(config: config)
                    .frame(maxWidth: .infinity, maxHeight: .infinity)
        }
    }
}
```

Please keep in mind that referral code and client ID are subject to change and should not be hard coded.

### Additional Functions
In the `NiftCardFlowConfig`, there are optional functions that can be called, many of which can be used for callbacks (with Combine).

To receive errors (such as network errors), `getErrorUpdates` can be called.
```swift
public func getErrorUpdates() -> AnyPublisher<Error, Never>
```

To handle the look and feel of toast messages, call `connectToastHandler()`.
```swift
public func connectToastHandler() -> AnyPublisher<String, Never>
```
**Note**: Calling this will disable default toast message handling. To enable the default toast message handling again, please call `disconnectToastHandler`.

To control how URLs are opened/displayed, please call `connectLinkHandler`.
```swift
public func connectLinkHandler() -> AnyPublisher<String, Never>
```

**Note**: Similar to `connectToastHandler`, `connectLinkHandler` will stop the SDK from opening any URLs. Call `disconnectLinkHandler` to enabled the default behaviour again.

To receive a callback when the user selects a gift, call `getSelectionUpdates`
```swift
public func getSelectionUpdates() -> AnyPublisher<Void, Never>
```

And finally, while we do not at the moment have a default dark theme, the SDK does support having custom light and dark themes. To switch for a view that's already shown, feel free to call `setIsDarkTheme`.
```swift
public func setIsDarkTheme(darkTheme: Bool) 
```

## Requirements
Download the latest version of Xcode. Xcode 11 and above comes with a built-in Swift Package Manager.

For more information on Swift Package Manager, view the [official Swift documentation](https://www.swift.org/package-manager/).

## How to install
To install for iOS development:

via Xcode:
1. Go to File > Add Package Dependencies
2. Enter the package URL with token (https://oauth2:{GITHUB_TOKEN}@github.com/nift-sdks/nift-card-flow-ios.git). This should not require signing in separately with a user.
3. Select your dependency rule (usually Up to Next Major Version)

or add the following to your Package.swift
```
.package(url: "https://oauth2:{GITHUB_TOKEN}@github.com/nift-sdks/nift-card-flow-ios.git", from: "2.9.1")
```

## Types
### CustomerInfo
| name            | type         | default           |
|:----------------|:-------------|:------------------|
| `customer`      | Customer     | - (required)      |
| `countryCode`   | SupportedCountry  | `.US`             |
| `language`      | SupportedLanguage | `.en`             |
| `profile`       | `[String: Any]`       | - (optional)      |
| `passedMLDA`    | Bool      | - (optional)      |

- `passedMLDA`: whether the user is of minimum legal drinking age or not.
- `countryCode`: ISO 2 letter country code. Right now only the US, Canada, and the UK are supported. Defaults to the US.
- `language`: language code. Right now only english and french are supported. Defaults to english.**\***
- `profile`: any user info you want to attach to the customer in Nift to provide better gifts. If used, this must be a hash.

**Note:** Even if a language is selected, if the gift or your location (based on client ID/referral code) doesn't support the lanugage, a language that is supported will be shown instead (most likely english). i.e. only Canadian locations support french at the moment.

### Customer
| name        | type   | default      |
|:------------|:-------|:-------------|
| `firstName` | string | - (required) |
| `lastName`  | string | - (required) |
| `email`     | string | - (required) |
| `zipcode`   | string | - (optional) |

### NiftCountry
`.US | .CA | .GB` 

### NiftLanguage
`.en | .fr`
