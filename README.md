# PhaseShift

**Modular, open-source environmental chamber controller**

PhaseShift provides three programmable phases—**Ramp**, **Plateau**, and **Hold**—for precise control of drying, curing, fermenting, incubating, or any other staged environmental process. Built on [ESPHome](https://esphome.io), it's designed for reliability, modularity, and easy customization.

---

## Features

- **Ramp** temperature, dewpoint, fan speed, and modulation linearly over a user-defined period
- **Plateau** at fixed setpoints for a steady mid-phase
- **Hold** final conditions indefinitely
- Fully **modular config**: packages split into `hardware`, `controls`, `sensors`, `climate`, `actuators`, `scripts`, and `timing`
- **Compile-time flexibility**: units (°F/°C), MAC-suffix device naming, adjustable step sizes
- **Three PID loops**: cold-side heatsink, chamber temperature, and chamber dewpoint
- **Icing detection** with automatic **defrost cycle**
- Tracks **elapsed time per phase** to drive dynamic logic

---

## Structure

Your ESPHome configuration is organized into discrete packages for clarity and reusability:

```
/config
├── esphome/
    ├── phaseshift.yaml         # main entrypoint
    ├── secrets.yaml            # sensitive credentials
    └── packages/
        ├── hardware_dallas.yaml     # for hardware using dallas based temperature sensors
        ├── hardware_tmp117.yaml     # for hardware using tmp117 based temperature sensors
        ├── controls.yaml            # template buttons, inputs, UI entities
        ├── sensors.yaml             # temperature, dewpoint, elapsed-time sensors
        ├── climate.yaml             # climate device definitions (PID loops)
        ├── actuators.yaml           # relays, fans, heaters, defrost logic
        ├── scripts.yaml             # custom lambda scripts & phase transitions
        └── timing.yaml              # schedules, timers, phase-duration logic
```

This layout splits out the main section of the code, so you can drop in or swap out modules without touching the rest of your config.

---

## Secrets

# It is highly recommended to use secrets

Store sensitive values in a separate `secrets.yaml` file at the root of your config directory.

```yaml
# secrets.yaml
wifi_ssid: "YOUR_WIFI_SSID"
wifi_password: "YOUR_WIFI_PASSWORD"
api_key: "YOUR_ESPHOME_API_KEY"
ota_pw: "YOUR_OTA_PASSWORD"
```

In your main YAML (`phaseshift.yaml`), reference these with the `!secret` tag:

```yaml
esphome:
  name: $device_id
  api:
    encryption_key: !secret api_key
  ota:
    password: !secret ota_pw

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password
```

---

## Substitutions

Below is a full list of all available substitutions in `phaseshift.yaml`, their default values, and what each controls:

```yaml
substitutions:
  device_id:  "phaseshift"              # Unique ESPHome node identifier (entity IDs & hostname)
  device_name: "PhaseShift"             # Human-readable name in Home Assistant
  use_fahrenheit: "true"                # Use Fahrenheit (true) or Celsius (false) for temperature values
  temp_unit: "°F"                       # Display suffix for temperature sensors
  default_max_current: "3.5"            # Default overcurrent trip point (amps) for TEC drivers

  ramp_duration: "8"                    # Length of the Ramp phase (days)
  plateau_duration: "4"                 # Length of the Plateau phase (days)

  ramp_start_dewpoint: "56.0"           # Starting dew point for Ramp phase (°F)
  ramp_end_dewpoint: "52.0"             # Ending dew point for Ramp phase (°F)
  ramp_start_temp: "68.0"               # Starting temperature for Ramp phase (°F)
  ramp_end_temp: "68.0"                 # Ending temperature for Ramp phase (°F)
  ramp_start_fan_speed: "100"           # Fan speed at start of Ramp (% output)
  ramp_end_fan_speed: "10"              # Fan speed at end of Ramp (% output)
  ramp_start_fan_modulation: "100"      # PWM modulation at start of Ramp (% duty cycle)
  ramp_end_fan_modulation: "10"         # PWM modulation at end of Ramp (% duty cycle)

  plateau_dewpoint: "52.0"              # Dew point during Plateau phase (°F)
  plateau_temp: "68.0"                  # Temperature during Plateau phase (°F)
  plateau_fan_speed: "10"               # Fan speed during Plateau (% output)
  plateau_fan_modulation: "10"          # PWM modulation during Plateau (% duty cycle)

  hold_dewpoint: "54.0"                 # Dew point during Hold phase (°F)
  hold_temp: "68.0"                     # Temperature during Hold phase (°F)
  hold_fan_speed: "10"                  # Fan speed during Hold (% output)
  hold_fan_modulation: "10"             # PWM modulation during Hold (% duty cycle)

  # Networking (optional static IP)
  static_ip: "192.168.50.224"           # Static IP address for the node (if used)
  gateway:   "192.168.50.1"             # Network gateway
  subnet:    "255.255.255.0"            # Subnet mask

  # I²C pins
  i2c_sda: "21"                         # GPIO pin for I²C SDA line
  i2c_scl: "22"                         # GPIO pin for I²C SCL line

  # I²C device addresses
  ads1115_address:  "0x49"              # I²C address for ADS1115 ADC
  mcp4728_address:  "0x60"              # I²C address for MCP4728 DAC

  # TEC driver control pins
  nsleep_a_pin:   "26"                  # GPIO pin for NSLEEP on TEC driver A
  nsleep_b_pin:   "18"                  # GPIO pin for NSLEEP on TEC driver B
  p_mode_pin:     "19"                  # GPIO pin to select PWM mode on drivers

  # TEC PWM outputs & frequency
  pwm_tec_a_pin:  "13"                  # PWM output pin for TEC channel A
  pwm_tec_b_pin:  "33"                  # PWM output pin for TEC channel B
  pwm_tec_freq:   "19531Hz"             # PWM frequency for TEC drivers

  # Fan PWM outputs & frequency
  pwm_fan_hot_pin:  "32"                # PWM pin for hot-side heatsink fan
  pwm_fan_cold_pin: "25"                # PWM pin for cold-side heatsink fan
  pwm_fan_circ_pin: "17"                # PWM pin for circulation fan
  pwm_fan_freq:     "25000Hz"           # PWM frequency for all fans

  # Heater relay pin
  heat_pin:       "12"                  # GPIO pin for heater relay control

  # Heatsink temperature sensor configuration
  dallas_pin:         "27"              # GPIO pin for 1-Wire bus (DS18B20 sensors)
  cold_temp_address:  "0x4B"            # 1-Wire address for cold heatsink sensor
  hot_temp_address:   "0x48"            # 1-Wire address for hot heatsink sensor

  # SNTP timezone
  timezone: "America/Los_Angeles"       # Timezone for SNTP synchronization
```

Each of these substitutions lets you tailor PhaseShift to your hardware, scheduling, and network environment without diving into the lower‑level packages.

## Reccomended Phase Structure for Cannabis Drying

| Phase       | Duration (Days) | Tdb Start → End (°F) | Td Start → End (°F) | Notes                                    |
| ----------- | --------------- | -------------------- | ------------------- | ---------------------------------------- |
| **Ramp**    | 7.0             | 68.0 → 64.4          | 58.1 → 51.8         | Smooth taper toward target aw (\~0.62)   |
| **Plateau** | 3.0             | 64.4 → 64.4          | 51.8 → 52.7         | Mild dewpoint recovery for equilibration |
| **Hold**    | ∞               | 64.4                 | 52.7                | Final holding phase for stabilization    |

| Phase       | Speed Start → End | Duty Start → End  | Notes                                    |
| ----------- | ----------------- | ----------------- | ---------------------------------------- |
| **Ramp**    | 100% → 40%        | 60s/min → 20s/min | Stronger early air exchange tapering off |
| **Plateau** | 10%               | 30s/min           | Gentle air movement for equilibration    |
| **Hold**    | 10%               | 30s/min           | Minimal airflow to avoid overdry risk    |

## Config Values to Enter in UI

RAMP
- Duration: 7 days

- Start Temp: 68.0°F

- End Temp: 64.4°F

- Start Dew Point: 58.1°F

- End Dew Point: 51.8°F

- Start Speed: 100%

- End Speed: 40%

- Start Duty: 60 sec/min

- End Duty: 20 sec/min

PLATEAU
- Duration: 3 days
- Temp: 64.4°F
- Dew Point: 52.7°F
- Speed: 10%
- Duty: 30 sec/min

HOLD
- Temp: 64.4°F
- Dew Point: 52.7°F
- Speed: 10%
- Duty: 30 sec/min




