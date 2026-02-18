# Nift's Android SDK for the Nift Card Flow

## Using the SDK

### Main Use
Our Android SDKs gives access to a few classes to support instatiating and using the `NiftCardFlowFragment`(Android View) or `NiftCardFlowScreen` (Jetpack Compose).

```kotlin
val fragment = NiftCardFlowFragment(niftCardFlowConfig)
```
```kotlin
NiftCardFlowScreen(niftCardFlowConfig, modifier = Modifier)
```

To instatiate either view, a `NiftCardFlowConfig` is needed. As named, this class is what configures the nift card flow. 
```kotlin
NiftCardFlowConfig(
    context: Context,
    customerInfo: CustomerInfo,
    referralCode: String,
    clientId: String,
    isDarkTheme: Boolean = false,
    fontFamily: FontFamily? = null
)
```
To create the config, please provide context, customer info, Nift referral code, and the client ID (will be given to you). While Nift does not yet have an optional dark theme, a custom one can easily be configured by Nift and shown with the optional argument `isDarkTheme`. And because fonts in Android require a resource ID that can change any time, we provided an optional argument for configuring the font family (compose).

The `CustomerInfo` type contains the customer (first name, last name, email, and an optional zipcode) and optional fields country code, preferred language, whether the user has passed minimum drinking age or not, and a profile  for any extra info that will help select better gifts for the customer.

**Note:** Detailed type info is listed below.

All together, creating the fragment looks like the following:
```kotlin
val customerInfo = CustomerInfo(customer = Customer(firstName = "Person", lastName = "Surname", email = "person@email.com"))
val context = requireContext()
val config = NiftCardFlowConfig(context, customerInfo, "referralcode1234", "12345")
val fragment =  NiftCardFlowFragment(config)
```

And creating the screen looks like the following:
```kotlin
@Composable
fun ComposableView(...) {
  val customerInfo = CustomerInfo(customer = Customer(firstName = "Person", lastName = "Surname", email = "person@email.com"))
  val context = LocalContext.current
  val config = NiftCardFlowConfig(context, customerInfo, "referralcode1234", "12345")
  NiftCardFlowScreen(config)
}
```

**Note:** Please keep in mind that referral code and client ID are subject to change and should not be hard coded.

### Additional Functions
In the NiftCardFlowConfig, there are optional functions that can be called, many of which can be used for callbacks (with Kotlin Flow).

To receive errors (such as network errors), `getErrorUpdates` can be called.
```kotlin
fun getErrorUpdates(): Flow<Throwable> 
```

To handle the look and feel of toast messages, call `connectToastHandler`.
```kotlin
suspend fun connectToastHandler(): Flow<String>
```
**Note:** Calling this will disable default toast message handling. To enable the default toast message handling again, please call `disconnectToastHandler`.

To control how URLs are opened/displayed, please call `connectLinkHandler`.
```kotlin
suspend fun connectLinkHandler(): Flow<String>
```
**Note:** Similar to `connectToastHandler`, `connectLinkHandler` will stop the SDK from opening any URLs. Call disconnectLinkHandler to enable the default behaviour again.

To receive a callback when the user selects a gift, call `getSelectionUpdates`
```kotlin
fun getSelectionUpdates(): Flow<Unit>
```

And finally, while we do not at the moment have a default dark theme, the SDK does support having custom light and dark themes (configured by Nift). To switch for a view that's already shown, feel free to call setIsDarkTheme.
```kotlin
suspend fun setIsDarkTheme(value: Boolean)
```

## Requirements
Download the latest version of Android Studio and gradle. We currently support a minimum SDK of 26

## How to install
To install for Android development:

In Android Studio:
1. Go to build.gradle(app)
2. Under repositories, add the repo URl for your package. i.e.
   ```
   repositories {
       google()
       mavenCentral()
       maven {
         name = "GitHubPackages"
         url = uri("https://maven.pkg.github.com/nift-sdks/nift-card-flow-android-source")
         credentials {
           password = System.getenv("NIFT_ANDROID_PACKAGE_TOKEN")
           username = "oauth"
         }
       }
     }
   }
   ```
3. Enter the package name and version
   ```
   dependencies {
       implementation 'com.nift:nift-card-flow:2.1.2'
   }
   ```
4. Sync

## Types
### CustomerInfo
| name            | type         | default           |
|:----------------|:-------------|:------------------|
| `customer`      | Customer     | - (required)      |
| `countryCode`   | SupportedCountry  | `US`             |
| `language`      | SupportedLanguage | `en`             |
| `profile`       | `Map<String, Any>`       | - (optional)      |
| `passedMLDA`    | Boolean      | - (optional)      |

- `passedMLDA`: whether the user is of minimum legal drinking age or not. Leaving this null uses the default Nift behaviour
- `countryCode`: ISO 2 letter country code. Right now only the US, Canada, and the UK are supported. Defaults to the US.
- `language`: language code. Right now only english and french are supported. Defaults to english.**\***
- `profile`: any user info you want to attach to the customer in Nift to provide better gifts. If used, this must be a map with simple values.

**Note:** Even if a language is selected, if the gift or your location (based on client ID/referral code) doesn't support the lanugage, a language that is supported will be shown instead (most likely english). i.e. only Canadian locations support french at the moment.

### Customer
| name        | type   | default      |
|:------------|:-------|:-------------|
| `firstName` | string | - (required) |
| `lastName`  | string | - (required) |
| `email`     | string | - (required) |
| `zipcode`   | string | - (optional) |

### NiftCountry
`US | CA | GB` 

### NiftLanguage
`en | fr`
