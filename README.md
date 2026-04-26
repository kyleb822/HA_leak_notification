# 💧 Leak Sensor Alert — Home Assistant Blueprint

A Home Assistant automation blueprint that sends **simultaneous push notifications** to up to two mobile devices and an **Alexa announcement** the moment a moisture sensor detects a leak. Designed to be reusable — set it up once, then create a new automation instance for every sensor in your home in seconds.

---

## Features

- Triggers on any `binary_sensor` with `device_class: moisture`
- 5-second sustained detection delay to prevent false positives
- Parallel push notifications to a primary and optional secondary mobile device
- Secondary notify service is optional — leave blank for single-device households
- Alexa announcement across a configurable device list
- Fully customizable notification titles, messages, and Alexa devices per instance
- `mode: single` to prevent duplicate alerts from rapid re-triggers

---

## Requirements

- [Home Assistant Companion App](https://companion.home-assistant.io/) installed on your mobile device(s)
- [Alexa Media Player](https://github.com/alandtse/alexa_media_player) HACS integration
- A script named `notify_alexa_script` that accepts `message`, `devices`, and `type` parameters

---

## Installation

### Option 1 — Import via URL (recommended)

Click the button below to import the blueprint directly into your Home Assistant instance:

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fkyleb822%2FHA_leak_notification%2Fblob%2Fmain%2Fleak_sensor_alert.yaml)

### Option 2 — Manual install

1. Download `leak_sensor_alert.yaml` from this repo
2. Place it in your Home Assistant config directory at:
   ```
   /config/blueprints/automation/custom/leak_sensor_alert.yaml
   ```
3. Restart Home Assistant or reload blueprints via **Developer Tools → YAML → Reload Automations**

---

## Configuration

Once imported, create a new automation from the blueprint via **Settings → Automations → + Create Automation → From Blueprint → Leak Sensor Alert**.

| Input | Required | Description | Example |
|---|---|---|---|
| `moisture_sensor` | ✅ | The `binary_sensor` moisture entity to monitor | `binary_sensor.kitchen_sink_leak_sensor_moisture` |
| `push_title` | ✅ | Title for the push notification | `⚠️ WATER LEAK - Kitchen Sink` |
| `push_message` | ✅ | Body of the push notification | `There is a leak under the kitchen sink!` |
| `alexa_message` | ✅ | Announcement text for Alexa devices | `There's a leak under the kitchen sink!` |
| `notify_primary` | ✅ | Notify service for your primary mobile device | `notify.mobile_app_your_iphone` |
| `notify_secondary` | ☑️ Optional | Notify service for a second mobile device | `notify.mobile_app_partner_iphone` |
| `alexa_devices` | ✅ | List of Alexa Media Player device IDs | See below |

### Finding your mobile notify service

In Home Assistant, go to **Developer Tools → Actions** and search for `notify.mobile_app` — you'll see one entry per device that has the Companion App installed.

### Finding your Alexa device IDs

The `alexa_devices` input takes a list of internal Alexa Media Player device identifiers (not entity IDs). Find them in the device registry under **Settings → Devices & Services → Alexa Media Player**, or by running this in **Developer Tools → Template**:

```jinja
{% for state in states.media_player %}
  {% if 'alexa' in state.entity_id %}
    {{ state.entity_id }}: {{ state.attributes.serial_number }}
  {% endif %}
{% endfor %}
```

---

## Example — Multiple Sensor Setup

Create one automation instance per sensor. Each instance is fully independent with its own sensor, messages, notify services, and Alexa device list:

```yaml
# Washer
alias: Leak - Washer
use_blueprint:
  path: custom/leak_sensor_alert.yaml
  input:
    moisture_sensor: binary_sensor.washer_leak_sensor_moisture
    push_title: "⚠️ WATER LEAK - Washer"
    push_message: "There is a leak behind the washer!"
    alexa_message: "There's a leak behind the washer in the laundry room!"
    notify_primary: notify.mobile_app_your_iphone
    notify_secondary: notify.mobile_app_partner_iphone
    alexa_devices:
      - "your_alexa_device_id_1"
      - "your_alexa_device_id_2"

# Kitchen Sink
alias: Leak - Kitchen Sink
use_blueprint:
  path: custom/leak_sensor_alert.yaml
  input:
    moisture_sensor: binary_sensor.kitchen_sink_leak_sensor_moisture
    push_title: "⚠️ WATER LEAK - Kitchen Sink"
    push_message: "There is a leak under the kitchen sink!"
    alexa_message: "There's a leak under the kitchen sink!"
    notify_primary: notify.mobile_app_your_iphone
    notify_secondary: notify.mobile_app_partner_iphone
    alexa_devices:
      - "your_alexa_device_id_1"
      - "your_alexa_device_id_2"
```

---

## Customization

### Adjust the detection delay

The default trigger delay is 5 seconds. To change it, edit the `for` block in the trigger section of the blueprint YAML:

```yaml
trigger:
  - platform: state
    entity_id: !input moisture_sensor
    from: "off"
    to: "on"
    for:
      seconds: 5  # change this value
```

### Single device only

Leave `notify_secondary` blank when creating the automation instance and the second notification will be skipped automatically.

---

## How It Works

```
Moisture sensor → off → on (sustained 5s)
        │
        ▼
  ┌──────────────────────────────────────┐
  │           Parallel actions            │
  │  • Push → Primary mobile device      │
  │  • Push → Secondary device (if set)  │
  └──────────────────────────────────────┘
        │
        ▼
  Alexa announcement on all configured devices
```

---

## License

MIT — free to use, modify, and share.
