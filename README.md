# ESPresense

![GitHub release (latest by date)](https://img.shields.io/github/v/release/ESPresense/ESPresense)
![GitHub all releases](https://img.shields.io/github/downloads/ESPresense/ESPresense/total)
[![.github/workflows/main.yml](https://github.com/ESPresense/ESPresense/actions/workflows/build.yml/badge.svg)](https://github.com/ESPresense/ESPresense/actions/workflows/build.yml)


An ESP32 based presence detection node for use with the [Home Assistant](https://www.home-assistant.io/) [`mqtt_room` component](https://www.home-assistant.io/components/sensor.mqtt_room/) or [ESPresense-companion](https://github.com/ESPresense/ESPresense-companion) for indoor positioning

**Documentation:** https://espresense.com/

**Building:** [building](./BUILDING.md).

**Release Notes:** [changelog](./CHANGELOG.md).



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

