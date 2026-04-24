# ESPHome Samsung AC IR Component

An ESPHome external component for Samsung split-type air conditioners using infrared control.

Tested on **Samsung AR12MCFHDWKYFE** and similar Samsung Wind-Free series units.


---

## Installation

### Option A — GitHub external component (recommended)

```yaml
external_components:
  - source: github://farhadbinfarooq/samsung-ac-wifi@main
    components: [samsung]
    
```

### Option B — Local component

Copy the `components/samsung/` folder into your ESPHome config directory under `custom_components/samsung/`, then:

```yaml
external_components:
  - source:
      type: local
      path: custom_components
    components: [samsung]
```

---

## Hardware Installation

You have two options for connecting the ESP32 to your Samsung AC.

---

### Method 1 — Non-Invasive (Recommended for beginners)

Build a separate IR transmitter circuit and connect a VS1838B IR receiver module to the ESP32 GPIOs. Place the ESP32 near the AC indoor unit so it can both receive signals from the physical remote and send commands to the AC IR sensor.

![Non-invasive wiring](assets/wiring_noninvasive.svg)

**Components needed:**
- IR LED (940nm, e.g. TSAL6400)
- BC547 NPN transistor
- 1kΩ resistor
- VS1838B IR receiver module

**Connections:**

| ESP32 Pin | Connects to |
|---|---|
| GPIO4 | 1kΩ → Base of BC547 |
| 3.3V | Collector → IR LED Anode |
| GND | Emitter of BC547, IR LED Cathode, VS1838B GND (pin 2) |
| GPIO15 | VS1838B OUT (pin 1) |
| 3.3V | VS1838B VCC (pin 3) |

---

### Method 2 — Invasive (Advanced, more stable)

Connect the ESP32 directly to the Samsung AC's IR board by tapping onto the IR LED signal trace. This bypasses the need for a separate IR circuit and gives much more reliable signal detection.

![Invasive wiring](assets/wiring_invasive.svg)

> ⚠️ **Warning:** Identify the correct solder pads using a multimeter before connecting. Wrong connections may damage your AC board or ESP32. The IR signal line on Samsung boards runs at 5V logic — use a voltage divider or logic level shifter if your ESP32 GPIO is not 5V tolerant.

**Solder pads to identify:**
- **5V** — powers the ESP32 via VIN
- **GND** — common ground
- **IR signal** — the line driving the IR LED on the board (shared TX+RX)

**YAML for invasive method** — same GPIO for TX and RX using open-drain:

```yaml
remote_receiver:
  id: ir_receiver
  pin:
    number: GPIO4
    inverted: true
    mode: OUTPUT_OPEN_DRAIN
    allow_other_uses: true
  tolerance: 55%
  idle: 5ms

remote_transmitter:
  pin:
    number: GPIO4
    inverted: true
    mode: OUTPUT_OPEN_DRAIN
    allow_other_uses: true
  carrier_duty_percent: 50%

climate:
  - platform: samsung
    name: "Remote Controller"
    receiver_id: ir_receiver
```

---

## Full Example Configuration (Non-Invasive)

```yaml
esphome:
  name: samsung-ac
  friendly_name: "Samsung AC"

esp32:
  board: esp32dev
  framework:
    type: arduino

external_components:
  - source: github://farhadbinfarooq/samsung-ac-wifi@main
    components: [samsung]
    

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
  ap:
    ssid: "Samsung-AC Fallback"

captive_portal:

api:
  encryption:
    key: !secret api_key

ota:
  - platform: esphome
    password: !secret ota_password

logger:
  level: DEBUG

remote_transmitter:
  pin: GPIO4
  carrier_duty_percent: 50%

remote_receiver:
  id: ir_receiver
  pin:
    number: GPIO15
    inverted: true
    mode:
      input: true
      pullup: true
  tolerance: 55%
  dump: all

climate:
  - platform: samsung
    name: "Remote Controller"
    receiver_id: ir_receiver
```

---

## Supported Features

| Feature | Supported |
|---|---|
| Cool | ✅ |
| Heat | ✅ |
| Dry | ✅ |
| Fan Only | ✅ |
| Auto | ✅ |
| Fan Speed (Auto / Low / Medium / High) | ✅ |
| Swing Off | ✅ |
| Swing Vertical | ✅ |
| Swing Horizontal | ✅ |
| Swing Both | ✅ |
| IR Receive (sync state from remote) | ✅ |
| Power On / Off | ✅ |
| Temperature range | 16°C – 30°C |

---

## Tested Models

| Model | Status |
|---|---|
| Samsung AR12MCFHDWKYFE | ✅ Tested |

If you test this on another Samsung model, please open an issue or PR to update this list.

---

