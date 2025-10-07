# PhaseShift Temperature Unit Handling - Complete Guide

## 🌡️ The Golden Rules

### **Rule 1: Hardware ALWAYS Uses Celsius**
These components **never** accept Fahrenheit:
- ✅ PID climate controllers (`pid_drybulb`, `pid_dewpoint`, `pid_cold`)
- ✅ Cold heatsink setpoint (`cold_hs_setpoint`)
- ✅ Frost limit (`frost_limit`)
- ✅ All sensor readings (TMP117, Dallas, SHT3x)
- ✅ All internal calculations (dewpoint math, vapor pressure)

**Why?** ESPHome climate components are hard-coded to use Celsius internally.

### **Rule 2: User-Facing Values Can Be Either**
These respect the `use_fahrenheit` switch:
- 🔄 Drybulb target (`drybulb_setpoint`)
- 🔄 Dewpoint target (`dewpoint_setpoint`)
- 🔄 All ramp parameters (start/end temps)
- 🔄 All plateau parameters
- 🔄 All hold parameters
- 🔄 Profile JSON temperature values

### **Rule 3: Conversion Happens at ONE Point**
The `submit_values` button is the **only** place where F→C conversion occurs for PID controllers.

---

## 📊 Data Flow Diagram

```
PROFILE (F or C)
    ↓
convert_temp() function
    ↓
NUMBER ENTITIES (match user's switch)
    ↓
submit_values button
    ↓
F→C conversion if needed
    ↓
drybulb_celcius sensor (ALWAYS °C)
dewpoint_celcius sensor (ALWAYS °C)
    ↓
PID CONTROLLERS (ALWAYS °C)
    ↓
HARDWARE (ALWAYS °C)
```

---

## 🔧 Implementation Details

### **1. Profile Loading**

When a profile loads, it checks:
```cpp
// Read profile's declared unit
std::string unit = meta["temperature_unit"] | "fahrenheit";
bool profile_is_fahrenheit = (unit == "fahrenheit");

// Check user's preference
bool user_wants_fahrenheit = id(use_fahrenheit).state;

// Convert if they don't match
if (profile_is_fahrenheit != user_wants_fahrenheit) {
    // Do conversion
}
```

**Example:**
- Profile says: `"temperature_unit": "fahrenheit"`, value = 68°F
- User switch: OFF (wants Celsius)
- Conversion: (68 - 32) × 5/9 = 20°C
- Result: Number entity shows "20.0°C"

### **2. Submit Values (The Conversion Gateway)**

```yaml
button:
  - platform: template
    id: submit_values
    on_press:
      - lambda: |-
          float drybulb_value = id(drybulb_setpoint).state;
          
          // Only convert if user is in Fahrenheit mode
          if (id(use_fahrenheit).state) {
            drybulb_value = (drybulb_value - 32.0) * 5.0 / 9.0;
          }
          
          // Store in Celsius sensor
          id(drybulb_celcius).publish_state(drybulb_value);
          
      // Send Celsius value to PID
      - climate.control:
          id: pid_drybulb
          target_temperature: !lambda 'return id(drybulb_celcius).state;'
```

**Key Point:** The PIDs always receive Celsius, regardless of user preference.

### **3. Advanced Settings (Always Celsius)**

```cpp
// These are NEVER converted - always Celsius!
if (adv.containsKey("heatsink_control")) {
    auto hs = adv["heatsink_control"];
    // Direct assignment - no conversion
    id(cold_hs_setpoint).publish_state(hs["cold_setpoint"] | 11.11);
    id(frost_limit).publish_state(hs["frost_limit"] | 1.0);
}
```

**Why?** These control hardware directly and must be precise. Using Celsius avoids rounding errors.

---

## ✅ Verification Checklist

### **Test Scenario 1: User in Fahrenheit Mode**

1. User has `use_fahrenheit` switch ON
2. Load Default profile (fahrenheit)
3. Profile value: 68°F
4. **Expected:**
   - `drybulb_setpoint` shows: 68.0°F ✅
   - User clicks submit
   - `drybulb_celcius` receives: 20.0°C ✅
   - PID receives: 20.0°C ✅

### **Test Scenario 2: User in Celsius Mode**

1. User has `use_fahrenheit` switch OFF
2. Load Default profile (fahrenheit)
3. Profile value: 68°F
4. **Expected:**
   - Profile converts: 68°F → 20°C
   - `drybulb_setpoint` shows: 20.0°C ✅
   - User clicks submit
   - No conversion needed (already C)
   - `drybulb_celcius` receives: 20.0°C ✅
   - PID receives: 20.0°C ✅

### **Test Scenario 3: Celsius Profile, Fahrenheit User**

1. User has `use_fahrenheit` switch ON
2. Load hypothetical Celsius profile
3. Profile value: 20°C
4. **Expected:**
   - Profile converts: 20°C → 68°F
   - `drybulb_setpoint` shows: 68.0°F ✅
   - User clicks submit
   - Converts back: 68°F → 20°C
   - `drybulb_celcius` receives: 20.0°C ✅
   - PID receives: 20.0°C ✅

### **Test Scenario 4: Cold Setpoint (Always Celsius)**

1. User in Fahrenheit mode
2. Load profile with `cold_setpoint: 11.11`
3. **Expected:**
   - `cold_hs_setpoint` shows: 11.11°C ✅ (no conversion!)
   - This number entity has `unit: "°C"` hardcoded
   - PID receives: 11.11°C ✅

---

## 🐛 Common Mistakes to Avoid

### **❌ WRONG: Converting cold_setpoint**
```cpp
// DON'T DO THIS!
if (user_wants_fahrenheit) {
    float cold_f = (cold_c * 9.0 / 5.0) + 32.0;
    id(cold_hs_setpoint).publish_state(cold_f);  // BROKEN!
}
```

