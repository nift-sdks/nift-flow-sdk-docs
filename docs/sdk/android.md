# Android SDKs

## Using the SDK

### Main Use
Our Android SDKs gives access to a few classes to support instatiating and using the `NiftCardFlowFragment`(Android View) or `NiftCardFlowScreen` (Jetpack Compose).
```kotlin
val fragment = NiftCardFlowFragment(niftCardFlowConfig)
```
```kotlin
NiftCardFlowScreen(niftCardFlowConfig, modifier = Modifier)
```

To create the config, please provide context, a valid customer, Nift referral code, and the client ID (will be given to you).
There is an optional `passedMLDA` field for verifying the user is above or under the minimum drinking age. If this field is not set, the default Nift behaviour will be used.
```kotlin
NiftCardFlowConfig(context: Context, customer: Customer, referralCode: String, cliendId: String, passedMLDA: Boolean? = null)
```

And finally, `Customer` is a simple class that requires first name, last name, and email and has an optional zipcode field (for serving local gifts).
```kotlin
Customer(firstName: String, lastName: String, email: String, zipcode: String? = null)
```

All together, creating the fragment looks like the following:
```kotlin
val customer = Customer(firstName = "Person", lastName = "Surname", email = "person@email.com")
val context = requireContext()
val config = NiftCardFlowConfig(context, customer, "referralcode1234", "12345")
val fragment =  NiftCardFlowFragment(config)
```

And creating the screen looks like the following:
```kotlin
@Composable
fun ComposableView(...) {
val customer = Customer(firstName = "Person", lastName = "Surname", email = "person@email.com")
val context = LocalContext.current
val config = NiftCardFlowConfig(context, customer, "referralcode1234", "12345")
NiftCardFlowScreen(config)
```

Please keep in mind that referral code and client ID are subject to change and should not be hard coded.

### Additional Functions
Two types of preloading functions are provided in the config. One that uses `await()` and must be called from a suspend function, and another that completes asynchronously but doesn't require a suspend function

```kotlin
suspend fun preloadActivation(context: Context)
```
```kotlin
  fun preloadActivationAsync(context: Context)
```

There is also an `isLoaded` function
```kotlin
  fun isLoaded(): Boolean
```
And a function that returns a flow of caught errors. This is to support any custom error handling/visualization.
```
  fun getErrorUpdates(): Flow<Throwable>
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
         url = uri("https://maven.pkg.github.com/ogwee/[library-name]")
         credentials {
           username = System.getenv("NIFT_ANDROID_PACKAGE_USERNAME")
           password = System.getenv("NIFT_ANDROID_PACKAGE_TOKEN")
         }
       }
     }
   }
   ```
3. Enter the package name and version
   ```
   dependencies {
       implementation '{package-name}:1.0.1'
   }
   ```
4. Sync
