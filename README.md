# AENO Eco Heater — Tuya Local device profile

A tested custom device profile for **[make-all/tuya-local](https://github.com/make-all/tuya-local)**, enabling local Home Assistant control of supported AENO Eco Heater devices with a display.

> **Not an official AENO or Tuya integration.** This repository contains a community-maintained device profile, not a standalone Home Assistant integration.

## Compatibility

| Property | Value |
| --- | --- |
| Device | AENO Eco Heater with display |
| Manufacturer | AENO |
| Tuya product ID | `ik5evzjpiz4hagr2` |
| Tuya Things Data Model ID | `000004yinq` |
| Home Assistant integration | `make-all/tuya-local` (not `localtuya`) |
| Validation | Tested on three real heaters (bedroom, kitchen, office) |

Other AENO hardware revisions and Tuya product IDs are **not** confirmed compatible.

## Features

The included `aeno_eco_heater.yaml` defines these 13 Home Assistant entities:

| Entity type | Feature | Tuya DP |
| --- | --- | --- |
| Climate | Heating on/off, target and current room temperature, heating/idle state | 1, 2, 3, 11 |
| Select | Smart / Manual operating mode | 4 |
| Lock | Child lock | 7 |
| Number | Panel temperature, **60–120 °C**, step 5 °C | 102 |
| Select | Backlight level: Off / Low / Medium / High | 108 |
| Select | Shutdown timer: Cancel or 1–24 hours | 19 |
| Sensor | Instantaneous electrical power (W) | 12 |
| Sensor | Total electrical energy (kWh; scale 1/1000) | 101 |
| Sensor | Outlet temperature (°C) | 105 |
| Sensor | Supply voltage (V; scale 1/10) | 109 |
| Sensor | Current (mA) | 110 |
| Sensor | Shutdown timer remaining (minutes) | 20 |
| Binary sensor | Fault indicator | 21 |

DP 106 (`LED brightness`) and DP 107 (`Display brightness`) are deliberately excluded: they did not cause a useful observable change on the tested devices. DP 10 (`light`) was not present in the local status payload; DP 108 controls the backlight instead.

## Installation (Home Assistant OS)

1. Install **Tuya Local by make-all** via HACS and configure the heater's local connection according to [Tuya Local documentation](https://github.com/make-all/tuya-local).
2. Download `aeno_eco_heater.yaml` from this repository.
3. Copy it to:
   ```text
   /config/custom_components/tuya_local/devices/aeno_eco_heater.yaml
   ```
   Depending on your file editor, `/config` may already be the root directory.
4. Restart Home Assistant to reload the custom integration and its device profiles.
5. In **Settings → Devices & services → Tuya Local**, add the heater and select **AENO Eco Heater (with display)** when offered. If the heater was already added under another device profile, back up its connection details and re-add it with the correct one.
6. Verify the climate entity, panel-temperature control, backlight selector, power and energy readings before relying on automations.

**Important:** Files placed inside `custom_components/tuya_local/devices/` can be overwritten by HACS updates. Keep a backup and restore the custom profile after updates if it is not yet included upstream.

## Example automation: backlight Low during the day, Off at night

The entities below are **examples from the tested installation**. Replace them with the entity IDs generated in *your* Home Assistant setup.

```yaml
alias: AENO LED Displays
triggers:
  - trigger: time
    at: "08:00:00"
  - trigger: time
    at: "22:00:00"
  - trigger: homeassistant
    event: start
    id: ha_start
conditions: []
actions:
  - if:
      - condition: trigger
        id: ha_start
    then:
      - delay:
          seconds: 30
  - choose:
      - conditions:
          - condition: time
            after: "22:00:00"
            before: "08:00:00"
        sequence:
          - action: select.select_option
            target:
              entity_id:
                - select.kabinet_aeno_kabinet_backlight_level
                - select.kukhnia_aeno_kukhnia_backlight_level
                - select.spalnia_aeno_spalnia_backlight_level
            data:
              option: "Off"
    default:
      - action: select.select_option
        target:
          entity_id:
            - select.kabinet_aeno_kabinet_backlight_level
            - select.kukhnia_aeno_kukhnia_backlight_level
            - select.spalnia_aeno_spalnia_backlight_level
        data:
          option: "Low"
mode: restart
```

## Notes and safety

- This profile communicates over the local network; **the heater itself may still connect to the Tuya cloud** unless independently restricted.
- Do not remove or re-pair the heater in its original app unless you are prepared for a possible local-key change.
- The panel temperature setting is **not a burn-safety guarantee**. Hot surfaces can cause injury, even at the minimum permitted setting. Follow the manufacturer's installation and clearance instructions, especially around children and people with limited mobility.
- The actual supported entities depend on the local DPS exposed by the heater firmware.
- The power and energy sensors are based on device reports; they have not been calibrated against an independent electricity meter.

## Contributing upstream

To submit this profile to the [Tuya Local project](https://github.com/make-all/tuya-local), propose adding `aeno_eco_heater.yaml` in its `custom_components/tuya_local/devices/` directory, together with the Tuya product ID and test results.

## Acknowledgments

Thanks to the maintainers of [make-all/tuya-local](https://github.com/make-all/tuya-local), and to the wider Home Assistant community.
