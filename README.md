<p align="center">
  <h1 align="center">EVVA Abrevva Android SDK</h1>
</p>

<p align="center">
  <a href="#quick-start"><img src="https://img.shields.io/badge/package-Gradle-fce500?logo=Gradle&logoColor=209BC4" alt="Package managers"></a>
  <a href="https://central.sonatype.com/artifact/com.evva.xesar/abrevva-sdk-android"><img alt="Maven Central Version" src="https://img.shields.io/maven-central/v/com.evva.xesar/abrevva-sdk-android"></a>
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-EVVA_License-yellow.svg?color=fce500&logo=data:image/svg+xml;base64,PCEtLSBHZW5lcmF0ZWQgYnkgSWNvTW9vbi5pbyAtLT4KPHN2ZyB2ZXJzaW9uPSIxLjEiIHhtbG5zPSJodHRwOi8vd3d3LnczLm9yZy8yMDAwL3N2ZyIgd2lkdGg9IjY0MCIgaGVpZ2h0PSIxMDI0IiB2aWV3Qm94PSIwIDAgNjQwIDEwMjQiPgo8ZyBpZD0iaWNvbW9vbi1pZ25vcmUiPgo8L2c+CjxwYXRoIGZpbGw9IiNmY2U1MDAiIGQ9Ik02MjIuNDIzIDUxMS40NDhsLTMzMS43NDYtNDY0LjU1MmgtMjg4LjE1N2wzMjkuODI1IDQ2NC41NTItMzI5LjgyNSA0NjYuNjY0aDI3NS42MTJ6Ij48L3BhdGg+Cjwvc3ZnPgo=" alt="EVVA License"></a>
</p>

The EVVA Abrevva Android SDK is a collection of tools to work with electronical EVVA access components. It allows for
scanning and connecting via BLE.

