# Xendit Fingerprint SDK React Native

React Native SDK for device identification and fingerprinting with Xendit services.

Supports Android and iOS React Native mobile applications.

## Basic Usage

Initialize the SDK with your public API key on application start up and
perform a scan.

> The SDK must be initialized before use.

```js
// App.js
import React from 'react';
import { SafeAreaView } from 'react-native';
import FingerprintSdk from 'xendit-fingerprint-rn';

const App = () => {
    React.useEffect(() => {
        // Initialize the SDK on every application start up
        FingerprintSdk.init('MY_PUBLIC_API_KEY').then((sessionID) => {
            // Run a scan after completing initialization
            FingerprintSdk.scan();
        });
    });

    return <SafeAreaView>{appContents}</SafeAreaView>;
};

export default App;
```

Session ID is retrievable from either `FingerprintSdk.init()` on SDK
initialization or from the `FingerprintSdk.getSessionId()` convenience method.

Both of these functions returns a `Promise` that resolves to a Session ID of type `string`

```js
// On SDK init
const sessionID = await FingerprintSdk.init('MY_PUBLIC_API_KEY');

// After SDK init
const sessionID = await FingerprintSdk.getSessionId();
```

This Session ID can then be passed on to other Xendit APIs that support device
fingerprinting. Please refer to the respective API's documentation for further info.

## Requirements

-   React native (0.60 or newer recommended for autolinking)
-   Android 5.0 (API 21) or newer
-   iOS 11.0 or newer

## Installation

With Yarn:

```sh
$ yarn add xendit-fingerprint-sdk-rn
```

Install the package with NPM:

```sh
$ npm install --save xendit-fingerprint-sdk-rn
```

Linking the library:

> note for React Native 0.60 and above, [autolinking](https://reactnative.dev/blog/2019/07/03/version-60#native-modules-are-now-autolinked) is available and this step should be skipped.

```sh
$ npx react-native link xendit-fingerprint-sdk-rn
```

### iOS Setup

Frameworks must be enabled in CocoaPods by adding `use_frameworks!` into
your Podfile.

```ruby
target 'MyProject' do
  use_frameworks!
  # ...etc
end
```

Enable the `BUILD_LIBRARY_FOR_DISTRIBUTION` setting for every Cocoapod target using a `post_install` block
in your iOS `Podfile`. Either by including this snippet into an existing `post_install` block or
creating a new block at the bottom of your `Podfile`.

```ruby
post_install do |installer|
  installer.pods_project.targets.each do |target|
    target.build_configurations.each do |config|
      config.build_settings['BUILD_LIBRARY_FOR_DISTRIBUTION'] = 'YES'
    end
  end
end
```

Run `pod install` in your iOS directory containing the `Podfile`.

```bash
$ (cd ios; pod install)
```

### Android Setup

Ensure that mavenCentral is included as a repository in `build.gradle`.

For example:

```groovy
allprojects {
    repositories {
        mavenCentral()
        mavenLocal()
        maven {
            // All of React Native (JS, Obj-C sources, Android binaries) is installed from npm
            url("$rootDir/../node_modules/react-native/android")
        }
        maven {
            // Android JSC is installed from npm
            url("$rootDir/../node_modules/jsc-android/dist")
        }
        google()
        maven { url 'https://www.jitpack.io' }
    }
}
```

## Methods

All SDK methods are asynchronous, returning a `Promise` object. Refer to the
returns section of each method for the resolved type.

### `init()`

The SDK must be initialized before it can be used.

-   Initialization must be ran on every application start up.
-   The SDK will associate a session ID for the running app until it's killed.
-   A new session ID will be assigned on every application start up .

```javascript
const sessionId = await FingerprintSdk.init(publicKey);
```

#### **Parameters**

| Name      | Type   | Required | Description                                                                         |
| --------- | ------ | -------- | ----------------------------------------------------------------------------------- |
| publicKey | string | Yes      | A valid public API key provided by [Xendit Dashboard](https://dashboard.xendit.co/) |

> Do not use your private API key!

#### **Returns**

| Name      | Type   | Description          |
| --------- | ------ | -------------------- |
| sessionId | string | Generated session ID |

### `scan()`

Scans the device mobile device and sends the device fingerprint data to Xendit.

-   Scan should be called immediately after initializing the SDK.
-   Scan can be called multiple times within a session.
-   Do not `await` the scan, let it run in the background. This avoids blocking
    any foreground application code execution.

```javascript
FingerprintSdk.scan();
// Or
FingerprintSdk.scan(customerEventName, customerEventID);
```

#### **Parameters**

| Name              | Type   | Required | Description                                                                                                         |
| ----------------- | ------ | -------- | ------------------------------------------------------------------------------------------------------------------- |
| customerEventName | string | No       | Optional event name to associate with this scan. Recommended to use snake case formatting. e.g. `'some_event_name'` |
| customerEventID   | string | No       | Optional identifier associated with the event. e.g. user account ID                                                 |

### `getSessionId()`

Convenience method to retrieve Session ID after SDK initialization.

```javascript
const sessionId = await FingerprintSdk.getSessionId();
```

#### **Returns**

| Name      | Type   | Description          |
| --------- | ------ | -------------------- |
| sessionId | string | Generated session ID |

### `setEnabled()`

Enables or disables the SDK.

-   When disabled, the scan method will do nothing. No data will be sent to Xendit.
-   When enabled, the SDK will resume operating as normal.
-   `await` should be used to ensure the SDK is completely enabled/disabled.
-   The SDK can be disabled before initializing.

```javascript
await FingerprintSdk.setEnabled(enable);
```

#### **Parameters**

| Name   | Type    | Required | Description                                          |
| ------ | ------- | -------- | ---------------------------------------------------- |
| enable | boolean | Yes      | `true` enables the SDK.<br>`false` disables the SDK. |

#### **Returns**

| Name | Type | Description                                  |
| ---- | ---- | -------------------------------------------- |
|      | void | Resolves once SDK is enabled/disabled fully. |
