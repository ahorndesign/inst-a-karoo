# Project Brief: Karoo Insta360 BLE Remote Control App

## Goal
Build an Android app for the Hammerhead Karoo 2/3 that acts as a Bluetooth LE remote control for Insta360 action cameras, allowing a cyclist to start/stop video recording and take photos directly from their bike computer during a ride.

---

## Target Devices
- **Bike computer:** Hammerhead Karoo 2 or Karoo 3 (Android-based, runs Android 10+)
- **Camera (development/test):** Insta360 Ace Pro
- **Camera (target):** Insta360 X3, X4, Ace Pro, ONE RS (protocol is shared across these models)

---

## Platform: Karoo Extensions SDK
The Karoo runs a customised Android OS. Third-party apps are built using the **Karoo Extensions SDK**:
- GitHub: https://github.com/hammerheadnav/karoo-ext
- Apps are standard Android APKs, sideloaded or distributed via Hammerhead's app store
- The SDK provides Karoo-specific UI components (data fields, overlays) on top of standard Android APIs
- Standard Android BLE APIs are available and usable

---

## BLE Protocol: Impersonating the Insta360 GPS Remote

Insta360 cameras pair with a BLE peripheral that advertises itself as **"Insta360 GPS Remote"**. The app must impersonate this device name to be recognised by the camera.

### BLE Service & Characteristics (X3, confirmed working via community reverse engineering)
```
serviceUUID:         0000be80-0000-1000-8000-00805f9b34fb
writeCharacteristic: 0000be81-0000-1000-8000-00805f9b34fb
readCharacteristic:  0000be82-0000-1000-8000-00805f9b34fb
```

### Command Format
- First byte: command length (including itself)
- Next 16 bytes: command payload
- Optional: protobuf-encoded data appended
- BLE payload split into 20-byte blocks when sending
- Message counter (SN): 2-byte value starting at 0x0200, may not be strictly required

### Known Commands (Mode 0x04)
```
0x01  Enter app mode
0x02  Switch mode / close app mode
0x03  Take photo (single)
0x04  Start video (normal)
0x05  Stop video
0x29  Start Bullet Time video
0x30  Stop video (alternate)
0x33  Start HDR video
0x34  Stop HDR video
0x3D  Start TimeShift video
0x3E  Stop TimeShift video
0x45  Start loop recording
0x46  Stop loop recording
0x20  Reboot camera
0x39  Factory reset (DO NOT USE in normal operation)
0x18  Wipe SD card (DO NOT USE in normal operation)
```

### Notes on Commands
- Burst photo command is **not yet documented** — needs BLE sniffing with the Ace Pro to discover
- The Ace Pro is expected to use the same protocol as the X3 (confirmed by third-party remotes supporting both)
- Commands 0x19, 0x41, 0x43, 0x44 are known to freeze the camera and require a reboot — avoid

---

## Connection Flow
1. App scans for Insta360 camera by BLE device name or known service UUID
2. App connects to camera as a GATT client
3. App writes commands to `writeCharacteristic` (0000be81)
4. Camera responses come back on `readCharacteristic` (0000be82) — subscribe to notifications
5. A foreground Android service is required to maintain the BLE connection when the app is backgrounded

---

## Reference Implementations
- **ESP32 Arduino implementation (BLE control):** https://github.com/pchwalek/insta360_ble_esp32
- **Hackaday project (X3 BLE commands):** https://hackaday.io/project/188975-insta360-x3-ble-remote-control-with-esp32
- **Medium write-up (reverse engineering the GPS remote):** https://medium.com/@patrickchwalek/ble-control-of-insta360-cameras-7bf6894648a4
- **Garmin Connect IQ equivalent app** (proves the concept works on a cycle computer): https://apps.garmin.com/en-EN/apps/bb53f4bb-6c8c-4369-bca9-84cc09b25526
- **WiFi protocol reverse engineering** (useful for understanding protobuf structure): https://www.rigacci.org/wiki/doku.php/doc/appunti/hardware/insta360_one_rs_wifi_reverse_engineering

---

## Desired App Behaviour

### v1 Scope
- Karoo extension with a simple UI overlay or data field accessible during a ride
- Connect to a paired Insta360 camera on app launch (auto-reconnect if disconnected)
- **Start recording** button
- **Stop recording** button
- **Take photo** button
- Visual feedback showing connection state and current recording status
- Foreground service to maintain BLE connection throughout the ride

### v2 / Stretch Goals
- Mode selector: Normal Video / HDR Video / TimeShift / Photo
- Burst photo (requires discovering the command byte via BLE sniffing)
- Highlight marker during recording
- Camera battery/status display (if readable via BLE notifications)

---

## Tech Stack
- **Language:** Kotlin
- **Min SDK:** Android 10 (API 29)
- **BLE:** Android BluetoothLeGatt APIs (standard Android, no third-party BLE library required, though FastBLE or RxAndroidBle are acceptable if scaffolding is cleaner)
- **Karoo SDK:** karoo-ext (latest version from GitHub)
- **Build:** Gradle with Kotlin DSL preferred

---

## Key Android Permissions Required
```xml
<uses-permission android:name="android.permission.BLUETOOTH_SCAN" />
<uses-permission android:name="android.permission.BLUETOOTH_CONNECT" />
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" /> <!-- required for BLE scan on Android <12 -->
<uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
```

---

## What to Build First
1. Project scaffold as a Karoo Extension (follow karoo-ext README structure)
2. BLE service: scan → connect → maintain connection with auto-reconnect
3. Command sender: wraps the known command bytes in the correct frame format
4. Minimal Karoo UI: three buttons (record, stop, photo) + connection status indicator
5. Foreground service wrapper to keep BLE alive during rides

---

## Open Questions / Things That Need Empirical Testing
- Exact BLE advertisement payload the Ace Pro expects (may differ slightly from X3)
- Whether the Ace Pro responds to the same service UUID as the X3
- Burst photo command byte (not in community docs — needs sniffing)
- Whether a message counter (SN) is validated by the camera or ignored

---

## Filesystem Scope
Only read and write files within this project directory.
Do not access, modify or read files outside of this repository.
Do not read environment files, dotfiles, or system configuration.

---

## Dependencies
Do not install packages globally.
Do not modify system-level configuration.
Only add dependencies to this project's build files (build.gradle.kts).
Ask before adding any new third-party library dependency.

---

## Restrictions
Never delete files without explicit confirmation.
Never run destructive git operations (force push, rebase, reset --hard) without confirmation.
Never run the camera's factory reset or SD wipe commands in any test code.

---

## Network
Only fetch content from URLs explicitly provided in this brief or by me in the session.
Do not make requests to any other external services.