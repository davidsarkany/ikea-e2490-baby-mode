# IKEA E2490 BILRESA Scroll Wheel — Home Assistant Blueprint

> ⚠️ **Personal-use fork** — This is a modified version of the original blueprint by [**mcinnes01**](https://github.com/mcinnes01), intended **solely for personal use**. It is not an official release, and no support or warranty is provided.

A unified Home Assistant automation blueprint for the **IKEA E2490 BILRESA** scroll wheel remote, supporting both **Zigbee2MQTT** and **ZHA** integrations. The blueprint maps the scroll wheel to four configurable modes — brightness, volume, color temperature, and hue — and exposes both short and double button presses for custom actions.

---

## Features

- **Dual integration support** — Works with Zigbee2MQTT (MQTT) and ZHA out of the box. All four discrete button triggers (`on`, `off`, `on_double`, `off_double`) plus raw payload/event triggers for scroll level values are included.
- **Scroll wheel modes**
  | Mode | Target domain | Description |
  |---|---|---|
  | Brightness | `light` | Maps wheel position to brightness (0–255) |
  | Volume | `media_player` | Maps wheel position to volume level (0.0 – `volume_max`) |
  | Color temperature | `light` | Maps wheel position across the light's mired range |
  | Hue | `light` | Maps wheel position to hue (0°–360°), preserving saturation |

  > **Brightness behaviour**: Each notch on the scroll wheel corresponds to a **3% brightness step**. The wheel's range maps to **1% (min)** – **53% (max)** brightness on the target light.

- **Runtime mode cycling** — Provide an `input_select` helper and double-press the **ON** button to cycle through your configured modes at runtime.
- **Auto-reset to default mode** — With an `input_datetime` helper, the blueprint can automatically reset to your default scroll mode after a configurable inactivity timeout.
- **Custom button actions** — All four button events (`on` short, `off` short, `on` double, `off` double) accept optional custom action sequences. When left empty, sensible defaults apply (toggle light/media_player on/off, or set brightness to max on double-ON).
- **Volume clamping** — An optional `volume_max` slider limits how loud the volume mode can go.

---

## Requirements

- Home Assistant with **Zigbee2MQTT** or **ZHA** integration set up.
- An IKEA E2490 BILRESA scroll wheel paired to your Zigbee network.
- Target entities depending on which modes you intend to use:
  - **Brightness / Hue / Color temp**: a `light` entity
  - **Volume**: a `media_player` entity
- *(Optional)* An `input_select` helper with the options `brightness`, `volume`, `color_temp`, `hue` (exact order and values) for runtime mode switching.
- *(Optional)* An `input_datetime` helper for auto-reset on inactivity.

---

## Installation

1. Copy `blueprints/automation/ikea-e2490-bilresa-scroll-wheel.yaml` into your Home Assistant config directory (preserving the folder structure), or use the **"Import Blueprint"** button in the Home Assistant UI.
2. Create a new automation from the blueprint.
3. Fill in the required inputs:
   - **Controller Device** — your E2490 device (Zigbee2MQTT or ZHA).
   - **Scroll wheel target entity** — the default `light` for brightness control.
   - **Default scroll mode** — the mode to use at startup and after auto-reset.
4. Optionally configure the other mode targets, the mode helper, the inactivity helper, and custom button actions.

---

## Inputs reference

| Input | Type | Required | Description |
|---|---|---|---|
| `controller_device` | device | ✅ | The E2490 device (MQTT or ZHA integration) |
| `scroll_target` | entity (`light`) | ✅ | Default light target for brightness mode |
| `scroll_mode` | select | ✅ | Default scroll mode: `brightness`, `volume`, `color_temp`, or `hue` |
| `volume_target` | entity (`media_player`) | ❌ | Media player for volume mode |
| `color_temp_target` | entity (`light`) | ❌ | Light for color temperature mode |
| `hue_target` | entity (`light`) | ❌ | Light for hue mode |
| `scroll_mode_helper` | entity (`input_select`) | ❌ | Helper for runtime mode cycling via double-ON |
| `last_activity_helper` | entity (`input_datetime`) | ❌ | Timestamp helper for auto-reset detection |
| `reset_timeout` | number | ❌ | Seconds of inactivity before resetting to default mode (default: 60, 0 = disabled) |
| `volume_max` | number | ❌ | Upper volume limit, 0.0–1.0 (default: 1.0) |
| `button_on_short` | action | ❌ | Custom action for short ON press |
| `button_off_short` | action | ❌ | Custom action for short OFF press |
| `button_on_double` | action | ❌ | Custom action for double ON press |
| `button_off_double` | action | ❌ | Custom action for double OFF press |

---

## How mode cycling works

When a `scroll_mode_helper` (`input_select`) is configured:

1. **Double-press ON** cycles the helper to the next available mode among those for which a target entity has been provided.
2. The active mode determines what the scroll wheel controls.
3. If `last_activity_helper` and `reset_timeout` are also configured, the mode automatically resets to the default after the specified period of inactivity.

If no helper is provided, double-ON falls back to setting the light to full brightness (or running your custom action).

---

## Button default behaviours

| Button | Default action (no custom action set) |
|---|---|
| ON short | Turns on the light or media player (depending on active mode) |
| OFF short | Turns off the light or media player |
| ON double | Cycles mode (if helper configured), otherwise sets brightness to 100% |
| OFF double | Turns off the light |

---

## Credits

- **Original author**: [mcinnes01](https://github.com/mcinnes01)
- **Forked & modified** for personal use. This version may contain changes that are not present upstream — use at your own risk.

---

## License

This project is a personal fork. Refer to the original repository for license information. No license is implied by this fork.
