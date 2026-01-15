# Deploying ESPresense on Seeed Studio XIAO ESP32C3

This document describes how to integrate **ESPresense**—an open-source Bluetooth presence detection system—onto the **Seeed Studio XIAO ESP32-C3**. 
An XIAO ESP32-C3 based presence detection node for use with the [Home Assistant](https://www.home-assistant.io/) [`mqtt_room` component](https://www.home-assistant.io/components/sensor.mqtt_room/) for indoor positioning


## Recent Updates

### 1. Added XIAO ESP32-C3 Build Environment

The `.ini` configuration has been updated to add a dedicated build environment for **Seeed Studio XIAO ESP32-C3**. 

```ini
[env:seeed_xiao_esp32c3]
extends = esp32c3-cdc            ; XIAO ESP32-C3 uses native USB CDC
board = seeed_xiao_esp32c3
board_build.filesystem = spiffs
lib_deps =
  ${esp32c3.lib_deps}
  ${sensors.lib_deps}
build_flags =
  -D CORE_DEBUG_LEVEL=1
  -D FIRMWARE='"xiao-esp32c3"'   ; Firmware identifier
  -D SENSORS
  ${esp32c3-cdc.build_flags}
```

---

### 2. Improved Reporting Responsiveness

To make data reporting more responsive, several parameters in `defaults.h` were tuned as follows:

#### A. Increased Movement Sensitivity (Key Change)

The distance threshold that triggers an early report has been reduced from **0.5 m** to **0.1 m**. Even small movements will now activate the “early reporting” logic.

```cpp
#define DEFAULT_SKIP_DISTANCE 0.1  // changed from 0.5
```

#### B. Shortened Forced Reporting Interval

The maximum reporting interval has been reduced from **5 seconds** to **1–2 seconds**, ensuring more timely data updates even when movement is minimal.

```cpp
#define DEFAULT_SKIP_MS 1000       // changed from 5000, now is 1 second
```

#### C. Improved Wi-Fi and BLE Coexistence (2.4 GHz)

When using 2.4 GHz Wi-Fi (non-C5 chips), Bluetooth scanning is adjusted to avoid monopolizing the shared antenna. The BLE scan window is set slightly smaller than the scan interval, allowing Wi-Fi sufficient airtime for MQTT connections and data transmission.

```cpp
#define BLE_SCAN_INTERVAL 0x80  // 128
#define BLE_SCAN_WINDOW   0x60  // reduced from 0x80 (~75% duty cycle)
```

This change helps reduce internal contention and minimizes latency caused by radio resource preemption.


Below is a **full English technical wiki–style guide**, rewritten with clear structure, precise terminology, and step-by-step operational detail.
The tone, level of detail, and wording are aligned with what you would typically publish on **GitHub Wiki / ReadTheDocs / ESPresense Docs extensions**.

---


## 1. Project Overview

**ESPresense** is an open-source Bluetooth Low Energy (BLE) presence detection system designed to work with **Home Assistant** using **MQTT**.

In this migration:

* ESPresense is successfully compiled and deployed on **Seeed Studio XIAO ESP32C3**
* BLE scanning, Wi-Fi provisioning, and MQTT publishing work correctly
* Devices are automatically discovered by Home Assistant
* Room-level presence detection is achieved via `mqtt_room`

---

## 2. Fatures

* ✅ ESPresense builds and runs successfully on XIAO ESP32C3
* ✅ BLE, Wi-Fi, and MQTT modules compile without modification
* ✅ Web-based Wi-Fi/MQTT configuration portal works
* ✅ Seamless integration with Home Assistant
* ✅ BLE device detection verified via MQTT

---

## 3. Hardware Requirements

* Seeed Studio **XIAO ESP32C3**
* USB-C cable (data capable)
* PC (Windows/macOS/Linux)
* Stable Wi-Fi network (2.4 GHz)
* Home Assistant instance (HA OS / Supervised recommended)

---

## 4. Software & Tools

### Required Software

| Tool                               | Purpose                   |
| ---------------------------------- | ------------------------- |
| Git                                | Source code management    |
| ESP-IDF toolchain (via ESPresense) | Firmware build            |
| esptool                            | Flash erase & diagnostics |
| Web browser                        | Device configuration      |
| Home Assistant                     | Presence integration      |
| Mosquitto Broker (HA Add-on)       | MQTT backend              |
| MQTT Explorer (optional)           | MQTT debugging            |
| nRF Connect (Android only)         | BLE advertising           |

---

## 5. Flash Preparation (Erase Flash)

Before flashing ESPresense, **fully erase the device flash**.

### Steps

1. Connect XIAO ESP32C3 to your PC via USB
2. Open **PowerShell / Terminal**
3. Navigate to your ESPresense project directory
4. Run the following command:

```bash
esptool erase-flash
```

### Expected Output (Example)

```
esptool v5.1.0
Connected to ESP32-C3 on COM27:
Chip type:          ESP32-C3 (QFN32) (revision v0.4)
Features:           Wi-Fi, BT 5 (LE), Single Core, 160MHz
Embedded Flash:     4MB
Crystal frequency:  40MHz
USB mode:           USB-Serial/JTAG
MAC:                ec:da:3b:bd:ea:58

Flash memory erased successfully.
```

---

## 6. Home Assistant – MQTT Broker Setup

### Install Mosquitto Broker Add-on

1. Open **Home Assistant Web UI**
2. Click **Settings** (left sidebar)
3. Click **Add-ons**
4. Click **Add-on Store**
5. Search for **Mosquitto broker**
6. Click **Install**
7. After installation:

   * Click **Start**
   * Enable:

     * ✔ Start on boot
     * ✔ Watchdog

### Create MQTT Credentials

1. Go to **Settings → People → Users**
2. Create a dedicated MQTT user (recommended)
3. Note:

   * Username
   * Password
   * Home Assistant IP address

---

## 7. ESPresense Source Code

### Clone Repository

For development and customization, **clone the repository** instead of downloading ZIP files.

```bash
git clone https://github.com/Carla-Guo/ESPresense.git
cd ESPresense
```

Wait for dependency installation and environment setup to complete.

---

## 8. Build & Flash ESPresense

1. Build the firmware using the standard ESPresense build process
2. Flash firmware to the XIAO ESP32C3
3. After flashing completes successfully:

   * The device will reboot automatically

---

## 9. Wi-Fi & MQTT Configuration (Captive Portal)

### Connect to ESPresense AP

1. Power the XIAO ESP32C3
2. On your PC or phone:

   * Open **Wi-Fi settings**
   * Connect to the ESPresense AP (e.g. `ESPresense-XXXX`)
3. Open a browser
4. Navigate to:

```
http://192.168.4.1
```

---

### Configuration Fields

Fill in the following fields on the web page:

| Field              | Description                            |
| ------------------ | -------------------------------------- |
| **Room Name**      | Logical room identifier (e.g. `room1`) |
| **Wi-Fi SSID**     | 2.4 GHz network                        |
| **Wi-Fi Password** | Network password                       |
| **MQTT Broker IP** | Home Assistant IP                      |
| **MQTT Port**      | `1883` (default)                       |
| **MQTT Username**  | From Mosquitto setup                   |
| **MQTT Password**  | From Mosquitto setup                   |

> **Recommendation:**
> Deploy **at least two XIAO ESP32C3 nodes** in different rooms for meaningful presence comparison.

5. Click **Save**
6. Device reboots and connects to Wi-Fi & MQTT

---

## 10. Home Assistant Auto-Discovery

After successful connection:

1. Open **Home Assistant**
2. Navigate to **Settings → Devices & Services**
3. ESPresense nodes will appear automatically
4. Device name format:

```
ESPresense + <Room Name>
```

---

## 11. BLE Device Broadcasting

### iOS Devices

* iPhones **broadcast automatically**
* No additional setup required

### Android Devices (Manual Advertising)

Android devices do not broadcast by default.

#### Using nRF Connect

1. Install **nRF Connect** from Google Play
2. Open the app
3. Go to **ADVERTISER** tab
4. Tap **"+"**
5. Tap **ADD RECORD**
6. Select **Manufacturer Data**

Fill in:

| Field      | Value                                            |
| ---------- | ------------------------------------------------ |
| Company ID | `0x004C` (Apple)                                 |
| Data       | `0215E2C56DB5DFFB48D2B060D0F5A71096E000010001C5` |

**Data Format Explanation**

| Section         | Description     |
| --------------- | --------------- |
| `0215`          | iBeacon prefix  |
| `E2C56D...96E0` | UUID (16 bytes) |
| `0001`          | Major           |
| `0001`          | Minor           |
| `C5`            | Measured Power  |

7. Tap **DONE**
8. Open **Options**
9. Adjust **Advertising Interval**
10. Tap **Save**
11. Toggle advertising **ON**

Other supported devices:
[https://espresense.com/devices](https://espresense.com/devices)

---

## 12. MQTT Data Verification

### Using MQTT Explorer (PC)

1. Install **MQTT Explorer**
2. Connect to:

   * Host: HA IP
   * Port: 1883
   * Username / Password
3. Browse topics:

```
espresense/devices/
```

Search using the **first 8 characters of your UUID**

---

### Using Home Assistant MQTT Listener

1. Go to **Settings → Devices & Services**
2. Click **MQTT**
3. Click **Configure**
4. In **Listen to a topic**, enter:

```
espresense/devices/iBeacon:e2c56db5-dffb-48d2-b060-d0f5a71096e0-1-1/room1
```

Replace with your own device ID and room name.

---

## 13. Room Presence via MQTT Room Sensor

### Edit `configuration.yaml`

1. Go to **Settings → Add-ons → Terminal**
2. Open terminal
3. Run:

```bash
ls /config
```

Confirm `configuration.yaml` exists.

4. Edit the file:

```bash
nano /config/configuration.yaml
```

5. Append:

```yaml
sensor:
  - platform: mqtt_room
    device_id: "iBeacon:e2c56db5-dffb-48d2-b060-d0f5a71096e0-1-1"
    name: "My iBeacon"
    state_topic: "espresense/devices/iBeacon:e2c56db5-dffb-48d2-b060-d0f5a71096e0-1-1"
    timeout: 60
    away_timeout: 120
```

⚠ **YAML indentation is critical**

* Use spaces only
* Do not use TAB

6. Save:

   * `Ctrl + O` → Enter
   * `Ctrl + X`

---

### Validate & Restart

1. Go to **Settings → System → Developer Tools**
2. Open **YAML** tab
3. Click **Check Configuration**
4. If valid, click **Restart**

---

## 14. Validation

After restart:

1. Go to **Developer Tools → States**
2. Search for:

```
sensor.my_ibeacon
```

3. If the entity updates with room names, setup is complete
4. Add the entity to your **Dashboard**

---


