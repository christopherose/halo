![Halo_v1](assets/Halo_v1.JPG)

# HALO with relaxed CO₂ colors

This is a fork of [yashmulgaonkar/halo](https://github.com/yashmulgaonkar/halo), the open-source HALO air quality sensor. For the hardware, parts list, build guide and everything else, see the **[original repo](https://github.com/yashmulgaonkar/halo)**.

This fork makes three changes to the LVGL firmware (new UI):

1. **Relaxed CO₂ colors**: yellow starts at 1200 ppm instead of 600.
2. **Auto Night Mode**: dims the ring and screen from sunset to sunrise.
3. **Fewer MQTT updates**: about 70% fewer Home Assistant database writes.

Everything else matches the upstream release it's built on ([see the diff](https://github.com/yashmulgaonkar/halo/compare/lvgl-2026.11.10.1...christopherose:halo:co2-thresholds)).

### Relaxed CO₂ colors

Stock firmware turns CO₂ yellow at 600 ppm, but a closed-up house with a few people in it normally sits at 800–1200 ppm, so the ring looks like something is wrong all winter.

| CO₂ (ppm) | Stock | This fork |
|---|---|---|
| ≤ 600 | 🟢 | 🟢 |
| 600–1000 | 🟡 | 🟢 |
| 1000–1200 | 🟠 | 🟢 |
| 1200–1500 | 🟠 | 🟡 |
| 1500–2000 | 🔴 | 🟡 |
| 2000–5000 | 🔴 | 🟠 |
| > 5000 | 🔴 | 🔴 |

The ring light, the on-screen CO₂ value and the device web page all use the new bands. The ring still shows the worse of AQI and CO₂.

### Auto Night Mode

From sunset to sunrise the ring drops to 5% and the screen to 15%. At sunrise they go back to where they were. If you change the brightness during the night, your change sticks. Night mode only ever dims, so a ring or screen that's already dimmer is left alone.

Turn it on from the HALO's setup page (`http://<halo-ip>/setup`): tick **Auto Night Mode** under *Ring* and click **Apply brightness**. Home Assistant also gets an *Auto Night Mode* switch.

Sunrise and sunset are calculated on the device, and they shift through the year on their own. Out of the box HALO guesses your location from its timezone, which can be off by an hour or so. For exact times, enter your **latitude and longitude** in the same form. To find them, right-click your house in [Google Maps](https://maps.google.com) and click the numbers at the top of the menu to copy them (for example `41.88, -87.63`). The setup page shows the next sunset and sunrise so you can check them.

### Fewer MQTT updates

Stock firmware publishes every reading every 10 seconds. Home Assistant writes a new database row each time a value changes, so tiny wobbles (PM2.5 going 0.5 → 0.6 → 0.5) add up to ~40,000 rows a day from one HALO. With this fork it's closer to 10,000.

This fork still reads the sensors every 10 seconds, but it only sends a value when it moves by more than a small amount:

| Reading | Sent when it changes by |
|---|---|
| PM1 / PM2.5 / PM4 / PM10 | 5% (at least 0.5 µg/m³) |
| CO₂ | 5 ppm |
| AQI | 1 (and only alongside a PM2.5 update, since AQI is calculated from PM2.5) |
| Temperature | 0.2° |
| Humidity | 1% |
| Everything else | any change |

Every 15 minutes, any value that has drifted by a smaller amount is sent anyway, so Home Assistant never falls behind. Everything is sent right away when MQTT reconnects or a setting changes.

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

---

## Home Assistant dashboard

[`home-assistant/halo-dashboard.yaml`](home-assistant/halo-dashboard.yaml) is a ready-made dashboard: a plain-English air verdict, AQI / PM2.5 / CO₂ gauges, 24-hour trends, a 30-day PM2.5 chart, comfort readings, and device controls (ring light, screen, SEN55 fan clean, CO₂ calibration, reboot). The CO₂ gauge uses the same 1200 / 2000 / 5000 bands as the ring.

**Setup**

1. Get HALO into Home Assistant over MQTT (Settings → Devices & services → MQTT). It shows up as **Halo Air Quality Sensor**.
2. Go to **Settings → Dashboards → Add dashboard → New dashboard from scratch**. Name it something like *Air Quality*.
3. Open the new dashboard, click the **pencil** (edit), then **⋮ → Raw configuration editor**.
4. Replace everything there with the contents of `halo-dashboard.yaml` and click **Save**.

**Notes**

- The entity IDs match a HALO with default names. If you renamed it, or this is your second HALO (Home Assistant adds `_2` to the end of each ID, e.g. `sensor.halo_air_quality_sensor_co2_2`), update the IDs in the file before pasting. You can find them under Settings → Devices & services → Entities.
- Builds with a **SEN54** instead of a SEN55 have no NOx sensor. The NOx tile hides itself, and the NOx line in the VOC graph stays empty (delete it if you like). The HALO web page shows which sensor you have.
- The 30-day PM2.5 chart fills in as history builds up.
- Only press **Calibrate CO₂** after HALO has sat outdoors in fresh air for 5+ minutes.
- Tested on Home Assistant 2026.11.
