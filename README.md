<p align="center">
  <h1 align="center">EVVA Abrevva Android SDK</h1>
</p>

<!-- TODO: update links -->
<p align="center">
  <a><img src="https://img.shields.io/github/v/tag/evva-sfw/abrevva-sdk-android?color=fce500" alt="Package managers"></a>
  <a href="#quick-start"><img src="https://img.shields.io/badge/package-Gradle-fce500?logo=Gradle&logoColor=209BC4" alt="Package managers"></a>
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-EVVA_License-yellow.svg?color=fce500&logo=data:image/svg+xml;base64,PCEtLSBHZW5lcmF0ZWQgYnkgSWNvTW9vbi5pbyAtLT4KPHN2ZyB2ZXJzaW9uPSIxLjEiIHhtbG5zPSJodHRwOi8vd3d3LnczLm9yZy8yMDAwL3N2ZyIgd2lkdGg9IjY0MCIgaGVpZ2h0PSIxMDI0IiB2aWV3Qm94PSIwIDAgNjQwIDEwMjQiPgo8ZyBpZD0iaWNvbW9vbi1pZ25vcmUiPgo8L2c+CjxwYXRoIGZpbGw9IiNmY2U1MDAiIGQ9Ik02MjIuNDIzIDUxMS40NDhsLTMzMS43NDYtNDY0LjU1MmgtMjg4LjE1N2wzMjkuODI1IDQ2NC41NTItMzI5LjgyNSA0NjYuNjY0aDI3NS42MTJ6Ij48L3BhdGg+Cjwvc3ZnPgo=" alt="EVVA License"></a>
</p>

The EVVA Abrevva Android SDK is a collection of tools to work with electronical EVVA access components. It allows for scanning and connecting via BLE.

- [Features](#features)
- [Requirements](#requirements)
- [Installation](#installation)
- [Examples](#examples)

## Features

- BLE Scanner for EVVA components in range
- Localize EVVA components encountered by a scan
- Disengage EVVA components encountered by a scan
- Read / Write data via BLE

## Requirements

| Platform                    |  Installation            | Status                  |
|-----------------------------|-------------------------| ------------------------ |
| iOS                         |see [EVVA Abrevva IOS SDK](https://github.com/evva-sfw/abrevva-sdk-ios-pod-specs) | -
| Android 10+ (API level 29)  | [Gradle](#Gradle)       | Fully Tested             |

## Installation

### Gradle

[Gradle](https://gradle.org/) is a build automation tool for multi-language software development. For usage and installation instructions, visit their website. To integrate EVVA Abrevva Android SDK into your Android Studio project using Gradle, specify the dependency in your `build.gradle` File:

```gradle
    repositories {
        maven {
            url = uri("https://maven.pkg.github.com/evva-sfw/abrevva-sdk-android")
        }
    }
    ...
    dependencies {
      implementation group: "com.evva.xesar", name: "abrevva-sdk-android", version: "1.0.18"
    }
```

## Examples

### Initialize BleManager

To start off first initialize the SDK BleManager. You can pass an init callback closure for success indication.

```kotlin
    import com.evva.xesar.abrevva.ble.BleManager
    import android.util.Log

    public class Example {
        private lateinit var bleManager: BleManager
        private var bleDeviceMap: MutableMap<String, BleDevice> = mutableMapOf()

        fun initialize() {
            this.bleManager = BleManager(context)
        }
    }
```

### Scan for EVVA components

Use the BleManager to scan for components in range. You can pass several callback closures to react to the different events when scanning or connecting to components.

```kotlin
  fun requestLeScan() {
    val timeout: Long = 10_000

    this.bleManager.startScan(
      { success ->
        Log.d("BleManager", "Scan started /w success=${success}")
      },
      { device ->
        Log.d("BleManager", "Found device /w address=${device.device.address}")
        this.bleDeviceMap[device.device.address] = device
      },
      { address ->
        Log.d("BleManager", "Connected to device /w address=${address}")

      },
      { address ->
        Log.d("BleManager", "Disconnected from device /w address=${address}")
      },
      timeout
    )
  }
```

### Localize EVVA component

With the signalize method you can localize EVVA components. On a successful signalization the component will emit a melody indicating its location.

```kotlin
suspend fun signalize(deviceID: String) {
  val device: BleDevice? = bleDeviceMap[deviceID]

  device?.let {
    this.bleManager.signalize(it) { success ->
      println("Signalized /w success=${it}")
    }
  }
}
```
### Perform disengage for EVVA components

For the component disengage you have to provide access credentials to the EVVA component. Those are generally acquired in the form of access media metadata from the Xesar software.

```kotlin
suspend fun disengage(deviceID: String) {
    val device: BleDevice? = bleDeviceMap[deviceID]
    if (device == null) {
        return
    }
    val mobileID = ""           // hex string
    val mobileDeviceKey = ""    // hex string
    val mobileGroupID = ""      // hex string
    val mobileAccessData = ""   // hex string
    val isPermanentRelease = false
    
    bleManager.disengage(
        deviceID,
        mobileID,
        mobileDeviceKey,
        mobileGroupID,
        mobileAccessData,
        isPermanentRelease,
    ) {
        println("Disengage /w status=$it")
    }
}
```
There are several access status types upon attempting the component disengage.
```kotlin
enum class DisengageStatusType {
    // Component
    ERROR,
    AUTHORIZED,
    AUTHORIZED_PERMANENT_ENGAGE,
    AUTHORIZED_PERMANENT_DISENGAGE,
    AUTHORIZED_BATTERY_LOW,
    AUTHORIZED_OFFLINE,
    UNAUTHORIZED,
    UNAUTHORIZED_OFFLINE,
    SIGNAL_LOCALIZATION,
    MEDIUM_DEFECT_ONLINE,
    MEDIUM_BLACKLISTED,

    // Interface
    UNKNOWN_STATUS_CODE,
    UNABLE_TO_CONNECT,
    TIMEOUT,
}
```
