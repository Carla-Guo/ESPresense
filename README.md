# Deploying ESPresense on Seeed Studio XIAO ESP32-C3 with Home Assistant

This document describes how to integrate the open-source Bluetooth presence detection system, **ESPresense**, with the **Seeed Studio XIAO ESP32-C3**. The presence detection node based on the XIAO ESP32-C3 can work with the [MQTT Room component](https://www.home-assistant.io/components/sensor.mqtt_room/) of Home Assistant to enable indoor positioning.

---

## Project Overview

**ESPresense** is an open-source Bluetooth Low Energy (BLE) presence detection system designed to work with **Home Assistant** using **MQTT**.

In this migration:

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

### 2. Improved Reporting Responsiveness

To make data reporting more responsive, several parameters in `defaults.h` were tuned as follows:

#### A. Increased Movement Sensitivity

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

---

## Hardware Requirements

* Seeed Studio **XIAO ESP32-C3**
* USB-C cable (data capable)
* PC (Windows/macOS/Linux)
* Stable Wi-Fi network (2.4 GHz)
* Home Assistant instance

---

## Software & Tools

| Tool                               | Purpose                                               |
| ---------------------------------- | ----------------------------------------------------- |
| **PlatformIO**                     | Build system & flashing tool                          |
| ESP-IDF toolchain (via ESPresense) | Firmware build                                        |
| Web browser                        | Device configuration                                  |
| Home Assistant                     | Presence integration                                  |
| Mosquitto Broker (HA Add-on)       | MQTT backend                                          |
| MQTT Explorer (optional)           | MQTT debugging                                        |
| nRF Connect (Android only)         | BLE advertising                                       |

---


## 1. Home Assistant – MQTT Broker Setup

### Install Mosquitto Broker Add-on

- Open **Home Assistant**

   > If you haven't set up Home Assistant yet, please refer to this [guide](https://www.home-assistant.io/installation/) for configuration.

- Click **Settings → Add-ons → Add-on Store**
- Search for **Mosquitto broker**, install it. After installation:
   - Click **Start**
   - Enable:

     - ✔ Start on boot
     - ✔ Watchdog

### Create MQTT Credentials

- Go to **Configuration → Logins → Add**
- Create a dedicated MQTT user (recommended)
- Note:

   - Username
   - Password
   - Home Assistant IP address

    <img width="2365" height="1812" alt="image" src="https://github.com/user-attachments/assets/68f931d6-d24c-436e-9c69-3dbc06b011b1" />
  
    <img width="3170" height="1703" alt="image" src="https://github.com/user-attachments/assets/eae2c478-163c-4048-836c-4b721f4b44b6" />


---


## 2. Build & Flash ESPresense

- Download and open the [ESPresense code files](https://github.com/Carla-Guo/ESPresense). Wait for dependency installation and environment setup to complete.

   > If you haven't set up Platform IO yet, please refer to this [guide](https://docs.platformio.org/en/latest/integration/ide/vscode.html) for download.

- Select the **seeed_xiao_esp32c3** environment, select the correct port, build and upload.
  
  <img width="1920" height="1015" alt="image" src="https://github.com/user-attachments/assets/072622da-defd-4e7c-af3d-f2a8b1df3466" />

- Wait for the firmware flashing to complete, press the Reset button to restart the XIAO ESP32-C3.

---
## 3. Wi-Fi & MQTT Configuration (Captive Portal)

- Power the XIAO ESP32-C3
  
- On your PC or phone, open **Wi-Fi settings**, connect to the ESPresense AP (e.g. `ESPresense-XXXX`)
  
  <img width="400" height= auto alt="image" src="https://github.com/user-attachments/assets/4b2065b6-bdf1-485e-980b-fea265b34646" />

- Open browser: `http://192.168.4.1`
  
  <img width="3730" height="1928" alt="image" src="https://github.com/user-attachments/assets/b415ca5e-cc7c-41db-aaa6-7a61ec3eb071" />

- Fill in the following fields on the web page:

   | Field              | Description                            |
   | ------------------ | -------------------------------------- |
   | **Room Name**      | Logical room identifier (e.g. `room1`) |
   | **Wi-Fi SSID**     | 2.4 GHz network                        |
   | **Wi-Fi Password** | Network password                       |
   | **MQTT Broker IP** | Home Assistant IP                      |
   | **MQTT Port**      | `1883` (default)                       |
   | **MQTT Username**  | From Mosquitto setup                   |
   | **MQTT Password**  | From Mosquitto setup                   |

- Click **Save**
- After saving, restart the XIAO ESP32-C3. The device will automatically connect to WiFi and MQTT.

   > **Recommendation:**
   > Deploy **at least two XIAO ESP32-C3 nodes** in different rooms for meaningful presence comparison.

- Now, open **Home Assistant**, navigate to **Settings → Devices & Services**
  

- ESPresense nodes will appear automatically, device name format:

   ```
   ESPresense + <Room Name>
   ```
  <img width="3839" height="1822" alt="image" src="https://github.com/user-attachments/assets/15b729a8-fa42-4afe-a863-118bca700f24" />

  <img width="1920" height="877" alt="image" src="https://github.com/user-attachments/assets/6c18f249-01bb-4c31-af50-a328f77242fe" />

---

## 4. BLE Device Broadcasting

Next, we need to make your personal device discoverable by ESPresense via BLE. In some cases, you may need to install an app to enable your device to broadcast over BLE.

### iOS Devices

Apple devices emit various BTLE continuity messages, often identified by the fingerprint `apple:100?:*-*`.
>In households with multiple iPhones, the nearby info may collide, leading to duplicate fingerprints.You can refer to https://espresense.com/devices/apple to resolve the issue.

### Android Devices (Manual Advertising)

Android devices are typically tight lipped and need an app to get them to emit BLE advertisements. Thus, the need for an app to allow us to find it.

- Install **nRF Connect** and open the app.
- Go to **ADVERTISER → "+" → ADD RECORD**, select **Manufacturer Data**
- Fill in:

   | Field      | Value                                            |
   | ---------- | ------------------------------------------------ |
   | Company ID | `0x004C` (This is the fixed ID of Apple Inc., and all iBeacons must use this manufacturer ID.)|
   | Data       | `0215E2C56DB5DFFB48D2B060D0F5A71096E000010001C5` |

   **Data Format Explanation**

   | Section         | Description     |
   | --------------- | --------------- |
   | `0215`          | iBeacon prefix  |
   | `E2C56D...96E0` | UUID (16 bytes) |
   | `0001`          | Major           |
   | `0001`          | Minor           |
   | `C5`            | Measured Power  |

- Save and toggle advertising **ON**
  
  <img width="400" height= auto alt="image" src="https://github.com/user-attachments/assets/0d46eddc-968e-4f3f-8ee0-638afbcbc4a3" />   <img width="400" height= auto alt="image" src="https://github.com/user-attachments/assets/160594ad-27e5-41f0-9a5d-9d49742b328e" />



### Other supported devices:
[https://espresense.com/devices](https://espresense.com/devices)

---

### MQTT Data Verification

You can use [**MQTT Explorer**](https://mqtt-explorer.com/) to view all topics published by ESPresense.
-  Connect to:
   * Host: HA IP
   * Port: 1883
   * Username / Password

  <img width="1521" height="991" alt="image" src="https://github.com/user-attachments/assets/2f32c51a-3a4d-45c3-a627-46860cbd4ae2" />

  <img width="1920" height="944" alt="image" src="https://github.com/user-attachments/assets/4763b409-82b5-48f8-bc92-bc4111526698" />

-  Search using the **first 8 characters of your UUID**
  
  <img width="1920" height="944" alt="image" src="https://github.com/user-attachments/assets/3c953155-2c01-44c0-8412-3d9a8dbb1c87" />

      

---


## 5. Room Presence via MQTT Room Sensor

### Edit `configuration.yaml`

- Go to **Settings → Add-ons → Terminal**, open terminal
-  Run:

    ```bash
    ls /config
    ```
    Confirm `configuration.yaml` exists.

- Edit the file:

    ```bash
    nano /config/configuration.yaml
    ```
    <img width="1665" height="723" alt="image" src="https://github.com/user-attachments/assets/cd334a3f-2b13-45b4-af3f-89d4659125f0" />

- Append:

    ```yaml
    sensor:
      - platform: mqtt_room
        device_id: "iBeacon:e2c56db5-dffb-48d2-b060-d0f5a71096e0-1-1"
        name: "My iBeacon"
        state_topic: "espresense/devices/iBeacon:e2c56db5-dffb-48d2-b060-d0f5a71096e0-1-1"
        timeout: 60
        away_timeout: 120
    ```
    <img width="1888" height="877" alt="image" src="https://github.com/user-attachments/assets/124f720b-145f-47a9-a096-d8e80ecdfa74" />

>⚠ **YAML indentation is critical**
>* Use spaces only
>* Do not use TAB

- Save:

   * `Ctrl + O` → Enter
   * `Ctrl + X`

---

### Validate & Restart

- Go to **Settings → System → Developer Tools**
- Open **YAML** tab, click **Check Configuration**, if valid, click **Restart**

<img width="1754" height="749" alt="image" src="https://github.com/user-attachments/assets/29c7a513-5672-45b7-92de-208e72c87a06" />

---

## 6. Add entities to your dashboard

After restart:

- Go to **Developer Tools → States**
- Search for:

    ```
    sensor.my_ibeacon
    ```

- If the entity updates with room names, setup is complete
  
  <img width="1270" height="876" alt="image" src="https://github.com/user-attachments/assets/141503f0-2e59-46ac-a6b5-246279f8da2f" />

    >Sensor initialization may take some time. If the status shows not_home, please wait patiently for a while. If there is still no state update after an extended period, we recommend using MQTT Explorer to check whether ESPresense has gone offline or to verify that your personal device can be successfully scanned.

- Add the entity to your **Dashboard**
  
  <img width="1101" height="1292" alt="image" src="https://github.com/user-attachments/assets/fa639868-edfa-41e1-adb0-8f86405c6b10" />


---


