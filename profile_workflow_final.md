# PhaseShift Profile Workflow - Complete Guide

## 🎯 The Simple Rule

**Profile controls everything.**

When you load a profile:
1. Profile declares its unit (`"temperature_unit": "fahrenheit"`)
2. System sets your F/C switch to match
3. All values display in that unit
4. You can flip the switch if you want different units
5. Export saves in whatever unit your switch is set to

---

## 📖 Complete Workflows

### **Workflow 1: Load and Use a Profile**

```
┌─────────────────────────────────┐
│ User clicks "Load Built-in:     │
│ Cheddar" button                  │
└──────────────┬──────────────────┘
               ↓
┌─────────────────────────────────┐
│ Profile declares:                │
│ "temperature_unit": "fahrenheit" │
└──────────────┬──────────────────┘
               ↓
┌─────────────────────────────────┐
│ System checks your switch        │
│ Currently: Celsius (OFF)         │
└──────────────┬──────────────────┘
               ↓
┌─────────────────────────────────┐
│ System flips switch ON           │
│ Log: "→ Switching to Fahrenheit" │
└──────────────┬──────────────────┘
               ↓
┌─────────────────────────────────┐
│ Profile loads values:            │
│ - Ramp Start: 55°F               │
│ - Ramp End: 54°F                 │
│ - Plateau: 54°F                  │
└──────────────┬──────────────────┘
               ↓
┌─────────────────────────────────┐
│ Display shows: 55°F, 54°F, etc   │
│ User sees Fahrenheit everywhere  │
└─────────────────────────────────┘
```

**Result:** Profile loads exactly as author intended (Fahrenheit values).

---

### **Workflow 2: Load Profile, Then Change Units**

```
┌─────────────────────────────────┐
│ Load "Cheddar Aging" profile    │
│ Switch auto-set to: Fahrenheit   │
│ Display shows: 55°F → 54°F       │
└──────────────┬──────────────────┘
               ↓
┌─────────────────────────────────┐
│ User: "I prefer Celsius"         │
│ User flips switch OFF            │
└──────────────┬──────────────────┘
               ↓
┌─────────────────────────────────┐
│ Display updates:                 │
│ - 55°F → 12.8°C                  │
│ - 54°F → 12.2°C                  │
└──────────────┬──────────────────┘
               ↓
┌─────────────────────────────────┐
│ User adjusts: "12.8°C → 13.0°C"  │
│ Makes other tweaks               │
└──────────────┬──────────────────┘
               ↓
┌─────────────────────────────────┐
│ User: "Save this as MY profile"  │
│ Clicks "Export Current Settings" │
└──────────────┬──────────────────┘
               ↓
┌─────────────────────────────────┐
│ JSON exported with:              │
│ "temperature_unit": "celsius"    │
│ "start": 13.0 (in Celsius)       │
└─────────────────────────────────┘
```

**Result:** User's modified profile is now in Celsius.

---

### **Workflow 3: Create Profile from Scratch**

```
┌─────────────────────────────────┐
│ User chooses preferred unit      │
│ Flips switch to: Fahrenheit      │
└──────────────┬──────────────────┘
               ↓
┌─────────────────────────────────┐
│ User manually sets all values:   │
│ - Ramp Start Temp: 70°F          │
│ - Ramp End Temp: 50°F            │
│ - Plateau Temp: 50°F             │
│ - etc...                         │
└──────────────┬──────────────────┘
               ↓
┌─────────────────────────────────┐
│ User tests the profile           │
│ Monitors for 24+ hours           │
│ Makes adjustments                │
└──────────────┬──────────────────┘
               ↓
┌─────────────────────────────────┐
│ User satisfied with results      │
│ Clicks "Export Current Settings" │
└──────────────┬──────────────────┘
               ↓
┌─────────────────────────────────┐
│ JSON logged with:                │
│ "temperature_unit": "fahrenheit" │
│ All values in °F                 │
└──────────────┬──────────────────┘
               ↓
┌─────────────────────────────────┐
│ User copies JSON from logs       │
│ Creates: my_salami_drying.json   │
│ Shares with community            │
└─────────────────────────────────┘
```

