# ESP32 voice node — “Okay Nabu”

This directory contains a **sanitized reference firmware configuration** for the Home Assistant / Omakase voice node. It demonstrates **on-device TensorFlow Lite Micro wake-word inference** via ESPHome's `micro_wake_word` component.

## Components and pinout

| Part | Signal | Example ESP32 GPIO |
| --- | --- | --- |
| INMP441 microphone | BCLK / SCK | 32 |
| INMP441 microphone | WS / LRCLK | 33 |
| INMP441 microphone | SD / DIN | 34 |
| MAX98357A amplifier | BCLK | 32 |
| MAX98357A amplifier | LRC | 33 |
| MAX98357A amplifier | DIN | 25 |

Power and grounding must match the actual boards. The INMP441 L/R selection pin must be set for the left channel; confirm the breakout wiring. GPIO34 is input-only, appropriate for microphone data. **Do not connect a speaker directly to an ESP32 GPIO; the MAX98357A drives it.**

This wiring comes from the previous device prototype; verify on the physical board before flashing.

## TensorFlow Lite model

```yaml
micro_wake_word:
  models:
    - model: okay_nabu
```

The official ESPHome `okay_nabu` model runs on the ESP32 using TensorFlow Lite Micro. ESPHome resolves the model manifest and TFLite asset during build. The model is **not included as an unlicensed copied binary** in this repository.

ESPHome docs: https://esphome.io/components/micro_wake_word/

## Try the example

1. Use a recent ESPHome release; the configuration targets the modern ESP-IDF and microWakeWord syntax.
2. Copy the contents of `secrets.example.yaml` into your local ESPHome `secrets.yaml` and replace placeholders. **Do not put the private file into this repository.**
3. Ensure your Home Assistant Assist pipeline has a configured STT, conversation agent, and TTS provider.
4. Run `esphome config esphome/omakase-voice.yaml` to check the YAML.
5. Run `esphome run esphome/omakase-voice.yaml` after confirming your actual pinout.
6. Pair the ESPHome device in Home Assistant and select the desired Assist pipeline.
7. Monitor logs for the wake phrase, listening stage, speech transcription, and response playback.

## Validation status and limitations

Wake-word detection had previously been observed in device logs, but **reliable post-wake-word audio capture was an unresolved issue**. This published YAML has not been compiled, flashed, or independently verified against the physical device.

The configuration intentionally omits unrelated OLED code and complex features to preserve resources on ESP32-WROOM-32. Sharing an I2S bus for input and output, memory pressure, and microphone channel configuration should be tested. If the speaker produces no sound or the microphone drops out, troubleshoot I2S bus allocation, amp power, and audio transitions before presenting the build as fully functional.

Suggested reproducible test: 20 wake attempts, then 20 spoken commands; record wake detection, STT accuracy, successful action, and latency. Do not equate successful wake-word detection with successful end-to-end voice capture.
