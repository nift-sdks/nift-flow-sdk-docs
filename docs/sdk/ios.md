# iOS SDKs
## Using the SDK
### Main Use

Our iOS SDKs gives access to a few classes to support instatiating and using `NiftCardFlowView` (Swift UI)
```swift
NiftCardFlowView(config: niftCardFlowConfig)
```

To create the config, please provide a valid customer, Nift referral code, and the client ID (will be given to you along with a client secret).
There is an optional `passedMLDA` field for verifying the user as above or under the minimum drinking age. If this field is not set, the default Nift behaviour will be used.
```swift
NiftCardFlowConfig(customer: Customer, referralCode: String, clientId: String, passedMLDA: Bool? = nil)
```

And finally, `Customer` is a simple class that requires first name, last name, and email and has an optional zipcode field (for serving local gifts).
```swift
Customer(firstName: String, lastName: String, email: String, zipcode: String? = nil)
```

All together, creating the view looks like the following:
```swift
import [LibraryName]IosSdk
import NiftIosCommonBase # for Customer definition

struct ContentView: View {
    var config = CscNiftCardFlowConfig(
        customer: Customer(
            email: "person@email.com",
            firstName: "Person",
            lastName: "Surname"
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
In `NiftCardFlowConfig`, there is a preload function that uses async/await
```swift
public func preloadActivation() async -> Bool
```

There is also and `isLoaded` function
```swift
public func isLoaded() -> Bool
```

And a function that returns a publisher (for use with Combine) of errors. This is to support any custom error handling/visualization.
```swift
public func getErrorUpdates() -> AnyPublisher<Error, Never>
```

## Requirements
Download the latest version of Xcode. Xcode 11 and above comes with a built-in Swift Package Manager.

For more information on Swift Package Manager, view the [official Swift documentation](https://www.swift.org/package-manager/).

## How to install
To install for iOS development:

On Xcode:
1. Go to File > Add Package Dependencies
2. Enter the package URL (this will be given to you)
3. Select your dependency rule (usually Up to Next Major Version)