**Result:** New reusable profile created.

---

### **Workflow 4: Collaborative Profile Development**

```
User A (Fahrenheit)                    User B (Celsius)
        ↓                                      ↓
Creates "Blue Cheese" profile          Loads "Blue Cheese" profile
Exports in Fahrenheit                  Switch auto-set to Fahrenheit
        ↓                                      ↓
Shares JSON file                       Flips switch to Celsius
        ↓                                      ↓
                                       Makes tweaks in Celsius
                                       Exports in Celsius
                                               ↓
                                       Shares modified version
                                               ↓
User A loads the Celsius version
Switch auto-set to Celsius
Sees User B's changes
Can flip back to Fahrenheit if desired
```

**Result:** Profiles can be shared and modified regardless of unit preference.

---

## 🔑 Key Behaviors

### **Profile Loading**

| Profile Unit | User Switch Before | User Switch After | Display Shows |
|--------------|-------------------|------------------|---------------|
| Fahrenheit | OFF (Celsius) | **ON** (Fahrenheit) ✅ | °F values |
| Fahrenheit | ON (Fahrenheit) | ON (no change) | °F values |
| Celsius | ON (Fahrenheit) | **OFF** (Celsius) ✅ | °C values |
| Celsius | OFF (Celsius) | OFF (no change) | °C values |

**Log Messages:**
```
[profile] Profile declares unit: fahrenheit
[profile] → Switching display to Fahrenheit (profile requires it)
[profile] Ramp temp: 55.0 → 54.0
```

### **Profile Exporting**

| User Switch | Exported Unit | Values In | PIDs Receive |
|-------------|--------------|-----------|--------------|
| ON (°F) | `"fahrenheit"` | Fahrenheit | Celsius (converted) |
| OFF (°C) | `"celsius"` | Celsius | Celsius (direct) |

**Log Messages:**
```
[profile] ═══════════════════════════════════════════════
[profile]  EXPORTABLE JSON PROFILE
[profile]  Temperature Unit: fahrenheit
[profile] ═══════════════════════════════════════════════
[profile]   "temperature_unit": "fahrenheit",
[profile]     "start": 55.0,
```

---

## 🛡️ Safety Layer

**Hardware always receives Celsius:**

```
User Display (F or C)
        ↓
    submit_values button
        ↓
    Convert to Celsius (if needed)
        ↓
    drybulb_celcius sensor (always °C)
        ↓
    PID Controller (always °C)
        ↓
    Hardware (TEC, fans, heater)
```

**Advanced settings never convert:**
- `cold_hs_setpoint` always in °C
- `frost_limit` always in °C
- Profile can be F or C, but these stay C

---

## 💡 User Experience Examples

### **Example 1: American Cheesemaker**

"I think in Fahrenheit. All my recipes are in °F."

**Workflow:**
1. Create profile with switch ON (Fahrenheit)
2. Set ramp: 55°F → 50°F
3. Export → JSON in Fahrenheit
4. Share with other makers in US

**When loading:** Switch always goes to F, values show in F.

---

### **Example 2: European Maker**

"I use Celsius exclusively."

**Workflow:**
1. Load US profile (in Fahrenheit)
2. Switch auto-flips to F, shows 55°F → 50°F
3. User flips switch to C
4. Display updates: 12.8°C → 10.0°C
5. User adjusts to 13°C → 10°C
6. Export → JSON now in Celsius
7. Use this version going forward

**When loading:** Switch stays in C, values show in C.

---

### **Example 3: Scientific Application**

"I need precise control in Celsius."

**Workflow:**
1. Create profile with switch OFF (Celsius)
2. Set precise values: 12.50°C → 10.25°C
3. Export → JSON in Celsius with full precision
4. Load later → Switch set to C, exact values preserved

---

## 📊 Temperature Precision

| Unit | Precision | Example | Notes |
|------|-----------|---------|-------|
| Fahrenheit | 0.1°F | 68.3°F | Good for general use |
| Celsius | 0.1°C | 20.2°C | Better for precision |
| Celsius (advanced) | 0.01°C | 11.11°C | Hardware settings |