- [Features](#features)
- [Requirements](#requirements)
- [Installation](#installation)
- [Examples](#examples)

## Features

- BLE Scanner for EVVA components
- Localize scanned EVVA components
- Disengage scanned EVVA components
- Read / Write data via BLE

## Installation

[Gradle](https://gradle.org/) is a build automation tool for multi-language software development. For usage and
installation instructions, visit their website. To integrate EVVA Abrevva Android SDK into your Android Studio project
using Gradle, specify the dependency in your `build.gradle` file.

### Kotlin 2.x

**Android 11+ / API Level 30+**

```groovy
dependencies {
  implementation group: "com.evva.xesar", name: "abrevva-sdk-android", version: "4.0.0"
}
```

### Kotlin 1.9.x

**Android 10+ / API Level 29+**

```groovy
dependencies {
  implementation group: "com.evva.xesar", name: "abrevva-sdk-android", version: "3.2.3"
}
```

## Permissions

In your app's Manifest file add any needed install-time permissions:

```xml
<uses-permission android:name="android.permission.NFC" />
<uses-permission android:name="android.permission.BLUETOOTH_CONNECT"/>
<uses-permission android:name="android.permission.BLUETOOTH_SCAN"
                 android:usesPermissionFlags="neverForLocation"
                 tools:targetApi="s"/>
<uses-permission android:maxSdkVersion="30"
                 android:name="android.permission.BLUETOOTH"/>
<uses-permission android:maxSdkVersion="30"
                 android:name="android.permission.BLUETOOTH_ADMIN"/>
<uses-permission android:maxSdkVersion="30"
                 android:name="android.permission.ACCESS_COARSE_LOCATION"/>
<uses-permission android:maxSdkVersion="30"
                 android:name="android.permission.ACCESS_FINE_LOCATION"/>
```

## Caveats

In your app-level `build.gradle` you might want to exclude META-INF files to avoid gradle build errors:

```groovy
packagingOptions {
  resources.excludes.add("META-INF/*")
}
```

## Examples

### Initialize BleManager

To start off first initialize the SDK BleManager. You can pass an init callback closure for success indication.

```kotlin
import com.evva.xesar.abrevva.ble.BleDevice
import com.evva.xesar.abrevva.ble.BleManager

public class Example {
  private lateinit var bleManager: BleManager
  private var bleDeviceMap: MutableMap<String, BleDevice> = mutableMapOf()

  fun initialize() {
    this.bleManager = BleManager(context)
  }
}
```

### Scan for EVVA components

Use the BleManager to scan for components in range. You can pass several callback closures to react to the different
events when scanning or connecting to components.

```kotlin
fun scanForDevices() {
  val timeout: Long = 10_000

  this.bleManager.startScan(
    { device ->
      println("Found device: address=${device.address}")
      this.bleDeviceMap[device.address] = device
    },
    { success ->
      println("Scan started: success=${success}")
    },
    { success ->
      println("Scan stopped: success=${success}")
    },
    null, // optional EVVA mac filter
    null, // optional allow duplicates flag
    timeout
  )
}
```

### Read EVVA component advertisement

Get the EVVA advertisement data from a scanned EVVA component.

```kotlin
device.advertisementData?.let {
  println(it.rssi)
  println(it.isConnectable)

  it.manufacturerData?.let { md ->
    println(md.batteryStatus)
    println(md.isOnline)
    println(md.officeModeEnabled)
    println(md.officeModeActive)
    // ...
  }
}
```

There are several properties that can be accessed from the advertisement.

```kotlin
data class BleDeviceAdvertisementData(
  val rssi: Int,
  val isConnectable: Boolean? = null,
  val manufacturerData: BleDeviceManufacturerData? = null
)

data class BleDeviceManufacturerData(
  val companyIdentifier: UShort,
  val version: UByte,
  val componentType: UByte,
  val mainFirmwareVersionMajor: UByte,
  val mainFirmwareVersionMinor: UByte,
  val mainFirmwareVersionPatch: UShort,
  val componentHAL: Int,
  val batteryStatus: Boolean,
  val mainConstructionMode: Boolean,
  val subConstructionMode: Boolean,
  val isOnline: Boolean,
  val officeModeEnabled: Boolean,
  val twoFactorRequired: Boolean,
  val officeModeActive: Boolean,
  val reservedBits: Int?,
  val identifier: String,
  val subFirmwareVersionMajor: UByte?,
  val subFirmwareVersionMinor: UByte?,
  val subFirmwareVersionPatch: UShort?,
  val subComponentIdentifier: String?,
)
```

### Localize EVVA components

With the signalize method you can localize scanned EVVA components. On a successful signalization the component will emit a
melody indicating its location.

```kotlin
fun signalizeDevice(device: BleDevice) {
  this.bleManager.signalize(device) { success ->
    println("Signalize: success=$success")
  }
}
```

### Disengage EVVA components

For the component disengage you have to provide access credentials to the EVVA component. Those are generally acquired
in the form of access media metadata from the Xesar software.

```kotlin
fun disengageDevice(device: BleDevice) {
  val mobileId = ""           // sha256-hashed hex-encoded version of `xsMobileId` found in blob data.
  val mobileDeviceKey = ""    // mobile device key string from `xsMOBDK` found in blob data.
  val mobileGroupId = ""      // mobile group id string from `xsMOBGID` found in blob data.
  val mediumAccessData = ""   // access data string from `mediumDataFrame` found in blob data.
  val isPermanentRelease = false

  bleManager.disengage(
    device, // scanned EVVA component
    mobileId,
    mobileDeviceKey,
    mobileGroupId,
    mediumAccessData,
    isPermanentRelease,
  ) { status ->
    println("Disengage: status=$status")
  }
}
```

There are several access status types upon attempting the component disengage.

```kotlin
enum class DisengageStatusType {
  // Component
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
  ERROR,

  // Interface
  UNABLE_TO_CONNECT,
  UNABLE_TO_SET_NOTIFICATIONS,
  UNABLE_TO_READ_CHALLENGE,
  UNABLE_TO_WRITE_MDF,
  ACCESS_CIPHER_ERROR,
  BLE_ADAPTER_DISABLED,
  UNKNOWN_DEVICE,
  UNKNOWN_STATUS_CODE,
  TIMEOUT,
}
```

### Coding Identification Media

Use the CodingStation to write or update access data onto an EVVA identification medium.

```kotlin
class MainActivity {
  private lateinit var cs: CodingStation

  suspend fun writeMedium() {
    // Load credentials from service
    val cf: MqttConnectionOptionsTLS
    try {
      cf = AuthManager.getMqttConfigForXS(
        "url",          // host of the Xesar backend
        "clientId",     // coding station uuid from the Xesar backend
        "username",     // username
        "password"      // password
      )
    } catch (e: Exception) {
      println("Error getting auth credentials: $e")
      return
    }

    val cs = CodingStation(context)

    // Connect to coding station
    cs.connect(cf)
    // Wait for medium and start writing /w timeout
    cs.startTagReader(this, 5000)
    // Disconnect
    cs.disconnect()
  }

  // Needed to pass the found intent
  override fun onNewIntent(intent: Intent?) {
    super.onNewIntent(intent).also { cs.onHandleIntent(intent) }
  }
}
```
