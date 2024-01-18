# React Native SDKs

## Installation

### 1. Install package
Add a `.npmrc` file and include
```.sh
@ogwee:registry=https://npm.pkg.github.com
//npm.pkg.github.com/:_authToken={GITHUB_TOKEN}
```
Then you can install the react native package:
```sh
npm install @ogwee/{package-name}
```
or
```sh
yarn add @ogwee/{package-name}
```
### 2. Peer Dependencies
Make sure all peer dependencies are installed.

```sh
npm install @react-native-clipboard/clipboard
npm install @react-navigation/native
npm install @react-navigation/native-stack
npm install react-native-safe-area-context
npm install react-native-screens
npm install react-native-svg
npm install react-native-asset
npm install react-native-device-info
```
or
```sh
yarn add @react-native-clipboard/clipboard
yarn add @react-navigation/native
yarn add @react-navigation/native-stack
yarn add react-native-safe-area-context
yarn add react-native-screens
yarn add react-native-svg
yarn add react-native-asset
yarn add react-native-device-info
```

### 4. Font Assets
To support font assets used, merge the following to your project's `react-native.config.js` file (create the file if it does not exist already).


```js
module.exports = {
    project: {
        ios:{},
        android:{}
    },
    assets:["node_modules/@ogwee/{package-name}/src/assets/"],
}
```

To sync fonts, call
```sh
yarn react-native-asset
```

### 5. Install pods
`cd ios` and `pod install`

## Usage
To initialize the service, call `NiftCardFlow.initialize` with your clientID (will be given to you), referral code, and customer info. All fields are required, and this can be called multiple times. Please initialize the service before displaying the view. The view takes as a prop an optional errorHandler.

```js
import { NiftCardFlowView, NiftCardFlow } from 'ogwee/{package-name}';
import type { NiftCardCustomer, NiftErrorCallback } from 'ogwee/{package-name}';

// ...

NiftCardFlow.initialize(customer: NiftCardCustomer, referralCode: string, clientId: string, passedMLDA?: boolean)
const errorhandler: NiftErrorCallback = (_error) => {
  Toast.show('error', { duration: Toast.durations.SHORT });
};
const linkHandler: LinkCallback = (link) => {
  Toast.show(link, { duration: Toast.durations.SHORT });
};

// ...

<NiftCardFlowView errorHandler={errorhandler} linkHandler={linkHandler} />
```

### NiftCardCustomer
| name        | type   | default      |
|:------------|:-------|:-------------|
| `firstName` | string | - (required) |
| `lastName`  | string | - (required) |
| `email`     | string | - (required) |
| `zipcode`   | string | - (optional) |
