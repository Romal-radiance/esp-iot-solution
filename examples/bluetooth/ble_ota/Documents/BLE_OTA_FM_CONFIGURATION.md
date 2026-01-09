
# ESP BLE OTA (DFU) – README

This document describes the **ESP BLE OTA demo**, its protocol, ESP32-S3–specific configuration, and references for performing **DFU (Device Firmware Update) via a mobile application**.

---

## Overview

The **BLE OTA demo** is based on Espressif’s `ble_ota` component.  
It receives firmware over BLE and writes it to flash **sector-by-sector (4 KB)** until the OTA process is complete.

Supported targets include **ESP32, ESP32-S3, ESP32-H2, ESP32-C6**, with both **Bluedroid** and **NimBLE** hosts.

---

## 1. GATT Services Definition

The OTA component exposes **two BLE services**:

### 1.1 Device Information Service (DIS)
Used to display:
- Software version
- Hardware version

### 1.2 OTA Service
Used to perform firmware upgrade.

| Characteristic | UUID | Properties | Description |
|----------------|------|------------|-------------|
| RECV_FW_CHAR | 0x8020 | Write, Notify | Receives firmware packets and sends ACK |
| PROGRESS_BAR_CHAR | 0x8021 | Read, Notify | Reports OTA progress |
| COMMAND_CHAR | 0x8022 | Write, Notify | Sends OTA commands and receives ACK |
| CUSTOMER_CHAR | 0x8023 | Write, Notify | User-defined data channel |

---

## 2. Data Transmission Protocol

### 2.1 Command Packet Format

| Field | Command_ID | Payload | CRC16 |
|------|------------|---------|-------|
| Size | 2 bytes | 16 bytes | 2 bytes |
| Byte Index | 0–1 | 2–17 | 18–19 |

#### Command IDs
- **0x0001 – Start OTA**
  - Payload[2–5] → Firmware length
  - CRC16 calculated over bytes 0–17
- **0x0002 – Stop OTA**
- **0x0003 – Response**
  - Payload[2–3] → Command ID being acknowledged
  - Payload[4–5]:
    - 0x0000 → Accept
    - 0x0001 → Reject

---

### 2.2 Firmware Packet Format

| Field | Sector_Index | Packet_Seq | Payload |
|------|--------------|------------|---------|
| Size | 2 bytes | 1 byte | MTU-4 bytes |
| Byte Index | 0–1 | 2 | 3… |

Notes:
- Sector size is **4 KB**
- Sector index must be sequential (0, 1, 2…)
- Packet_Seq = **0xFF** indicates last packet of the sector
- Last packet contains **CRC16 of full 4 KB sector**

#### ACK Packet Format

| Field | Sector_Index | ACK_Status | CRC |
|------|--------------|------------|-----|
| Size | 2 bytes | 2 bytes | 2 bytes |

ACK_Status values:
- 0x0000 → Success
- 0x0001 → CRC error
- 0x0002 → Sector index error
- 0x0003 → Payload length error

---

## 3. Mobile DFU (BLE OTA via Phone)

### 3.1 Official Android BLE OTA App

Espressif provides an official Android application:

**Android App (APK):**  
https://github.com/EspressifApps/esp-ble-ota-android/releases

This app supports:
- BLE scanning
- Firmware selection (.bin)
- OTA progress monitoring

> iOS support is **not officially released** by Espressif for this demo.  
> For production-grade mobile DFU, ESP recommends **Protocomm** or **MCUboot + SMP**.

---

## 4. Sample ESP-IDF Code

**BLE OTA Example (Official):**  
https://github.com/espressif/esp-iot-solution/tree/master/examples/bluetooth/ble_ota

Recommended ESP-IDF version:
- **ESP-IDF v4.4 or later**

---

## 5. ESP32-S3 Specific Configuration (IMPORTANT)

When using **ESP32-S3**, additional configuration is mandatory.

### 5.1 NimBLE Configuration (Recommended)

Run:
```bash
idf.py menuconfig
 or
SDK Configuration Editor (menuconfig)
```

Set the following:

- **Component config → Bluetooth → Host**
  - Select **NimBLE – BLE only**

- **Flash Size**
  - Ensure firmware uses **4 MB flash**

---

### 5.2 Pre-Encrypted OTA

Firmware must be encrypted using RSA private key.

Menuconfig:
- Example Configuration → Type of OTA → Use Pre-Encrypted OTA
- Component config → OTA Manager → Enable pre-encrypted OTA

---
## 6. ESP32-C3 Specific Configuration

No external configuration required just select board and build the project as we do generally.

## 7. References

- ESP BLE OTA Example  
  https://github.com/espressif/esp-iot-solution/tree/master/examples/bluetooth/ble_ota

- Android BLE OTA App  
  https://github.com/EspressifApps/esp-ble-ota-android

- ESP-IDF Programming Guide  
  https://docs.espressif.com/projects/esp-idf/en/latest/esp32/

---
