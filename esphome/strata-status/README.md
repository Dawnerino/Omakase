# Strata inference and home-lab OLED displays

This subfolder documents an **ESPHome multi-display configuration fragment** that I use as a basis for visualizing Strata inference and home-lab infrastructure. The code is provided as an evolving engineering reference, not a ready-to-clone firmware image.

- [ESPHome YAML fragment](multi-oled.yaml)
- [Photo of the Strata inference display](../../images/esphome_inference_status.jpg)

## What it displays

**Channel 0 — Strata (SSD1306 128×64):** Inference phase (idle, prefill, thinking, decoding/generating), output speed in tokens per second, prefill progress, and a simple active-generation animation. Updated every 100 ms.

**Channel 1 — Home-lab dashboard (SSD1306 128×64):** Rotates between PowerBench, TrueNAS, and Vinabench views, showing utilization, temperature, and NAS pool summaries. A button on GPIO35 changes the page, and the page changes automatically after 10 seconds of inactivity.

**Channel 2 — Activity animation (SSD1306 128×32):** A vertical text-stream animation that turns on while Strata is actively generating or thinking.

## Hardware and connections

- ESP32 using **GPIO21 SDA** and **GPIO22 SCL** for the upstream 400 kHz I²C bus
- **TCA9548A I²C multiplexer** at address `0x70`
- Three OLEDs assigned to multiplexer channels 0, 1, and 2
- `GPIO35` page button to GND, **with an external 10 kΩ pull-up resistor to 3.3 V** (GPIO35 does not provide an internal pull-up)
- SSD1306 OLED addresses: `0x3C` on their independent mux buses

## Data flow

```text
Strata / GPU exporters / TrueNAS
              |
  Home Assistant sensor entities
              |
       ESPHome API
              |
    ESP32 + TCA9548A
        /     |     \
     Strata  Lab   Activity
      OLED   OLED   OLED
```

The YAML reads Home Assistant sensors directly. Entity IDs reflect the author's installation and **must be checked against your own Home Assistant configuration**. The two TrueNAS CPU/RAM entity IDs are explicit placeholders. The NAS pool display requires a `sensor.truenas_oled_pools` text sensor with a semicolon-separated `NAME=PERCENT` format (for example, `MAIN=67;MEDIA=81`).

## Using the example

1. Review `multi-oled.yaml` for I²C, fonts, sensors, the button, intervals, and three displays.
2. Merge the fragment into your own ESPHome node, supplying `esphome:`, `esp32:`, `wifi:`, `api:`, and OTA settings privately. **This fragment is not a complete standalone device YAML.**
3. Verify your I²C multiplexer wiring, display sizes, button pull-up, and Home Assistant sensor entity names.
4. Test configuration validation and flashing locally; no remote flashing or live Home Assistant changes are performed by this repository.
5. If compiling with a different ESPHome version, review any configuration syntax changes before using the fragment.

## Status

This is **work in progress**, and the multi-display code has not been independently compiled or hardware-validated as published here. Some sensor names and Home Assistant helpers are deployment-specific, and values will remain blank or unavailable without matching sources.

The code intentionally retains the display rendering lambdas, animations, and control logic from the documented configuration so that the implementation can be studied and adapted.
