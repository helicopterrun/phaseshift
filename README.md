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
