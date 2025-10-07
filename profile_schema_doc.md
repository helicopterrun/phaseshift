# PhaseShift JSON Profile Schema v1.0

## Overview

PhaseShift profiles are JSON documents that define complete environmental control recipes. Each profile specifies temperature, humidity, and fan parameters for three operational phases: **Ramp**, **Plateau**, and **Hold**.

## File Structure

```
profiles/
├── default.json              # Baseline profile (matches compile-time defaults)
├── cheddar_aging.json        # Example: cheese cave aging
├── thermal_stress_test.json  # Example: equipment validation
└── custom_profile.json       # Your custom profiles
```

## Schema Specification

### Root Object

```json
{
  "profile_metadata": { },      // Required: Profile identification
  "ramp_phase": { },            // Required: Initial transition phase
  "plateau_phase": { },         // Required: Intermediate stable phase
  "hold_phase": { },            // Required: Long-term stable phase
  "advanced_settings": { },     // Optional: Hardware tuning
  "profile_notes": { },         // Optional: Documentation
  "schema_version": "1.0",      // Required: Schema compatibility
  "checksum": "..."             // Optional: Validation hash
}
```

---

## Section Definitions

### 1. `profile_metadata` (Required)

Identifies the profile and provides usage context.

```json
"profile_metadata": {
  "name": "Profile Name",               // Display name (max 50 chars)
  "version": "1.0",                     // Semantic version
  "description": "Brief description",   // Purpose and use case
  "author": "Creator Name",             // Attribution
  "created": "2025-10-06",             // ISO 8601 date
  "temperature_unit": "fahrenheit",     // "fahrenheit" or "celsius"
  "tags": ["tag1", "tag2"]             // Searchable keywords
}
```

**Required fields:** `name`, `version`, `temperature_unit`

---

### 2. `ramp_phase` (Required)

Linear interpolation from start conditions to end conditions.

```json
"ramp_phase": {
  "description": "Phase purpose",       // Optional
  "duration_days": 5,                   // Integer 0-256
  "duration_hours": 0,                  // Integer 0-23
  "temperature": {
    "start": 68.0,                      // Float (unit from metadata)
    "end": 68.0,                        // Float
    "unit": "°F"                        // Display unit
  },
  "dewpoint": {
    "start": 58.0,                      // Float
    "end": 54.0,                        // Float
    "unit": "°F"
  },
  "circulation_fan": {
    "speed_start": 100,                 // Integer 0-100 (%)
    "speed_end": 40,                    // Integer 0-100 (%)
    "speed_unit": "%",
    "modulation_start": 60,             // Integer 0-60 (sec/min)
    "modulation_end": 30,               // Integer 0-60 (sec/min)
    "modulation_unit": "sec/min"
  }
}
```

**Behavior:** All parameters interpolate linearly based on elapsed time.

**Duration:** Total = (duration_days × 24) + duration_hours

---

### 3. `plateau_phase` (Required)

Static intermediate conditions between ramp and hold.

```json
"plateau_phase": {
  "description": "Phase purpose",       // Optional
  "duration_days": 5,                   // Integer 0-256
  "duration_hours": 0,                  // Integer 0-23
  "temperature": {
    "target": 68.0,                     // Float (static setpoint)
    "unit": "°F"
  },
  "dewpoint": {
    "target": 54.0,                     // Float (static setpoint)
    "unit": "°F"
  },
  "circulation_fan": {
    "speed": 10,                        // Integer 0-100 (%)
    "speed_unit": "%",
    "modulation": 30,                   // Integer 0-60 (sec/min)
    "modulation_unit": "sec/min"
  }
}
```

**Behavior:** All parameters remain constant during plateau phase.

---

### 4. `hold_phase` (Required)

Long-term stable storage conditions (no duration limit).

```json
"hold_phase": {
  "description": "Phase purpose",       // Optional
  "temperature": {
    "target": 68.0,                     // Float
    "unit": "°F"
  },
  "dewpoint": {
    "target": 55.0,                     // Float
    "unit": "°F"
  },
  "circulation_fan": {
    "speed": 10,                        // Integer 0-100 (%)
    "speed_unit": "%",
    "modulation": 30,                   // Integer 0-60 (sec/min)
    "modulation_unit": "sec/min"
  }
}
```

**Behavior:** Remains in this phase indefinitely until manually changed.

---

### 5. `advanced_settings` (Optional)

Hardware control parameters. **Modify with caution!**

```json
"advanced_settings": {
  "tec_control": {
    "max_current": 3.5,                 // Float 0.0-8.0 (Amps)
    "max_current_unit": "A",
    "max_current_limit": 8.0            // Hardware safety limit
  },
  "heatsink_control": {
    "cold_setpoint": 11.11,             // Float (°C only)
    "cold_setpoint_unit": "°C",
    "heating_range": 1.5,               // Float
    "cooling_range": -10.0,             // Float (negative)
    "cooling_alpha": 0.02,              // Float 0.0-0.25
    "heating_alpha": 0.04,              // Float 0.0-0.25
    "deadband": 0.1,                    // Float
    "bias": 2.0,                        // Float
    "wetting_force": 1.5                // Float
  },
  "safety_limits": {
    "frost_limit": 1.0,                 // Float (°C)
    "frost_limit_unit": "°C",
    "allow_below_freezing": false       // Boolean
  }
}
```

