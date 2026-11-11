![Halo_v1](assets/Halo_v1.JPG)

# HALO with relaxed CO₂ colors

This is a fork of [yashmulgaonkar/halo](https://github.com/yashmulgaonkar/halo), the open-source HALO air quality sensor. For the hardware, parts list, build guide and everything else, see the **[original repo](https://github.com/yashmulgaonkar/halo)**.

This fork changes one thing in the LVGL firmware (new UI): the **CO₂ color bands**. Stock firmware turns CO₂ yellow at 600 ppm, but a closed-up house with a few people in it normally sits at 800–1200 ppm, so the ring looks like something is wrong all winter.

| CO₂ (ppm) | Stock | This fork |
|---|---|---|
| ≤ 600 | 🟢 | 🟢 |
| 600–1000 | 🟡 | 🟢 |
| 1000–1200 | 🟠 | 🟢 |
| 1200–1500 | 🟠 | 🟡 |
| 1500–2000 | 🔴 | 🟡 |
| 2000–5000 | 🔴 | 🟠 |
| > 5000 | 🔴 | 🔴 |

The ring light, the on-screen CO₂ value and the device web page all use the new bands. The ring still shows the worse of AQI and CO₂. Everything else matches the upstream release it's built on ([see the diff](https://github.com/yashmulgaonkar/halo/compare/lvgl-2026.11.10.1...christopherose:halo:co2-thresholds)).

---

## Flashing

Download the files from the **[latest release](https://github.com/christopherose/halo/releases/latest)**:

- `firmware.bin`: update for a HALO that's already running the LVGL firmware (keeps Wi-Fi, MQTT and bias settings)
- `firmware.factory.bin`: full image for a blank board, a board still on the old ESPHome firmware, or a full reflash

You'll need Python 3. Pick one of the options below.

### Option 1: Over Wi-Fi (already on the LVGL firmware)

No cables or disassembly. You need the HALO's IP address: check your router's device list, or in Home Assistant open the HALO device page and look at the *Visit* link.

```sh
curl -LO https://raw.githubusercontent.com/espressif/arduino-esp32/master/tools/espota.py
python3 espota.py -i <halo-ip> -p 3232 -f firmware.bin
```

Wait for `100% Done...`. The ring goes dark during the upload, then HALO restarts on the new firmware in about 30 seconds.

### Option 2: Over USB

```sh
pip install esptool
```

Plug the T-Display-S3 Long into your computer over USB-C, and close any browser tabs or apps using the serial port (web flashers, serial monitors).

**Update (keeps your settings):**
```sh
esptool --chip esp32s3 write-flash 0x10000 firmware.bin
```

**Full flash (blank board or coming from ESPHome):**
```sh
esptool --chip esp32s3 write-flash 0x0 firmware.factory.bin
```
After a full flash, HALO starts a Wi-Fi hotspot for setup, same as the official firmware.

If esptool can't find the board, it may need download mode: hold **BOOT**, tap **RST**, then release BOOT. If you pass `--port`, use the port name (macOS: `/dev/cu.usbmodem*`, Linux: `/dev/ttyACM0`, Windows: `COM3` or similar).

### Troubleshooting

- **Boot loop after a full flash** (serial shows `rst:0x7 (TG0WDT_SYS_RST)`): flash again in DIO mode:
  ```sh
  esptool --chip esp32s3 write-flash --flash-mode dio 0x0 firmware.factory.bin
  ```
- **Wrong Wi-Fi details, can't get back in:** erase the saved settings and HALO goes back to setup mode:
  ```sh
  esptool --chip esp32s3 erase-region 0x9000 0x5000
  ```
- **Want stock firmware back?** Use the update button in Home Assistant or on the HALO's web page, or the [official installer](https://github.com/yashmulgaonkar/halo).

> ⚠️ Updating from Home Assistant or the HALO's web page installs **stock** firmware and undoes this change. To stay on this fork, update from this repo's releases instead.
