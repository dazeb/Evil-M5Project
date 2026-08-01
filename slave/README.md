# ESP32 Slave and Standalone

## Overview

These firmwares run on ESP32 devices to operate as **slaves** (paired with the Cardputer) or as **stand-alone** tools alongside Evil-M5Project.

- **Wardriving Slave** — scan APs on static or hopping channels, forward via ESP-NOW to Wardriving Master
- **Sniffer Slave** — capture EAPOL handshakes and forward to Handshake Master
- **Auto-Deauther** — continuous deauth on static or hopping channels, can stimulate EAPOL capture
- **Evil Twin** — standalone captive portal with deauth (ESP32-C5 supports 2.4/5 GHz)
- **EvilChatMesh Relay** — ESP-NOW repeater to extend the mesh chat network
- **Portal** — standalone captive portal with WebUI configuration
- **NTLM Sniffer** — capture NTLM hashes and forward via webhook
- **FindMyEvil** — FindMy/OpenHaystack BLE advertiser
- **CSI-Beacon** — WiFi CSI transmitter for presence detection with CSI Radar
- **Serial Deauther** — UART-controlled deauther bridged to Cardputer

Multiple ESP32 devices improve coverage. Static slaves locked to one channel prevent packet loss during hopping. Devices with **external antennas** outperform stock modules.

## Firmware Matrix

### Generic ESP32 (2.4 GHz)

| Sketch | Role | Pairs With | Notes |
|--------|------|-----------|-------|
| `slave_deauth_hopping.ino` | Standalone deauther (hopping) | Optional Sniffer | Channel-hopping, configurable boardID |
| `slave_deauth_hopping_atoms3.ino` | Standalone deauther (hopping) | Optional Sniffer | AtomS3 with display status |
| `slave_deauth_static.ino` | Standalone deauther (static) | Optional Sniffer | Fixed channel deauth |
| `slave_gps_channel_static.ino` | Wardriving slave (static) | Wardriving Master | boardID = channel number |
| `slave_gps_hopping.ino` | Wardriving slave (hopping) | Wardriving Master | Hops across configured channels |
| `slave_gps_hopping_atoms3.ino` | Wardriving slave (hopping) | Wardriving Master | AtomS3 with display + avatar |
| `slave_sniffer_channel_static.ino` | EAPOL sniffer (static) | Handshake Master | Fragments via ESP-NOW |
| `slave_unified_C3.ino` | Multi-mode (deauth/ward/multi) | Master (serial) | ESP32-C3, UART1 GPIO6/7, serial-controlled |
| `slave_portal.ino` | Standalone captive portal | — | LittleFS + WebUI config |
| `EvilChatMesh-Relay.ino` | Mesh relay | EvilChatMesh | ESP-NOW repeater, dedup |
| `EvilChatMesh-Relay-atomS3.ino` | Mesh relay | EvilChatMesh | AtomS3 with display |
| `FindMyEsp.ino` | FindMy BLE advertiser | — | ESP32-S3, configurable TX power |
| `NTLM-Sniffer-WebHook-ESP32.ino` | NTLM hash sniffer + webhook | — | LittleFS config, WebUI |
| `NTLM-Sniffer-WebHook-NanoC6.ino` | NTLM hash sniffer + webhook | — | NanoC6 with NeoPixel LED |
| `2-4ghz-deauther-serial.ino` | Serial-controlled deauther | Cardputer (UART) | UART1 GPIO6/7, full CLI |
| `CSI-Beacon.ino` | CSI presence detection beacon | CSI Radar | Fixed MAC, ESP-NOW 100Hz, HT40 |

### ESP32-C5 (2.4 + 5 GHz Dual-Band)

| Sketch | Role | Pairs With | Notes |
|--------|------|-----------|-------|
| `C5-Slave/Evil-Twin-C5.ino` | Standalone Evil Twin | — | Dual-band portal + deauth |
| `C5-Slave/slave_deauther_C5.ino` | Standalone deauther | Optional Sniffer | WS2812 LED, serial control |
| `C5-Slave/slave_gps_channel_multi_C5_5GHz.ino` | Wardriving slave (dual-band) | Wardriving Master | Predefined 2.4+5GHz channels |
| `C5-Slave/slave_multi_C5.ino` | Multi-mode (scan/deauth/sniff) | Master (Evil) | All-in-one with LED status |
| `C5-Slave/slave_sniffer_channel_hopping_c5.ino` | EAPOL sniffer (hopping) | Handshake Master | Full 2.4+5GHz channel list |
| `C5-Slave/slave_unified_C5.ino` | Multi-mode (deauth/ward/multi) | Master (serial) | UART1 GPIO6/7, serial-controlled |

## CSI-Beacon

The CSI-Beacon firmware turns any ESP32 into a WiFi CSI transmitter for the [CSI Radar](CSI-Radar) feature on the Cardputer.

- **Best hardware**: ESP32-C3 (ESP32-C6 has [known CSI bugs](https://github.com/espressif/esp-idf/issues/14271))
- **Protocol**: ESP-NOW broadcast at 100Hz + AP fallback
- **Fixed MAC**: `1A:00:00:00:00:ID` per Espressif convention
- **Config**: HT40 bandwidth, MCS0 rate, channel 11
- **Serial commands**: `beacon id N`, `beacon status`, `beacon help`

See the [CSI Radar wiki page](CSI-Radar) for setup and placement details.

## Installation

**Tested with Arduino IDE 1.8.19** and official Espressif cores.

1. Install [Arduino IDE 1.x](https://www.arduino.cc/en/software)
2. Add ESP32 core URL in Preferences:
   ```
   https://espressif.github.io/arduino-esp32/package_esp32_dev_index.json
   ```
3. Select the appropriate board (ESP32-C3, ESP32-C5, ESP32-S3, etc.)
4. Install required libraries (Adafruit NeoPixel via Library Manager)
5. Apply [deauth prerequisites patch](https://github.com/7h30th3r0n3/Evil-M5Project/tree/main/utilities/deauth_prerequisites)
6. Open desired `.ino`, compile and flash

## How It Works

1. Deploy one or more ESP32 devices as slaves (static or hopping)
2. Each device scans APs, sniffs EAPOL, or deauths on assigned channels
3. Data is forwarded to the Cardputer via ESP-NOW or serial:
   - Wardriving Master aggregates AP data + GPS → CSV (Wigle-compatible)
   - Handshake Master aggregates EAPOL captures → `.pcap`

## Tested Hardware

- **AtomS3**: [Buy here](https://s.click.aliexpress.com/e/_DnDXSKJ)
- **AtomS3 Lite**: [Buy here](https://s.click.aliexpress.com/e/_Dm0e95D)
- **ESP32-C3** (with external antenna): [Buy here](https://s.click.aliexpress.com/e/_DD1yibp)
- **WEMOS D1 Mini**: [Buy here](https://s.click.aliexpress.com/e/_DEWPrnz)
- **ESP32-C5** (dual-band)
- **M5Stack NanoC6**

External antennas improve performance significantly.

## Disclaimer

> This code is for educational and research purposes only.
> Using sniffing, deauthentication, or Evil-Twin tools on networks without the owner's permission is illegal.
> **No liability is assumed** for misuse.
