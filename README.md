# PhaseShift

**Modular, open-source environmental chamber controller**

PhaseShift provides three programmable phases—**Ramp**, **Plateau**, and **Hold**—for precise control of drying, curing, fermenting, incubating, or any other staged environmental process. Built on [ESPHome](https://esphome.io), it's designed for reliability, modularity, and easy customization.

---

## New custom hardware coming soon!!

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
- Duration: 5 days
- Start Temp: 68.0°F
- End Temp: 65.0°F
- Start Dew Point: 58.0°F
- End Dew Point: 51.5°F
- Start Speed: 100%
- End Speed: 40%
- Start Duty: 60 sec/min
- End Duty: 20 sec/min

PLATEAU
- Duration: 5 days
- Temp: 65.0°F
- Dew Point: 51.7°F
- Speed: 10%
- Duty: 30 sec/min

HOLD
- Temp: 65.0°F
- Dew Point: 52.1°F
- Speed: 10%
- Duty: 30 sec/min

![image](https://github.com/user-attachments/assets/56ba16ce-8eea-4b9c-a955-2428b5891d26)

![image](https://github.com/user-attachments/assets/71b1ff27-052e-48f0-933e-4ac3f776e5a6)

## Comparing the reccomended settings to other options

![image](https://github.com/user-attachments/assets/1be32278-f660-48c5-a994-6938bbe87c0f)

![image](https://github.com/user-attachments/assets/888ac249-8401-4d0a-a034-2add6de52298)


# Climate Control Chamber Performance Analysis Report

**Data Period:** September 26-28, 2025  
**Total Measurements:** 256,875  
**Analysis Date:** October 5, 2025

---

## 🎯 Executive Summary

**Overall System Rating: EXCELLENT ✅**

All three control parameters achieve 100% accuracy within ±0.25°F tolerance, demonstrating exceptional temperature and humidity control performance across 48 hours of continuous operation.

---

## 📊 Performance Overview

### 1. Cold Side Temperature Tracking
**Target:** Dynamic setpoint (cold_heatsink_target_calculation)  
**Rating:** EXCELLENT ⭐⭐⭐⭐⭐

| Metric | Value |
|--------|-------|
| **Mean Tracking Error (Bias)** | 0.0000°F |
| **Mean Absolute Error (MAE)** | 0.0067°F |
| **Standard Deviation** | 0.0084°F |
| **Maximum Error** | 0.0397°F |
| **Minimum Error** | -0.0348°F |
| **Error Range** | 0.0745°F |
| **Matched Measurements** | 171,052 |

**Tolerance Performance:**
- ✅ Within ±0.25°F: **100.00%**
- ✅ Within ±0.50°F: **100.00%**
- ✅ Within ±1.00°F: **100.00%**

**Error Distribution Percentiles:**
- 50th percentile (median): 0.0056°F
- 95th percentile: 0.0166°F
- 99th percentile: 0.0220°F

**Key Findings:**
- Zero systematic bias - no temperature drift
- Exceptional tracking accuracy (MAE < 0.01°F)
- Perfect adherence to tight tolerance bands
- Target setpoint ranged from 48.595°F to 49.430°F (0.835°F span)
- System tracks dynamic setpoint changes with minimal lag

---

### 2. Dew Point Control
**Target:** 54.0°F  
**Rating:** EXCELLENT ⭐⭐⭐⭐⭐

| Metric | Value |
|--------|-------|
| **Target Temperature** | 54.000°F |
| **Actual Mean** | 53.996°F |
| **Mean Absolute Error (MAE)** | 0.0226°F |
| **Standard Deviation** | 0.0310°F |
| **Bias** | -0.004°F |
| **Temperature Range** | 53.878°F - 54.084°F |
| **Range Span** | 0.206°F |
| **Total Measurements** | 34,304 |

**Tolerance Performance:**
- ✅ Within ±0.25°F: **100.00%**
- ✅ Within ±0.50°F: **100.00%**
- ✅ Within ±1.00°F: **100.00%**

**Key Findings:**
- Minimal bias of -0.004°F (essentially zero)
- Outstanding humidity control stability
- Exceptionally tight precision (σ = 0.031°F)
- Perfect tolerance compliance across all measurements

---

### 3. Probe Temperature Control
**Target:** 68.0°F  
**Rating:** EXCELLENT ⭐⭐⭐⭐⭐

| Metric | Value |
|--------|-------|
| **Target Temperature** | 68.000°F |
| **Actual Mean** | 68.000°F |
| **Mean Absolute Error (MAE)** | 0.0077°F |
| **Standard Deviation** | 0.0097°F |
| **Bias** | 0.000°F |
| **Temperature Range** | 67.956°F - 68.042°F |
| **Range Span** | 0.087°F |
| **Total Measurements** | 34,319 |

**Tolerance Performance:**
- ✅ Within ±0.25°F: **100.00%**
- ✅ Within ±0.50°F: **100.00%**
- ✅ Within ±1.00°F: **100.00%**

**Key Findings:**
- Perfect mean accuracy (zero bias)
- Tightest control of all three parameters
- Best-in-class precision (σ = 0.0097°F)
- Minimal temperature variation (0.087°F total range)

---

## 📈 Comparative Analysis

### Error Metrics Ranking (Best to Worst)

**Mean Absolute Error:**
1. Cold Side Tracking: 0.0067°F ⭐
2. Probe Temperature: 0.0077°F
3. Dew Point: 0.0226°F

**Standard Deviation (Precision):**
1. Cold Side Tracking: 0.0084°F ⭐
2. Probe Temperature: 0.0097°F
3. Dew Point: 0.0310°F

**All parameters demonstrate excellent control well within engineering tolerances.**

---

## 🔬 Technical Methodology

### Cold Side Temperature Analysis Approach

The cold side temperature analysis was performed using a **time-synchronized matching algorithm**:

1. **Data Preparation:**
   - Sorted 171,052 cold_side_temperature measurements by timestamp
   - Sorted 17,184 cold_heatsink_target_calculation measurements by timestamp

2. **Matching Algorithm:**
   - For each cold side measurement, identified the most recent target setpoint using last-value-carried-forward (LVCF) method
   - This accounts for the ~10:1 ratio of temperature measurements to target updates

3. **Error Calculation:**
   - Tracking Error = Actual Cold Side Temperature - Target Setpoint
   - Computed for each of 171,052 matched pairs

4. **Statistical Analysis:**
   - Mean error, MAE, standard deviation
   - Percentile distribution
   - Tolerance band compliance

**Important Note:** This method provides the true tracking performance. A naive comparison to the mean target value would incorrectly suggest MAE = 0.145°F, which is **21.6× worse** than the actual performance of 0.0067°F.

---

## ✅ Conclusions

### System Strengths
1. **Exceptional Tracking Precision** - Cold side temperature follows dynamic setpoint with <0.01°F error
2. **Zero Systematic Bias** - All parameters show negligible long-term drift
3. **Perfect Tolerance Compliance** - 100% of measurements within ±0.25°F for all parameters
4. **Robust Performance** - Consistent control maintained over 256,875 measurements across 48 hours
5. **Multi-Parameter Excellence** - Simultaneous control of temperature and humidity at high precision

### Performance Grade: A+ (Excellent)

All control objectives have been met and exceeded:
- ✅ Cold side temperature tracks dynamic target setpoint: **0.0067°F MAE**
- ✅ Probe temperature maintained at 68.0°F: **0.0077°F MAE**
- ✅ Dew point maintained at 54.0°F: **0.0226°F MAE**

---

## 📁 Data Summary

| Parameter | Entity ID | Measurements |
|-----------|-----------|--------------|
| Cold Side Temperature | `sensor.phaseshift_1cda28_cold_side_temperature` | 171,052 |
| Cold Heatsink Target | `sensor.phaseshift_1cda28_cold_heatsink_target_calculation` | 17,184 |
| Probe Temperature | `sensor.phaseshift_1cda28_probe_temperature` | 34,319 |
| Dew Point | `sensor.phaseshift_1cda28_dew_point` | 34,304 |

**Time Range:** 2025-09-26 07:00:00 UTC to 2025-09-28 06:59:59 UTC (48 hours)

---

## 🛠️ Recommendations

The system is performing exceptionally well. Consider the following for future optimization:

1. **Documentation** - Record current control parameters/PID tuning for reference
2. **Monitoring** - Implement automated alerts if MAE exceeds 0.05°F threshold
3. **Long-term Analysis** - Track performance metrics over weeks/months to identify any degradation
4. **Validation** - Periodically verify sensor calibration to maintain accuracy

---

**Analysis performed using:**
- Data source: `history 1.csv`
- Analysis tool: Time-series statistical analysis with timestamp matching
- Total data points analyzed: 256,875

*Report generated: October 2025*



