# Bluetooth Serial Plugin for Cordova

This plugin enables serial communication over Bluetooth. It was
originally written for communicating between Android or iOS and an
Arduino.

It can be used to communicate with Bluetooth barcode scanners in SPP
mode.

Android uses Classic Bluetooth.  iOS support has been disabled at this
point.

Under Android 12 (API 31), Android support targets the lowest permission
profiles, so only the BLUETOOTH_CONNECT permission is used, meaning that
this plugin no longer manages discoverability.

## Supported Platforms

* Android
* Browser (Testing only. See [comments](https://github.com/don/BluetoothSerial/blob/master/src/browser/bluetoothSerial.js).)

[Supporting other Bluetooth Low Energy hardware](#supporting-other-ble-hardware)

## Limitations

 * The phone must initiate the Bluetooth connection
 * Will *not* connect Android to Android[*](https://github.com/don/BluetoothSerial/issues/50#issuecomment-66405396)

# Installing

Install with Cordova cli

    $ cordova plugin add cordova-plugin-bluetooth-serial

Note that this plugin's id changed from `com.megster.cordova.bluetoothserial` to `cordova-plugin-bluetooth-serial` as part of the migration from the [Cordova plugin repo](http://plugins.cordova.io/) to [npm](https://www.npmjs.com/).

# Examples

There are some [sample projects](https://github.com/don/BluetoothSerial/tree/master/examples) included with the plugin.

# API

## Methods

- [bluetoothSerial.connect](#connect)
- [bluetoothSerial.connectInsecure](#connectInsecure)
- [bluetoothSerial.disconnect](#disconnect)
- [bluetoothSerial.write](#write)
- [bluetoothSerial.subscribeRawData](#subscriberawdata)
- [bluetoothSerial.unsubscribeRawData](#unsubscriberawdata)
- [bluetoothSerial.list](#list)
- [bluetoothSerial.isEnabled](#isenabled)
- [bluetoothSerial.isConnected](#isconnected)
- [bluetoothSerial.showBluetoothSettings](#showbluetoothsettings)
- [bluetoothSerial.enable](#enable)

## connect

Connect to a Bluetooth device.

    bluetoothSerial.connect(macAddress_or_uuid, connectSuccess, connectFailure);

### Description

Function `connect` connects to a Bluetooth device.  The callback is long running.  Success will be called when the connection is successful.  Failure is called if the connection fails, or later if the connection disconnects. An error message is passed to the failure callback.

#### Android
For Android, `connect` takes a MAC address of the remote device.

### Parameters

- __macAddress_or_uuid__: Identifier of the remote device.
- __connectSuccess__: Success callback function that is invoked when the connection is successful.
- __connectFailure__: Error callback function, invoked when error occurs or the connection disconnects.

## connectInsecure

Connect insecurely to a Bluetooth device.

    bluetoothSerial.connectInsecure(macAddress, connectSuccess, connectFailure);

### Description

Function `connectInsecure` works like [connect](#connect), but creates an insecure connection to a Bluetooth device.  See the [Android docs](http://goo.gl/1mFjZY) for more information.

#### Android
For Android, `connectInsecure` takes a macAddress of the remote device.

### Parameters

- __macAddress__: Identifier of the remote device.
- __connectSuccess__: Success callback function that is invoked when the connection is successful.
- __connectFailure__: Error callback function, invoked when error occurs or the connection disconnects.


## disconnect

Disconnect.

    bluetoothSerial.disconnect([success], [failure]);

### Description

Function `disconnect` disconnects the current connection.

### Parameters

- __success__: Success callback function that is invoked after the connection is disconnected. [optional]
- __failure__: Error callback function, invoked when error occurs. [optional]

## write

Writes data to the serial port.

    bluetoothSerial.write(data, success, failure);

### Description

Function `write` data to the serial port. Data can be an ArrayBuffer, string, array of integers, or a Uint8Array.

Internally string, integer array, and Uint8Array are converted to an ArrayBuffer. String conversion assume 8bit characters.

### Parameters

- __data__: ArrayBuffer of data
- __success__: Success callback function that is invoked when the connection is successful. [optional]
- __failure__: Error callback function, invoked when error occurs. [optional]

### Quick Example

    // string
    bluetoothSerial.write("hello, world", success, failure);

    // array of int (or bytes)
    bluetoothSerial.write([186, 220, 222], success, failure);

    // Typed Array
    var data = new Uint8Array(4);
    data[0] = 0x41;
    data[1] = 0x42;
    data[2] = 0x43;
    data[3] = 0x44;
    bluetoothSerial.write(data, success, failure);

    // Array Buffer
    bluetoothSerial.write(data.buffer, success, failure);

## subscribeRawData

Subscribe to be notified when data is received.

    bluetoothSerial.subscribeRawData(success, failure);

### Description

Function `subscribeRawData` registers a callback that is called when data is received. The callback is called immediately when data is received. The data is sent to callback as an ArrayBuffer. The callback is a long running callback and will exist until `unsubscribeRawData` is called.

### Parameters

- __success__: Success callback function that is invoked with the data.
- __failure__: Error callback function, invoked when error occurs. [optional]

### Quick Example

    // the success callback is called whenever data is received
    bluetoothSerial.subscribeRawData(function (data) {
        var bytes = new Uint8Array(data);
        console.log(bytes);
    }, failure);

## unsubscribeRawData

Unsubscribe from a subscription.

    bluetoothSerial.unsubscribeRawData(success, failure);

### Description

Function `unsubscribeRawData` removes any notification added by `subscribeRawData` and kills the callback.

### Parameters

- __success__: Success callback function that is invoked when the connection is successful. [optional]
- __failure__: Error callback function, invoked when error occurs. [optional]

### Quick Example

    bluetoothSerial.unsubscribeRawData();

## list

Lists bonded devices

    bluetoothSerial.list(success, failure);

### Description

#### Android

Function `list` lists the paired Bluetooth devices.  The success callback is called with a list of objects.

Example list passed to success callback.  See [BluetoothDevice](http://developer.android.com/reference/android/bluetooth/BluetoothDevice.html#getName\(\)) and [BluetoothClass#getDeviceClass](http://developer.android.com/reference/android/bluetooth/BluetoothClass.html#getDeviceClass\(\)).

    [{
        "class": 276,
        "id": "10:BF:48:CB:00:00",
        "address": "10:BF:48:CB:00:00",
        "name": "Nexus 7"
    }, {
        "class": 7936,
        "id": "00:06:66:4D:00:00",
        "address": "00:06:66:4D:00:00",
        "name": "RN42"
    }]

### Parameters

- __success__: Success callback function that is invoked with a list of bonded devices.
- __failure__: Error callback function, invoked when error occurs. [optional]

### Quick Example

    bluetoothSerial.list(function(devices) {
        devices.forEach(function(device) {
            console.log(device.id);
        })
    }, failure);

## isConnected

Reports the connection status.

    bluetoothSerial.isConnected(success, failure);

### Description

Function `isConnected` calls the success callback when connected to a peer and the failure callback when *not* connected.

### Parameters

- __success__: Success callback function, invoked when device connected.
- __failure__: Error callback function, invoked when device is NOT connected.

### Quick Example

    bluetoothSerial.isConnected(
        function() {
            console.log("Bluetooth is connected");
        },
        function() {
            console.log("Bluetooth is *not* connected");
        }
    );

## isEnabled

Reports if bluetooth is enabled.

    bluetoothSerial.isEnabled(success, failure);

### Description

Function `isEnabled` calls the success callback when bluetooth is enabled and the failure callback when bluetooth is *not* enabled.

### Parameters

- __success__: Success callback function, invoked when Bluetooth is enabled.
- __failure__: Error callback function, invoked when Bluetooth is NOT enabled.

### Quick Example

    bluetoothSerial.isEnabled(
        function() {
            console.log("Bluetooth is enabled");
        },
        function() {
            console.log("Bluetooth is *not* enabled");
        }
    );

## showBluetoothSettings

Show the Bluetooth settings on the device.

    bluetoothSerial.showBluetoothSettings(success, failure);

### Description

Function `showBluetoothSettings` opens the Bluetooth settings on the operating systems.

### Parameters

- __success__: Success callback function [optional]
- __failure__: Error callback function, invoked when error occurs. [optional]

### Quick Example

    bluetoothSerial.showBluetoothSettings();

## enable

Enable Bluetooth on the device.

    bluetoothSerial.enable(success, failure);

### Description

Function `enable` prompts the user to enable Bluetooth.

#### Android

`enable` is only supported on Android.

If `enable` is called when Bluetooth is already enabled, the user will not prompted and the success callback will be invoked.

### Parameters

- __success__: Success callback function, invoked if the user enabled Bluetooth.
- __failure__: Error callback function, invoked if the user does not enabled Bluetooth.

### Quick Example

    bluetoothSerial.enable(
        function() {
            console.log("Bluetooth is enabled");
        },
        function() {
            console.log("The user did *not* enable Bluetooth");
        }
    );

# Misc

## Where does this work?

### Android

Current development is done with Cordova 4.2 on Android 5. Theoretically this code runs on PhoneGap 2.9 and greater.  It should support Android-10 (2.3.2) and greater, but I only test with Android 4.x+.

Development Devices include
 * Nexus 5 with Android 5
 * Samsung Galaxy Tab 10.1 (GT-P7510) with Android 4.0.4 (see [Issue #8](https://github.com/don/BluetoothSerial/issues/8))
 * Google Nexus S with Android 4.1.2
 * Nexus 4 with Android 5
 * Samsung Galaxy S4 with Android 4.3

On the Arduino side I test with [Sparkfun Mate Silver](https://www.sparkfun.com/products/10393) and the [Seeed Studio Bluetooth Shield](http://www.seeedstudio.com/depot/bluetooth-shield-p-866.html?cPath=19_21). The code should be generic and work with most hardware.

I highly recommend [Adafruit's Bluefruit EZ-Link](http://www.adafruit.com/products/1588).

### Supporting other BLE hardware

For Bluetooth Low Energy, this plugin supports some hardware running known UART-like services, but can support any Bluetooth Low Energy hardware with a "serial like" service. This means a transmit characteristic that is writable and a receive characteristic that supports notification.

Edit [BLEdefines.h](src/ios/BLEDefines.h) and adjust the UUIDs for your service.

See [Issue 141](https://github.com/don/BluetoothSerial/issues/141#issuecomment-161500473) for details on how to add support for Amp'ed RF Technology BT43H.

## Props

### Android

Most of the Bluetooth implementation was borrowed from the Bluetooth Chat example in the Android SDK.

## Wrong Bluetooth Plugin?

If you don't need **serial** over Bluetooth, try the [PhoneGap Bluetooth Plugin for Android](https://github.com/phonegap/phonegap-plugins/tree/DEPRECATED/Android/Bluetooth/2.2.0) or perhaps [phonegap-plugin-bluetooth](https://github.com/tanelih/phonegap-bluetooth-plugin).

If you need generic Bluetooth Low Energy support checkout my [Cordova BLE Plugin](https://github.com/don/cordova-plugin-ble-central).

If you need BLE for RFduino checkout my [RFduino Plugin](https://github.com/don/cordova-plugin-rfduino).

## What format should the Mac Address be in?
An example a properly formatted mac address is ``AA:BB:CC:DD:EE:FF``

## Feedback

Try the code. If you find an problem or missing feature, file an issue or create a pull request.