**Why wrong?** The cold heatsink PID expects Celsius. Converting it would send wrong values to hardware.

### **❌ WRONG: Converting in multiple places**
```cpp
// DON'T DO THIS!
// Profile loader converts F→C
// Then submit_values converts F→C again
// Result: Double conversion!
```

**Why wrong?** You'd get 20°C → 68°F → 20°C → -6.67°C (disaster!)

### **❌ WRONG: Forgetting to convert profile values**
```cpp
// DON'T DO THIS!
id(ramp_start_temp).publish_state(temp["start"]);  // No conversion!
```

**Why wrong?** If profile is F but user wants C, you'd show 68°C instead of 20°C.

### **✅ CORRECT: Single conversion point**
```cpp
// Profile loader: Convert to match user preference
id(ramp_start_temp).publish_state(convert_temp(68.0));

// submit_values: Convert user units to Celsius
if (id(use_fahrenheit).state) {
    value = (value - 32.0) * 5.0 / 9.0;
}
id(drybulb_celcius).publish_state(value);

// PID: Receives Celsius
climate.control(id(pid_drybulb), target=drybulb_celcius);
```

---

## 📋 Entity Unit Reference

| Entity | Unit | Converts? | Notes |
|--------|------|-----------|-------|
| `drybulb_setpoint` | F or C | ✅ Yes | Matches user switch |
| `dewpoint_setpoint` | F or C | ✅ Yes | Matches user switch |
| `drybulb_celcius` | °C | ❌ No | Internal - always C |
| `dewpoint_celcius` | °C | ❌ No | Internal - always C |
| `cold_hs_setpoint` | °C | ❌ No | Hardware - always C |
| `frost_limit` | °C | ❌ No | Hardware - always C |
| `ramp_start_temp` | F or C | ✅ Yes | Matches user switch |
| `ramp_end_temp` | F or C | ✅ Yes | Matches user switch |
| `plateau_temp` | F or C | ✅ Yes | Matches user switch |
| `hold_temp` | F or C | ✅ Yes | Matches user switch |
| `chamber_temp` | °C | ❌ No | Sensor reading |
| `cold_temp` | °C | ❌ No | Sensor reading |
| `hot_temp` | °C | ❌ No | Sensor reading |

---

## 🧪 Testing Temperature Conversion

### **Quick Test Script**

```yaml
# Add to test harness for verification
button:
  - platform: template
    name: "Test Temperature Conversion"
    on_press:
      - lambda: |-
          ESP_LOGI("test", "=== TEMPERATURE CONVERSION TEST ===");
          
          // Test 1: Fahrenheit mode
          ESP_LOGI("test", "TEST 1: User in Fahrenheit");
          ESP_LOGI("test", "  drybulb_setpoint: %.1f°F", id(drybulb_setpoint).state);
          float converted = (id(drybulb_setpoint).state - 32.0) * 5.0 / 9.0;
          ESP_LOGI("test", "  After F→C: %.1f°C", converted);
          ESP_LOGI("test", "  drybulb_celcius: %.1f°C", id(drybulb_celcius).state);
          ESP_LOGI("test", "  Match? %s", (abs(converted - id(drybulb_celcius).state) < 0.1) ? "✅ YES" : "❌ NO");
          
          // Test 2: Check cold setpoint
          ESP_LOGI("test", "TEST 2: Cold Heatsink");
          ESP_LOGI("test", "  cold_hs_setpoint: %.2f°C", id(cold_hs_setpoint).state);
          ESP_LOGI("test", "  Should be: 11.11°C");
          ESP_LOGI("test", "  Match? %s", (abs(id(cold_hs_setpoint).state - 11.11) < 0.01) ? "✅ YES" : "❌ NO");
          
          ESP_LOGI("test", "================================");
```

### **Expected Output (Fahrenheit mode, Default profile)**

```
[test] === TEMPERATURE CONVERSION TEST ===
[test] TEST 1: User in Fahrenheit
[test]   drybulb_setpoint: 68.0°F
[test]   After F→C: 20.0°C
[test]   drybulb_celcius: 20.0°C
[test]   Match? ✅ YES
[test] TEST 2: Cold Heatsink
[test]   cold_hs_setpoint: 11.11°C
[test]   Should be: 11.11°C
[test]   Match? ✅ YES
[test] ================================
```

---

## 💡 Key Takeaways

1. **Two-layer system:**
   - User layer (F or C based on switch)
   - Hardware layer (always C)

2. **Single conversion point:**
   - Profile → user units (in loader)
   - User units → Celsius (in submit_values)

3. **Never convert twice:**
   - Each value converted maximum once per path

4. **Hardware exceptions:**
   - `cold_hs_setpoint`, `frost_limit` always Celsius
   - Never displayed in Fahrenheit

5. **Profile flexibility:**
   - Profiles can be F or C
   - System converts automatically
   - User sees their preferred units

---

## 🎯 Summary

**The system is designed so:**
- ✅ Users work in their preferred units
- ✅ Profiles can be authored in either F or C
- ✅ Hardware always receives correct Celsius values
- ✅ No double-conversion bugs
- ✅ Clear separation of concerns

**Critical files to check:**
1. `profiles.yaml` - Has `convert_temp()` function
2. `controls.yaml` - Has runtime F/C in `submit_values`
3. `phaseshift.yaml` - No `use_fahrenheit` substitution

**If you see weird temperatures:**
1. Check logs for conversion messages
2. Verify `use_fahrenheit` switch state
3. Check profile `temperature_unit` metadata
4. Run the test button script above

Your implementation is **correct** as long as you use the updated `parse_and_apply_profile` script with the `convert_temp()` lambda! 🎉