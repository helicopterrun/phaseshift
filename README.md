PhaseShift
Modular, open-source environmental chamber controller
Provides three programmable phases—Ramp, Plateau, and Hold—for precise dry, cure, incubate, ferment, or other staged processes using ESPHome.

Features
Ramp your temperature, dewpoint, fan speed, and fan modulation linearly over a user-defined duration

Plateau at fixed setpoints for a second phase

Hold on final setpoints indefinitely

Fully modular config split into packages (hardware, controls, sensors, climate, actuators, scripts, timing)

Compile-time unit selection (°F/°C), automatic MAC-suffix naming, and customizable step-sizes

Three PID loops for cold-side heatsink, chamber dry-bulb, and chamber dewpoint control

Built-in icing detection and “defrost” cycle

Elapsed-time tracking for each phase

Quickstart
Clone the repo

bash
Copy
Edit
git clone https://github.com/yourusername/phaseshift.git
cd phaseshift
Install ESPHome (if needed)

bash
Copy
Edit
pip install esphome
Edit phaseshift.yaml

Set your Wi-Fi credentials (!secret wifi_ssid / !secret wifi_password)

Tweak the substitutions: block for your durations, ramp start/end values, units, and steps

Compile & Flash

bash
Copy
Edit
esphome phaseshift.yaml run
Add to Home Assistant via the ESPHome integration and enjoy your new chamber controller!

Configuration
All of the “knobs” you turn live in the top of phaseshift.yaml under substitutions:. Examples:

yaml
Copy
Edit
substitutions:
  # Device identity
  device_id:       "phaseshift"
  device_name:     "PhaseShift"

  # Phase durations (in days)
  ramp_duration:    "8"
  plateau_duration: "4"

  # Ramp endpoints (°F)
  ramp_start_temp:      "68.0"
  ramp_end_temp:        "52.0"
  ramp_start_dewpoint:  "56.0"
  ramp_end_dewpoint:    "52.0"

  # Fan ramp (% and sec/min)
  ramp_start_fan_speed:      "10"
  ramp_end_fan_speed:        "100"
  ramp_start_fan_modulation: "10"
  ramp_end_fan_modulation:   "60"

  # Units & step sizes
  temp_unit:   "°F"    # or "°C"
  temp_step_f: "0.1"   # UI step in °F
  temp_step_c: "0.1"   # Internal step in °C (if needed)
Each package (hardware.yaml, controls.yaml, etc.) pulls in these substitutions so you only change values in one place.

Project Structure
pgsql
Copy
Edit
phaseshift/
├── phaseshift.yaml      ← main entrypoint
├── .gitignore
├── LICENSE
├── README.md            ← this file
└── packages/
    ├── hardware.yaml    ← Wi-Fi, I²C, ADC/DAC, SNTP, API/OTA
    ├── controls.yaml    ← number/select/button definitions
    ├── sensors.yaml     ← template + hardware sensors
    ├── climate.yaml     ← PID loops
    ├── actuators.yaml   ← outputs & switches
    ├── scripts.yaml     ← automation scripts
    └── timing.yaml      ← interval & on_time automations
Optionally, you can add:

pgsql
Copy
Edit
docs/
├── wiring.md           ← wiring diagrams
└── usage.md            ← detailed examples & tips
Contributing
Fork the repo and create a new branch

Make changes (add a new sensor, tweak a script, etc.)

Test on real hardware or in the ESPHome dashboard

Submit a PR with a clear description and, if adding hardware, include a wiring diagram under docs/

License
This project is released under the MIT License.
© 2025 Your Name or Organization
