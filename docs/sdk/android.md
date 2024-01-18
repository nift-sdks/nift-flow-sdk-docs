# Android SDKs

## Using the SDK

Our Android SDKs gives access to a few classes to support instatiating and using the `NiftCardFlowFragment`.
```kotlin
val fragment = NiftCardFlowFragment(niftCardFlowConfig)
```

To create the config, please provide a valid customer, Nift referral code, and the client ID (will be given to you along with a client secret).
There is an optional `passedMLDA` field for verifying the user as above or under the minimum drinking age. If this field is not set, the default Nift behaviour will be used.
```kotlin
NiftCardFlowConfig(customer: Customer, referralCode: String, clientId: String, passedMLDA: Boolean? = null)
```

And finally, `Customer` is a simple class that requires first name, last name, and email and has an optional zipcode field (for serving local gifts).
```kotlin
Customer(firstName: String, lastName: String, email: String, zipcode: String? = null)
```

All together, creating the fragment looks like the following:
```kotlin
val customer = Customer(firstName = "Person", lastName = "Surname", email = "person@email.com")
val config = NiftCardFlowConfig(customer = customer, referralCode = "code", clientId = "xxxxxxxxxx")
val fragment =  NiftCardFlowFragment(config)
```

Please keep in mind that referral code and client ID are subject to change and should not be hard coded.

## Requirements
Download the latest version of Xcode. Xcode 11 and above comes with a built-in Swift Package Manager.

For more information on Swift Package Manager, view the [official Swift documentation](https://www.swift.org/package-manager/).

## How to install
To install for iOS development:

On Xcode:
1. Go to File > Add Package Dependencies
2. Enter the package URL
3. Select your dependency rule (usually Up to Next Major Version)