**Warning:** Invalid advanced settings can damage hardware or cause unsafe operation.

---

### 6. `profile_notes` (Optional)

Human-readable documentation and usage guidance.

```json
"profile_notes": {
  "usage": "How to use this profile",
  "modification_tips": [
    "Tip 1",
    "Tip 2"
  ],
  "compatible_applications": [
    "Use case 1",
    "Use case 2"
  ],
  "warnings": [
    "⚠️ Warning 1",
    "⚠️ Warning 2"
  ]
}
```

---

## Value Constraints

| Parameter | Type | Min | Max | Unit | Notes |
|-----------|------|-----|-----|------|-------|
| Temperature | Float | -100 | 100 | °F or °C | Based on metadata unit |
| Dewpoint | Float | -100 | 100 | °F or °C | Must be < temperature |
| Duration (days) | Integer | 0 | 256 | days | |
| Duration (hours) | Integer | 0 | 23 | hours | Added to days |
| Fan Speed | Integer | 0 | 100 | % | 0 = off, 100 = max |
| Fan Modulation | Integer | 0 | 60 | sec/min | 60 = continuous |
| TEC Current | Float | 0.0 | 8.0 | A | Hardware limit |
| Cold Setpoint | Float | -15.0 | 100.0 | °C | Always Celsius |
| Alpha values | Float | 0.0 | 0.25 | unitless | PID tuning |

---

## Temperature Unit Handling

**In Profiles:**
- All temperature values use the unit specified in `profile_metadata.temperature_unit`
- Display units (e.g., `"unit": "°F"`) are for documentation only

**In ESPHome:**
- Internal calculations always use Celsius
- Conversion happens at load time based on metadata
- Runtime F/C toggle switch changes display only

---

## Validation Rules

Profiles must pass these checks before loading:

### Required Field Validation
- [ ] `profile_metadata.name` exists
- [ ] `profile_metadata.temperature_unit` is "fahrenheit" or "celsius"
- [ ] All three phases (ramp, plateau, hold) exist
- [ ] Each phase has required temperature/dewpoint/fan objects

### Safety Validation
- [ ] TEC max_current ≤ 8.0 A
- [ ] Dewpoint < drybulb temperature in all phases
- [ ] Cold setpoint ≥ -15.0°C (unless below_freezing = true)
- [ ] Duration values within allowed ranges
- [ ] Fan speeds 0-100%
- [ ] Fan modulation 0-60 sec/min

### Logical Validation
- [ ] Ramp start/end values are sensible (not wildly different units)
- [ ] Phase durations are reasonable (warn if > 100 days)
- [ ] Advanced settings, if present, have valid structure

---

## Example Usage in ESPHome

```yaml
# Load profile button in Home Assistant
button:
  - platform: template
    name: "Load Default Profile"
    on_press:
      - lambda: |-
          // Read JSON file
          std::string json_content = R"(
            { "profile_metadata": { "name": "Default" }, ... }
          )";
          
          // Parse and apply (see profiles.yaml)
```

---

## Migration from Substitutions

| Old (Substitution) | New (JSON Path) |
|-------------------|-----------------|
| `ramp_start_temp` | `ramp_phase.temperature.start` |
| `ramp_duration` | `ramp_phase.duration_days` |
| `plateau_temp` | `plateau_phase.temperature.target` |
| `hold_dewpoint` | `hold_phase.dewpoint.target` |
| `default_max_current` | `advanced_settings.tec_control.max_current` |

---

## Best Practices

### Creating New Profiles
1. **Start with default.json** as a template
2. **Modify one parameter at a time** and test
3. **Document changes** in `profile_notes`
4. **Use descriptive names** and tags
5. **Include warnings** for non-standard settings

### Profile Organization
- **Descriptive filenames:** `cheddar_60day_aging.json`
- **Version control:** Increment version on changes
- **Backup originals:** Keep working copies separate

### Safety
- **Never exceed hardware limits** (8A TEC current)
- **Test new profiles** without sensitive materials first
- **Monitor first run** of any aggressive profile
- **Document tested ranges** in profile_notes

---

## Future Schema Extensions

Planned for v1.1+:
- Multi-cycle support (repeat phases)
- Conditional transitions (if/then logic)
- External sensor integration
- Profile inheritance (extend base profiles)
- Checksum validation for integrity

---

## Support

For schema questions or profile development help:
- GitHub: [PhaseShift Repository]
- Documentation: [Full System Guide]
- Community: [PhaseShift Forums]
