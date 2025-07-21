# React Native SDKs

## Installation

### 1. Install package
Add a `.npmrc` file and include
```sh
@nift-sdks:registry=https://npm.pkg.github.com
//npm.pkg.github.com/:_authToken={GITHUB_TOKEN}
```
and/or a `.yarnrc.yml` file and include
```sh
npmScopes:
  nift-sdks:
    npmAlwaysAuth: true
    npmRegistryServer: "https://npm.pkg.github.com/nift-sdks/"
    npmAuthToken: "{GITHUB_TOKEN}"
```

Then you can then install version **3.4.3**:
```sh
npm install @nift-sdks/nift-card-flow-react-native
```
or
```sh
yarn add @nift-sdks/nift-card-flow-react-native
```
### 2. Peer Dependencies
Make sure all peer dependencies are installed.

```sh
npm install @react-native-clipboard/clipboard
npm install lottie-react-native
npm install react-native-device-info
npm install react-native-gesture-handler
npm install react-native-reanimated
npm install react-native-svg
```
or
```sh
yarn add @react-native-clipboard/clipboard
yarn add lottie-react-native
yarn add react-native-device-info
yarn add react-native-gesture-handler
yarn add react-native-reanimated
yarn add react-native-svg
```

### 4. Font Assets
To support font assets used, merge the following to your project's `react-native.config.js` file (create the file if it does not exist already).


```js
module.exports = {
    project: {
        ios:{},
        android:{}
    },
    assets:["node_modules/@nift-sdks/nift-card-flow-react-native/src/assets"],
}
```

To sync fonts, call
```sh
npx react-native-asset
```
If the `npx` command isn't available, install as a dev dependency instead and call `yarn react-native-asset`

### 5. Install pods
`cd ios` and `pod install`

### 6. Add GIF support
React Native has GIF animations off by default for Android. Because many gift images are actually GIFs, it's important to enable GIF support.
In the android, look for `build.gradle` file under app. Under dependencies, add the following:
```kotlin
implementation("com.facebook.fresco:animated-gif:3.4.0")
```
**Note**: This version might not be the latest version of the library. Check [here](https://github.com/facebook/react-native/blob/main/packages/react-native/gradle/libs.versions.toml) to see the latest version of fresco.

## Usage
To initialize the service, call `NiftCardFlow.initialize` with your clientID (will be given to you), referral code, and customer info (type shown below). `initialize` must be called before displaying the view, but can be called multiple times.

```js
import { NiftCardFlowView, NiftCardFlow } from 'nift-sdks/nift-card-flow-react-native';
import type { NiftCardCustomer, NiftErrorCallback } from 'nift-sdks/nift-card-flow-react-native';

// ...

NiftCardFlow.initialize(customerInfo: NiftCardCustomer, referralCode: string, clientId: string)
```

The view takes the following functions as optional props:
- `linkHandler`: used when URLs need to be opened in a special way
- `errorHandler`: used to know what errors may happen (i.e. network errors) especially so that they can be displayed somehow to the user
- `selectionHandler`: if set, this function is called when a user successfully selects a gift
- `toastHandler`: if set, this function is called with a message that is meant to be shown as a toast message. Allows anyone implementing this to customize how toasts are displayed
- `isDarkTheme`: Whether to show the dark or light theme. Defaults to false for light theme. This only works if a dark theme has been specified.
```
const errorhandler: NiftErrorCallback = (_error) => {
  Toast.show('error', { duration: Toast.durations.SHORT });
};
const linkHandler: NiftLinkCallback = (link) => {
  Toast.show(link, { duration: Toast.durations.SHORT });
};
const selectionHandler: () => {
  Toast.show('user selected a gift!');
};

// ...

<NiftCardFlowView errorHandler={errorhandler} linkHandler={linkHandler}
                  selectionHandler={selectionHandler}
                  isDarkTheme={false} />
```

**Error Handling**

The error handler in the react native SDK has the potential to throw 2 main errors:
 - `ApiResponseError`: can happen because of a network related issue.
 - `SecurityError`: happens when a user isn't allowed to access our servers. This is usually country related (i.e. trying to access from an unsupported country).
  

## Types
### NiftCardCustomer
| name            | type         | default           |
|:----------------|:-------------|:------------------|
| `customer`      | Customer     | - (required)      |
| `countryCode`   | NiftCountry  | `'US'`            |
| `language`      | NiftLanguage | `'en'` (required) |
| `userProfile`   | object       | - (optional)      |
| `passedMLDA`    | boolean      | - (optional)      |

- `passedMLDA`: whether the user is of minimum legal drinking age or not.
- `countryCode`: ISO 2 letter country code. Right now only the US, Canada, and the UK are supported. Defaults to the US.
- `language`: language code. Right now only english and french are supported. Defaults to english.**\***
- `userProfile`: any user info you want to attach to the customer in Nift to provide better gifts. If used, this must be a hash.

**Note:** Even if a language is selected, if the gift or your location (based on client ID/referral code) doesn't support the lanugage, a language that is supported will be shown instead (most likely english). i.e. only Canadian locations support french.

### Customer
| name        | type   | default      |
|:------------|:-------|:-------------|
| `firstName` | string | - (required) |
| `lastName`  | string | - (required) |
| `email`     | string | - (required) |
| `zipcode`   | string | - (optional) |

### NiftCountry
`'US' | 'CA' | 'GB'` 

### NiftLanguage
`'en' | 'fr'`
