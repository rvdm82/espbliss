# espbliss: ESPHome config for Bliss Smart Blinds (ESP32)

An [ESPHome](https://esphome.io) configuration that controls **Hunter Douglas "Bliss Smart Blinds"**
motors (BLE name `HDxxxx`, e.g. HD1300) from an **ESP32** using the
[`ble_client`](https://esphome.io/components/ble_client/) component. The blind shows up in Home
Assistant as a cover with position control, plus battery, speed and diagnostic entities.

## Protocol source / credits

**All protocol information (GATT UUIDs, login frame, command bytes, status frame layout) comes from
the reverse-engineering work in [viggfred/blissble](https://github.com/viggfred/blissble)**
(see its [protocol notes](https://github.com/viggfred/blissble#protocol-notes), MIT licensed).
This project is an independent ESPHome port of that knowledge and is not affiliated with
viggfred/blissble or Hunter Douglas. Many thanks to the author.

## Features

- Template **cover**: open / close / stop / go-to-position (0-100 %)
- **Speed** select (100 / 75 / 50 / 25 %)
- Slow up / slow down (fine step) buttons and a manual Refresh button
- Battery text sensor (normal / low / none) and device-reported position sensor
- **On-demand connection** to save the motor's battery: connect, log in, send the command,
  drop the link after an idle timeout. Includes a retry loop, since the motor's BLE link is weak
  and often needs several attempts.

## Setup

1. Copy `secrets.yaml.example` to `secrets.yaml` and fill it in.
2. Edit the substitutions at the top of `bliss-blind.yaml`:
   - `blind_mac`: the motor's address (sticker in the head rail, or scan for the `HDxxxx` device)
   - `blind_range`: `1000` for HD1300-class motors, `100` for motors that use a single position byte
   - `invert_position`: set `"true"` if open and closed are swapped in Home Assistant
3. Build and flash: `esphome run bliss-blind.yaml`

Close the Bliss phone app (and switch off the phone's Bluetooth) while testing; the motor
accepts only one connection at a time. Keep the board close to the motor.

## Notes

- **Keep `auto_connect: true`.** With `auto_connect: false` the connection fails with status 133,
  most likely because the motor uses a random BLE address type that only the scanner path picks up.
  On-demand behaviour is implemented with `set_enabled()` instead.
- The motor does not report live movement direction, so the config polls status while moving and
  marks the cover idle once the position is stable.
- Speed is not reported by the motor, so the Speed select is optimistic.
- Not implemented: favorites, timers/schedules, clock sync.
- The motor's protocol has no real authentication or encryption (a fixed password shared by all
  units), so anyone in Bluetooth range can control the blind. That is a property of the device.
- The code is mostly AI generated.

## Status

> **Only tested on an HD1300 blind.** This code has been tested with a single HD1300 motor, using
> ESPHome 2026.10 on an ESP32-C6 and a standard ESP32. Other Bliss/HD-series models are untested and may behave
> differently (position range, status frame layout, etc.).

## License

MIT, see [LICENSE](LICENSE).