**Recommendation:** For cheese aging (± 2°F tolerance), either unit works. For scientific work, Celsius offers better precision.

---

## 🧪 Testing the System

### **Test 1: Profile Controls Switch**

```bash
# Watch logs while loading
esphome logs phaseshift_test.yaml

# Load Fahrenheit profile
# Expected:
[profile] Profile declares unit: fahrenheit
[profile] → Switching display to Fahrenheit (profile requires it)

# Load Celsius profile
# Expected:
[profile] Profile declares unit: celsius
[profile] → Switching display to Celsius (profile requires it)
```

### **Test 2: Unit Conversion**

```bash
# 1. Load Default (Fahrenheit) profile
# 2. Check display shows: 68°F
# 3. Flip switch to Celsius
# 4. Check display shows: 20°C
# 5. Verify PIDs receive 20°C (check logs)
```

### **Test 3: Export Preserves Units**

```bash
# 1. Switch to Fahrenheit
# 2. Set values manually
# 3. Click "Export Current Settings"
# 4. Check logs:
[profile]    "temperature_unit": "fahrenheit",
[profile]      "start": 70.0,

# 5. Switch to Celsius
# 6. Click "Export Current Settings"
# 7. Check logs:
[profile]    "temperature_unit": "celsius",
[profile]      "start": 21.1,
```

---

## ✅ Success Criteria

You know it's working correctly when:

1. **Loading Fahrenheit profile** → Switch goes ON, display shows °F
2. **Loading Celsius profile** → Switch goes OFF, display shows °C
3. **Flipping switch after load** → Display converts, PIDs still correct
4. **Exporting** → JSON unit matches current switch state
5. **Re-loading exported profile** → Restores exact same state
6. **Hardware** → Always receives Celsius regardless of display
7. **Cold setpoint** → Always shows °C, never converts

---

## 🔍 Troubleshooting

### **Problem: Switch doesn't change when loading profile**

**Check:**
```yaml
# In parse_and_apply_profile script, look for:
if (profile_is_fahrenheit && !current_switch) {
    id(use_fahrenheit).turn_on();  // ← This line should execute
}
```

**Debug:**
- Add more logging to see what profile declares
- Verify JSON has `"temperature_unit"` field
- Check switch state before/after load

---

### **Problem: Values seem wrong after loading**

**Verify the flow:**
1. Profile declares: `"fahrenheit"`
2. Switch changed to: ON (Fahrenheit)
3. Values loaded: 68.0
4. Display shows: 68.0°F ✅

**Not:**
1. Profile declares: `"fahrenheit"`
2. Switch stays: OFF (Celsius)
3. Values loaded: 68.0
4. Display shows: 68.0°C ❌ Wrong!

**Fix:** Make sure switch-changing logic runs before value loading.

---

### **Problem: Export shows wrong unit**

**Check order:**
```cpp
// CORRECT:
std::string unit = id(use_fahrenheit).state ? "fahrenheit" : "celsius";
ESP_LOGI("profile", "\"temperature_unit\": \"%s\",", unit.c_str());

// WRONG:
// Hardcoded: "temperature_unit": "fahrenheit"
```

**Fix:** Use dynamic unit based on switch state.

---

### **Problem: PIDs receiving wrong values**

**Verify conversion happens:**
```yaml
button:
  - platform: template
    id: submit_values
    on_press:
      - lambda: |-
          float value = id(drybulb_setpoint).state;
          if (id(use_fahrenheit).state) {
            value = (value - 32.0) * 5.0 / 9.0;  // ← MUST convert F→C
          }
          id(drybulb_celcius).publish_state(value);
```

**Test:**
- Set drybulb to 68°F (switch ON)
- Click submit
- Check `drybulb_celcius` sensor shows 20.0°C

---

## 📚 JSON Profile Examples

### **Fahrenheit Profile (Most Common)**

