# Cordova plugin for getting Advertising ID (IDFA or AAID)

## Index

<!-- MarkdownTOC levels="2" autolink="true" -->

- [Supported Platforms](#supported-platforms)
- [Installation](#installation)
- [API](#api)
- [Example](#example)

<!-- /MarkdownTOC -->

## Supported Platforms

- iOS

## Installation

    $ cordova plugin add cordova-plugin-idfaios

## API

The API is available on the `cordova.plugins.idfa` global object.

### getInfo()

Returns a `Promise<object>` with the following fields:

- `trackingLimited`: `boolean` - Whether usage of advertising id is allowed by user.
- `idfa`: `string` (_iOS only_) - Identifier for advertisers.
- `trackingPermission` (_iOS 14+ only_): [`number`](#tracking-permission-values)
   Tracking permission status, available on iOS 14+ devices.

### requestPermission()

_(iOS only)_ A one-time request to authorize or deny access to app-related data that can be used for
tracking the user or the device. See [Apple's API docs](https://developer.apple.com/documentation/apptrackingtransparency/attrackingmanager/3547037-requesttrackingauthorization)
for more info on the dialog presented to the user. Available only for iOS 14+ devices.

Returns a `Promise<`[`number`](#tracking-permission-values)`>`. On devices
with iOS < 14 the method will return a rejected promise.

**Note:** You should make sure to set the
[`NSUserTrackingUsageDescription`](https://developer.apple.com/documentation/bundleresources/information_property_list/nsusertrackingusagedescription) key in your app's
Information Property List file, otherwise your app will crash when you use this API.
You can do it with the following code in your Cordova project's `config.xml`:
```xml
<platform name="ios">
    <edit-config target="NSUserTrackingUsageDescription" file="*-Info.plist" mode="merge">
        <string>My tracking usage description</string>
    </edit-config>
</platform>
```

### Tracking Permission Values

The tracking permission values are `number`s returned by [`getInfo()`](#getinfo)
and [`requestPermission()`](#requestPermission). The possible values are stored in constants on the
plugin object. See the [example](#example) on how to use them.

For the meaning of the values see [the tracking transparency API docs](https://developer.apple.com/documentation/apptrackingtransparency/attrackingmanagerauthorizationstatus):

| Constant                           | Value | Description                                                                                               |
| :--------------------------------- | :---- | :-------------------------------------------------------------------------------------------------------- |
| TRACKING_PERMISSION_NOT_DETERMINED | 0     | User has not yet received an authorization request to authorize access to IDFA |
| TRACKING_PERMISSION_RESTRICTED     | 1     | User restricted the value returned if authorization to access IDFA |
| TRACKING_PERMISSION_DENIED         | 2     | The value returned if the user denies authorization to access IDFA |
| TRACKING_PERMISSION_AUTHORIZED     | 3     | The value returned if the user authorizes access to IDFA |

## Example

```js
const idfaPlugin = cordova.plugins.idfa;

idfaPlugin.getInfo()
    .then(info => {
        if (!info.trackingLimited) {
            return info.idfa || info.aaid;
        } else if (info.trackingPermission === idfaPlugin.TRACKING_PERMISSION_NOT_DETERMINED) {
            return idfaPlugin.requestPermission().then(result => {
                if (result === idfaPlugin.TRACKING_PERMISSION_AUTHORIZED) {
                    return idfaPlugin.getInfo().then(info => {
                        return info.idfa || info.aaid;
                    });
                }
            });
        }
    })
    .then(idfaOrAaid => {
        if (idfaOrAaid) {
            console.log(idfaOrAaid);
        }
    });
```

