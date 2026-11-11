![Halo_v1](assets/Halo_v1.JPG)

# HALO Home Assistant dashboard

A ready-made Home Assistant dashboard for [HALO](https://github.com/yashmulgaonkar/halo), the open-source air quality sensor. For the hardware, build guide and firmware, see the **[original repo](https://github.com/yashmulgaonkar/halo)**.

> **Looking for this fork's firmware?** You don't need it anymore. Its changes were merged into the official LVGL firmware, so just install the latest official release:
>
> - Relaxed CO₂ colors, now yellow at 1200, orange at 2000 and red at 3000 ppm ([#33](https://github.com/yashmulgaonkar/halo/pull/33))
> - About 70% fewer Home Assistant database writes from MQTT deadbands ([#34](https://github.com/yashmulgaonkar/halo/pull/34))
> - Auto Night Mode, which dims the ring and screen from sunset to sunrise ([#35](https://github.com/yashmulgaonkar/halo/pull/35))
>
> If you flashed a firmware release from this fork, updating from Home Assistant or the HALO's web page puts you on the official firmware with all of the above, and your settings carry over.

## Home Assistant dashboard

[`home-assistant/halo-dashboard.yaml`](home-assistant/halo-dashboard.yaml) is a ready-made dashboard: a plain-English air verdict, AQI / PM2.5 / CO₂ gauges, 24-hour trends, a 30-day PM2.5 chart, comfort readings, and device controls (ring light, screen, SEN55 fan clean, CO₂ calibration, reboot). The CO₂ gauge uses the same 1200 / 2000 / 3000 ppm bands as the ring.

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