```json
{
  "profile_metadata": {
    "name": "Cheddar Cave Aging",
    "version": "1.0",
    "description": "Traditional cave-aged cheddar",
    "temperature_unit": "fahrenheit",
    "tags": ["cheese", "cheddar", "aging"]
  },
  "ramp_phase": {
    "duration_days": 3,
    "duration_hours": 12,
    "temperature": {
      "start": 55.0,
      "end": 54.0,
      "unit": "°F"
    },
    "dewpoint": {
      "start": 52.0,
      "end": 48.0,
      "unit": "°F"
    },
    "circulation_fan": {
      "speed_start": 100,
      "speed_end": 30,
      "modulation_start": 60,
      "modulation_end": 20
    }
  },
  "plateau_phase": {
    "duration_days": 14,
    "temperature": {"target": 54.0},
    "dewpoint": {"target": 48.0},
    "circulation_fan": {"speed": 15, "modulation": 25}
  },
  "hold_phase": {
    "temperature": {"target": 54.0},
    "dewpoint": {"target": 49.0},
    "circulation_fan": {"speed": 10, "modulation": 20}
  },
  "advanced_settings": {
    "tec_control": {"max_current": 3.0},
    "heatsink_control": {
      "cold_setpoint": 10.0,
      "cold_setpoint_unit": "°C",
      "bias": 2.5,
      "wetting_force": 1.8
    },
    "safety_limits": {
      "frost_limit": 2.0,
      "frost_limit_unit": "°C",
      "allow_below_freezing": false
    }
  }
}
```

**When loaded:**
- Switch → ON (Fahrenheit)
- Display → 55°F, 54°F, 48°F, etc.
- PIDs receive → 12.8°C, 12.2°C, 8.9°C (converted)

---

### **Celsius Profile (Scientific)**

```json
{
  "profile_metadata": {
    "name": "Precision Thermal Test",
    "version": "1.0",
    "description": "High-precision temperature control",
    "temperature_unit": "celsius",
    "tags": ["scientific", "precision", "testing"]
  },
  "ramp_phase": {
    "duration_days": 0,
    "duration_hours": 12,
    "temperature": {
      "start": 25.0,
      "end": 10.0,
      "unit": "°C"
    },
    "dewpoint": {
      "start": 20.0,
      "end": 8.0,
      "unit": "°C"
    },
    "circulation_fan": {
      "speed_start": 100,
      "speed_end": 50,
      "modulation_start": 60,
      "modulation_end": 60
    }
  },
  "plateau_phase": {
    "duration_days": 1,
    "temperature": {"target": 10.0},
    "dewpoint": {"target": 8.0},
    "circulation_fan": {"speed": 50, "modulation": 60}
  },
  "hold_phase": {
    "temperature": {"target": 15.0},
    "dewpoint": {"target": 12.0},
    "circulation_fan": {"speed": 30, "modulation": 45}
  },
  "advanced_settings": {
    "tec_control": {"max_current": 4.0},
    "heatsink_control": {
      "cold_setpoint": 7.0,
      "bias": 1.8,
      "wetting_force": 1.5
    },
    "safety_limits": {
      "frost_limit": 1.0,
      "allow_below_freezing": false
    }
  }
}
```

**When loaded:**
- Switch → OFF (Celsius)
- Display → 25°C, 10°C, 8°C, etc.
- PIDs receive → 25°C, 10°C, 8°C (direct, no conversion)

---

## 🎯 Design Philosophy

### **Why Profile Controls Switch?**

**Alternative 1 (Rejected):** User switch controls, profile converts
- ❌ Profile author loses control
- ❌ Can't ensure "load exactly as designed"
- ❌ Sharing profiles becomes ambiguous

**Alternative 2 (Rejected):** Profile and user independent
- ❌ Confusing when they don't match
- ❌ Requires complex conversion logic
- ❌ Double conversion bugs

**Current Design (Chosen):** Profile controls, user can override
- ✅ Profile loads exactly as author intended
- ✅ Clear intent in JSON metadata
- ✅ User can flip switch anytime
- ✅ Export preserves current preference
- ✅ Simple, predictable behavior

---

## 📖 Real-World Scenarios

### **Scenario 1: US Cheesemaker Shares Recipe**

