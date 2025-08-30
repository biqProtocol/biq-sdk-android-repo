# BIQ Android SDK

![Latest Release](https://img.shields.io/github/v/tag/biqprotocol/biq-sdk-android-repo?label=latest&sort=semver)

The **BIQ Android SDK** is a lightweight library that enables **proof of presence** for Android applications using BLE beacons. When a user enters the proximity of one of our secure beacons and is authorized for a specific event, their presence is automatically validated.

---

## 🚀 Features

- Beacon scanning
- Secure presence validation through cryptographic signing
- Foreground/background scanning
- Boot auto-start support
- Notification support for beacon region entry
- State listeners for scanning, SDK state, and validation records
- Debugging tools and validation record inspection

---

## 📦 Installation

### Add the dependency

In your **module-level** `build.gradle.kts`:

```kotlin
implementation("biq.sdk.android:biq:<latest-version>")
```

### Configure repositories

In your **project-level** `settings.gradle.kts`:

```kotlin
pluginManagement {
    repositories {
        mavenCentral()
        maven {
            url = uri("https://biqprotocol.github.io/biq-sdk-android-repo/")
            content {
                includeGroup("biq.sdk.android")
            }
        }
    }
}

dependencyResolutionManagement {
    repositories {
        mavenCentral()
        maven {
            url = uri("https://biqprotocol.github.io/biq-sdk-android-repo/")
            content {
                includeGroup("biq.sdk.android")
            }
        }
    }
}
```

---

## 🔐 Required Permissions

Add the following permissions to your **AndroidManifest.xml**:

```xml
<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.FOREGROUND_SERVICE"/>
<uses-permission android:name="android.permission.POST_NOTIFICATIONS"/>
<uses-permission android:name="android.permission.ACCESS_BACKGROUND_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.BLUETOOTH_SCAN"/>
<uses-permission android:name="android.permission.BLUETOOTH_ADVERTISE"/>
<uses-permission android:name="android.permission.BLUETOOTH_CONNECT"/>
<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />
<uses-permission android:name="android.permission.FOREGROUND_SERVICE_LOCATION" />

<uses-feature android:name="android.hardware.bluetooth_le" android:required="true"/>
```

### Why These Permissions?

- **Location & Bluetooth**: Required for beacon detection and communication
- **Internet**: To communicate with the BIQ validation server
- **Foreground/Background Service**: Ensures scanning continues even when the app is not in the foreground
- **POST_NOTIFICATIONS**: To alert the user when entering a beacon region
- **RECEIVE_BOOT_COMPLETED**: Enables automatic scanning after device restarts

If using boot auto-scan, also add the following receiver:

```xml
<receiver
    android:name="biq.sdk.android.receivers.BootReceiver"
    android:enabled="true"
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.BOOT_COMPLETED" />
    </intent-filter>
</receiver>
```

---

## 🛠️ Usage

### Initialize the SDK

```kotlin
BiqController.Builder(this)
    .build()
    .setNotificationEntryPoint(this::class.java) // Optional
    .setBootAutoScan(true)                      // Optional, default: false
    .setDebugMode(true)                         // Optional, default: false
    .init(
        apiKey = "<your_api_key>",
        refreshToken = "<your_refresh_token>"
    )
```

### Start and Stop Presence Detection

```kotlin
BiqController.getInstance().startPresence()

BiqController.getInstance().stopPresence()
```

### Listening for State Changes

```kotlin
val listener = object : BiqController.BiqStateListener {
    override fun onSdkStateChanged(newState: BiqPresenceState) {
        // Handle SDK state change
    }

    override fun onScanningStateChanged(newState: BiqScanningState) {
        // Handle scanning state change
    }

    override fun onValidationRecordsChanged(newRecords: List<PresenceProofResponseDto>) {
        // Handle validation record updates
    }
}

BiqController.getInstance().setPresenceStateListener(listener)
```

### Permissions Handling

```kotlin
if (!BiqController.getInstance().areAllPermissionsGranted()) {
    val missingPermission = BiqController.getInstance().getMissingPermission()
    // Request this permission from the user
}
```

### Notifications

```kotlin
BiqController.getInstance().initNotificationData(
    iconResId = R.drawable.ic_notification, // Optional
    scanningNotificationTitle = "Scanning...",
    scanningNotificationMessage = "Looking for beacons nearby",
    proximityNotificationTitle = "Beacon detected!",
    proximityNotificationMessage = "Presence has been validated"
)
```

---

## 🧪 Debug Tools

### Load Debug Validation Records

```kotlin
lifecycleScope.launch {
    val records = BiqController.getInstance().loadDebugValidationRecords()
    // Use these to display presence history
}
```

### Clear Debug Validation Records

```kotlin
BiqController.getInstance().removeDebugValidationRecords()
```

### Query Validation Records from Database

```kotlin
lifecycleScope.launch {
    val records = BiqController.getInstance().getValidationRecords()
    // Process records
}
```

---

## 💬 Notes

- `.setNotificationEntryPoint(this::class.java)` sets the Activity that will open when the user taps the presence notification.
- `.setBootAutoScan(true)` will throw an exception if `RECEIVE_BOOT_COMPLETED` is not declared in the manifest.
- Debug mode is only active in debug builds (`BuildConfig.DEBUG == true`).

---

## 🧩 Dependencies

- AltBeacon (for beacon scanning)

---

## 📞 Support

For issues or feature requests, please contact [BIQ Support](https://www.biq.me).