**Creator (US):**
```
1. Creates "Sharp Cheddar 90-Day" profile
2. Works in Fahrenheit (switch ON)
3. Sets temps: 55°F → 52°F → 50°F
4. Exports → JSON in Fahrenheit
5. Shares on forum
```

**User (Europe):**
```
1. Downloads JSON
2. Loads profile
3. Switch auto-set to Fahrenheit
4. Sees: 55°F → 52°F → 50°F (as creator intended)
5. Optionally flips to Celsius: 12.8°C → 11.1°C → 10.0°C
6. Can use either way
```

**Result:** Recipe preserved exactly, but flexible for user preference.

---

### **Scenario 2: Lab Creates Standard Protocol**

**Lab (SI Units Required):**
```
1. Creates "Material Aging Protocol v2.0"
2. Works in Celsius (switch OFF)
3. Sets precise temps: 25.0°C → 10.5°C
4. Exports → JSON in Celsius
5. Publishes as standard
```

**Compliance Team:**
```
1. Loads standard protocol
2. Switch auto-set to Celsius
3. Sees exact values: 25.0°C → 10.5°C
4. Verifies matches published spec
5. Runs test
```

**Result:** Scientific precision maintained, no unit ambiguity.

---

### **Scenario 3: User Tweaks Existing Profile**

**Starting point:**
```
1. Load "Cheddar Aging" (Fahrenheit)
2. Switch set to Fahrenheit
3. Display: 55°F → 54°F
```

**User prefers Celsius:**
```
1. Flips switch OFF
2. Display: 12.8°C → 12.2°C
3. Thinks: "I want 13°C → 12°C"
4. Adjusts values
5. Exports → "My Cheddar (Celsius).json"
6. Future loads use Celsius
```

**Result:** User creates personal version in preferred units.

---

## 🔮 Future Enhancements

### **Possible Addition: Dual-Unit Display**

Could show both units simultaneously:
```yaml
sensor:
  - platform: template
    name: "Ramp Start (Both Units)"
    lambda: |-
      float temp = id(ramp_start_temp).state;
      if (id(use_fahrenheit).state) {
        float celsius = (temp - 32.0) * 5.0 / 9.0;
        char buf[32];
        snprintf(buf, sizeof(buf), "%.1f°F (%.1f°C)", temp, celsius);
        return std::string(buf);
      } else {
        float fahrenheit = (temp * 9.0 / 5.0) + 32.0;
        char buf[32];
        snprintf(buf, sizeof(buf), "%.1f°C (%.1f°F)", temp, fahrenheit);
        return std::string(buf);
      }
```

Display: `"55.0°F (12.8°C)"` or `"12.8°C (55.0°F)"`

---

### **Possible Addition: Profile Converter Tool**

```yaml
button:
  - platform: template
    name: "Convert Profile Units"
    on_press:
      - lambda: |-
          // Flip all temperature values
          if (id(use_fahrenheit).state) {
            // Currently F, convert to C
            id(ramp_start_temp).publish_state((id(ramp_start_temp).state - 32.0) * 5.0 / 9.0);
            // ... convert all temps
            id(use_fahrenheit).turn_off();
          } else {
            // Currently C, convert to F
            id(ramp_start_temp).publish_state((id(ramp_start_temp).state * 9.0 / 5.0) + 32.0);
            // ... convert all temps
            id(use_fahrenheit).turn_on();
          }
```

---

## ✨ Summary

**The Golden Rules:**
1. Profile declares its unit in metadata
2. Loading sets switch to match profile
3. Display always matches switch state
4. User can flip switch anytime
5. Export saves in current switch state
6. PIDs always receive Celsius (hardware layer)
7. Advanced settings always Celsius (never change)

**User Experience:**
- ✅ Load profile → See exactly what author intended
- ✅ Flip switch → See your preferred units
- ✅ Export → Save for future use
- ✅ Share → Others get same experience

**Technical Correctness:**
- ✅ No double conversions
- ✅ Hardware always gets Celsius
- ✅ Single conversion point (submit_values)
- ✅ Precision maintained
- ✅ Predictable behavior

**You're ready to test!** 🚀

Load a profile, watch the switch change, flip it back, export, and verify the cycle works perfectly.