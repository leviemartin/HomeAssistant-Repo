# Color Consistency Guide - Living Room ↔ Adjacent Zones

**How MAXIMUM color consistency is achieved between blueprints**

This guide explains the technical implementation of color consistency between the Intelligent Living Room blueprint and the Adjacent Zone Motion blueprint. These blueprints share IDENTICAL color logic to ensure seamless transitions when walking between rooms.

**Version**: 1.0.0
**Last Updated**: 2025-11-18

---

## Table of Contents

1. [Overview](#overview)
2. [Side-by-Side Code Comparison](#side-by-side-code-comparison)
3. [Circadian Color Temperature](#circadian-color-temperature)
4. [Dynamic Brightness](#dynamic-brightness)
5. [Lux Hysteresis System](#lux-hysteresis-system)
6. [Staged Turn-Off Warnings](#staged-turn-off-warnings)
7. [Update Timing and Transitions](#update-timing-and-transitions)
8. [Testing Color Consistency](#testing-color-consistency)
9. [Troubleshooting Mismatches](#troubleshooting-mismatches)

---

## Overview

### What is Color Consistency?

Color consistency means that when you walk from your living room to your hallway, the light color temperature remains IDENTICAL between the two spaces. This creates a seamless, professional lighting experience with no jarring color shifts.

### Why is This Important?

Without color consistency:
- Living room at 3500K (warm) → Hallway at 4500K (cool) = **Jarring shift**
- Living room at 80% brightness → Hallway at 100% = **Uncomfortable transition**
- Living room turns off at 150 lux → Hallway turns off at 200 lux = **Inconsistent behavior**

With color consistency:
- Living room at 3500K → Hallway at 3500K = **Seamless transition**
- Living room at 70% → Hallway at 70% = **Comfortable matching**
- Both turn off at 150 lux = **Predictable behavior**

### How is Consistency Achieved?

The Adjacent Zone Motion blueprint uses EXACT COPIES of critical formulas from the Living Room blueprint:

1. **Circadian Sun Elevation Formula**: Copied line-by-line from living room
2. **Dynamic Brightness Curve**: Identical lux-to-brightness mapping
3. **Lux Hysteresis Thresholds**: Hardcoded to match living room
4. **Warning Color Temperature**: Same 1800K warm during staged warnings
5. **Update Frequency**: Same 60-second circadian updates
6. **Transition Duration**: Same 4-second color transitions

**Result**: PERFECT color matching across all zones!

---

## Side-by-Side Code Comparison

### 1. Circadian Color Temperature Formula

**Living Room Blueprint** (lines 718-750):
```yaml
color_temp_kelvin: >
  {% if circadian_rhythm %}
    {% set elevation = sun_elevation | float %}

    {# Deep night (sun well below horizon) #}
    {% if elevation < -6 %}
      1800

    {# Civil twilight (sun just below horizon) - gentle warm transition #}
    {% elif elevation >= -6 and elevation < 0 %}
      {{ (1800 + (elevation + 6) * 200) | int }}  {# 1800K to 3000K #}

    {# Low sun (sunrise/sunset) - warming up or cooling down #}
    {% elif elevation >= 0 and elevation < 10 %}
      {{ (3000 + (elevation * 50)) | int }}  {# 3000K to 3500K #}

    {# Rising/falling sun - energizing transition #}
    {% elif elevation >= 10 and elevation < 30 %}
      {{ (3500 + (elevation - 10) * 75) | int }}  {# 3500K to 5000K #}

    {# High sun - peak alertness #}
    {% elif elevation >= 30 and elevation < 50 %}
      {{ (5000 + (elevation - 30) * 25) | int }}  {# 5000K to 5500K #}

    {# Very high sun (summer midday) - maximum cool daylight #}
    {% else %}
      5500
    {% endif %}
  {% else %}
    3000  {# Default warm white if circadian disabled #}
  {% endif %}
```

**Adjacent Zone Blueprint**:
```yaml
color_temp_kelvin: >
  {% if circadian_rhythm %}
    {% set elevation = sun_elevation | float %}

    {# Deep night (sun well below horizon) #}
    {% if elevation < -6 %}
      1800

    {# Civil twilight (sun just below horizon) - gentle warm transition #}
    {% elif elevation >= -6 and elevation < 0 %}
      {{ (1800 + (elevation + 6) * 200) | int }}  {# 1800K to 3000K #}

    {# Low sun (sunrise/sunset) - warming up or cooling down #}
    {% elif elevation >= 0 and elevation < 10 %}
      {{ (3000 + (elevation * 50)) | int }}  {# 3000K to 3500K #}

    {# Rising/falling sun - energizing transition #}
    {% elif elevation >= 10 and elevation < 30 %}
      {{ (3500 + (elevation - 10) * 75) | int }}  {# 3500K to 5000K #}

    {# High sun - peak alertness #}
    {% elif elevation >= 30 and elevation < 50 %}
      {{ (5000 + (elevation - 30) * 25) | int }}  {# 5000K to 5500K #}

    {# Very high sun (summer midday) - maximum cool daylight #}
    {% else %}
      5500
    {% endif %}
  {% else %}
    3000  {# Default warm white if circadian disabled #}
  {% endif %}
```

**Comparison**: ✅ **100% IDENTICAL** - Character-for-character match

---

### 2. Kelvin to Mireds Conversion

**Living Room Blueprint** (line 755):
```yaml
color_temp_mireds: "{{ (1000000 / (color_temp_kelvin | int)) | int }}"
```

**Adjacent Zone Blueprint**:
```yaml
color_temp_mireds: "{{ (1000000 / (color_temp_kelvin | int)) | int }}"
```

**Comparison**: ✅ **100% IDENTICAL**

---

### 3. Warning Color Temperature (Staged Turn-Off)

**Living Room Blueprint** (line 759):
```yaml
warning_color_temp_mireds: "{{ (1000000 / 1800) | int }}"
```

**Adjacent Zone Blueprint**:
```yaml
warning_color_temp_mireds: "{{ (1000000 / 1800) | int }}"
```

**Comparison**: ✅ **100% IDENTICAL** - Both use 1800K (556 mireds)

---

### 4. Dynamic Brightness Curve

**Living Room Blueprint** (lines 689-716):
```yaml
calculated_brightness: >
  {% if dynamic_brightness %}
    {% set lux = current_lux | float %}
    {% if lux <= 50 %}
      {{ brightness_at_50_lux }}
    {% elif lux <= 75 %}
      {# Linear interpolation between 50 and 75 lux #}
      {% set range_pct = (lux - 50) / 25 %}
      {{ (brightness_at_50_lux - (brightness_at_50_lux - brightness_at_75_lux) * range_pct) | int }}
    {% elif lux <= 100 %}
      {# Linear interpolation between 75 and 100 lux #}
      {% set range_pct = (lux - 75) / 25 %}
      {{ (brightness_at_75_lux - (brightness_at_75_lux - brightness_at_100_lux) * range_pct) | int }}
    {% elif lux <= 125 %}
      {# Linear interpolation between 100 and 125 lux #}
      {% set range_pct = (lux - 100) / 25 %}
      {{ (brightness_at_100_lux - (brightness_at_100_lux - brightness_at_125_lux) * range_pct) | int }}
    {% elif lux <= 150 %}
      {# Linear interpolation between 125 and 150 lux #}
      {% set range_pct = (lux - 125) / 25 %}
      {{ (brightness_at_125_lux - (brightness_at_125_lux - brightness_at_150_lux) * range_pct) | int }}
    {% else %}
      {# Above 150 lux - use minimum brightness #}
      {{ brightness_at_150_lux }}
    {% endif %}
  {% else %}
    100  {# Default full brightness if dynamic brightness disabled #}
  {% endif %}
```

**Adjacent Zone Blueprint** (HARDCODED values):
```yaml
calculated_brightness: >
  {% set lux = current_lux | float %}
  {% if lux <= 50 %}
    100
  {% elif lux <= 75 %}
    {# Linear interpolation between 50 and 75 lux #}
    {% set range_pct = (lux - 50) / 25 %}
    {{ (100 - (100 - 85) * range_pct) | int }}
  {% elif lux <= 100 %}
    {# Linear interpolation between 75 and 100 lux #}
    {% set range_pct = (lux - 75) / 25 %}
    {{ (85 - (85 - 70) * range_pct) | int }}
  {% elif lux <= 125 %}
    {# Linear interpolation between 100 and 125 lux #}
    {% set range_pct = (lux - 100) / 25 %}
    {{ (70 - (70 - 55) * range_pct) | int }}
  {% elif lux <= 150 %}
    {# Linear interpolation between 125 and 150 lux #}
    {% set range_pct = (lux - 125) / 25 %}
    {{ (55 - (55 - 40) * range_pct) | int }}
  {% else %}
    {# Above 150 lux - use minimum brightness #}
    40
  {% endif %}
```

**Comparison**: ✅ **100% IDENTICAL** (Adjacent Zone hardcodes the default values)

**Brightness Breakpoints**:
| Lux Level | Living Room (default) | Adjacent Zone (hardcoded) | Match? |
|-----------|----------------------|---------------------------|--------|
| ≤50 lux | 100% | 100% | ✅ IDENTICAL |
| 75 lux | 85% | 85% | ✅ IDENTICAL |
| 100 lux | 70% | 70% | ✅ IDENTICAL |
| 125 lux | 55% | 55% | ✅ IDENTICAL |
| 150 lux | 40% | 40% | ✅ IDENTICAL |

---

### 5. Lux Hysteresis Thresholds

**Living Room Blueprint** (default values from inputs):
```yaml
lux_on_threshold: !input lux_turn_on_threshold   # Default: 80
lux_off_threshold: !input lux_turn_off_threshold # Default: 150
lux_on_debounce_sec: !input lux_turn_on_debounce # Default: 120
lux_off_debounce_sec: !input lux_turn_off_debounce # Default: 300
sunrise_sunset_debounce_sec: !input sunrise_sunset_extended_debounce # Default: 600
```

**Adjacent Zone Blueprint** (HARDCODED):
```yaml
lux_on_threshold: 80          # Turn ON when lux drops below 80
lux_off_threshold: 150        # Turn OFF when lux rises above 150
lux_on_debounce_sec: 120      # 2 minutes ON debounce
lux_off_debounce_sec: 300     # 5 minutes OFF debounce
sunrise_sunset_debounce_sec: 600  # 10 minutes extended debounce
```

**Comparison**: ✅ **100% IDENTICAL** (Adjacent Zone hardcodes the living room defaults)

---

### 6. Staged Turn-Off Brightness Levels

**Living Room Blueprint** (default values from inputs):
```yaml
stage_1_brightness_pct: !input stage_1_brightness # Default: 40
stage_2_brightness_pct: !input stage_2_brightness # Default: 20
```

**Adjacent Zone Blueprint** (HARDCODED):
```yaml
stage_1_brightness_pct: 40    # First warning: 40% brightness
stage_2_brightness_pct: 20    # Second warning: 20% brightness
```

**Comparison**: ✅ **100% IDENTICAL**

---

### 7. Circadian Update Timing

**Living Room Blueprint** (default values from inputs):
```yaml
circadian_update_sec: !input circadian_update_interval # Default: 60
circadian_transition_sec: !input circadian_transition_duration # Default: 4
```

**Adjacent Zone Blueprint** (from inputs, default to same):
```yaml
circadian_update_sec: !input circadian_update_interval # Default: 60
circadian_transition_sec: !input circadian_transition_duration # Default: 4
```

**Comparison**: ✅ **100% IDENTICAL** (same defaults)

---

## Circadian Color Temperature

### Sun Elevation to Kelvin Mapping

Both blueprints use this EXACT elevation-to-Kelvin conversion:

| Sun Elevation | Kelvin | Mireds | Description |
|---------------|--------|--------|-------------|
| < -6° | 1800K | 556 | Deep night (amber warmth) |
| -6° to -5° | 2000K | 500 | Civil twilight start |
| -4° to -2° | 2400K | 417 | Civil twilight middle |
| -1° to 0° | 2800K | 357 | Civil twilight end |
| 0° to 5° | 3000K → 3250K | 333 → 308 | Sunrise/sunset |
| 5° to 10° | 3250K → 3500K | 308 → 286 | Low sun |
| 10° to 20° | 3500K → 4250K | 286 → 235 | Rising sun |
| 20° to 30° | 4250K → 5000K | 235 → 200 | Mid-morning/afternoon |
| 30° to 40° | 5000K → 5250K | 200 → 190 | High sun |
| 40° to 50° | 5250K → 5500K | 190 → 182 | Very high sun |
| > 50° | 5500K | 182 | Peak daylight (summer midday) |

### Formula Breakdown

**Elevation < -6° (Deep Night)**:
```
Kelvin = 1800
```

**Elevation -6° to 0° (Civil Twilight)**:
```
Kelvin = 1800 + (elevation + 6) * 200
Example: elevation = -3°
  → Kelvin = 1800 + (-3 + 6) * 200 = 1800 + 600 = 2400K
```

**Elevation 0° to 10° (Sunrise/Sunset)**:
```
Kelvin = 3000 + (elevation * 50)
Example: elevation = 5°
  → Kelvin = 3000 + (5 * 50) = 3000 + 250 = 3250K
```

**Elevation 10° to 30° (Rising/Falling Sun)**:
```
Kelvin = 3500 + (elevation - 10) * 75
Example: elevation = 20°
  → Kelvin = 3500 + (20 - 10) * 75 = 3500 + 750 = 4250K
```

**Elevation 30° to 50° (High Sun)**:
```
Kelvin = 5000 + (elevation - 30) * 25
Example: elevation = 40°
  → Kelvin = 5000 + (40 - 30) * 25 = 5000 + 250 = 5250K
```

**Elevation > 50° (Peak Daylight)**:
```
Kelvin = 5500
```

### Why This Works for Consistency

1. **Both blueprints read from same source**: `state_attr('sun.sun', 'elevation')`
2. **Both use identical formula**: Character-for-character match
3. **Both update simultaneously**: Every 60 seconds (default)
4. **Both convert identically**: `mireds = 1000000 / kelvin`

**Result**: At any given moment, living room and hallway have EXACT same color!

---

## Dynamic Brightness

### Lux to Brightness Mapping

Both blueprints use this EXACT lux-to-brightness conversion:

| Current Lux | Brightness | Reasoning |
|-------------|------------|-----------|
| ≤50 lux | 100% | Very dark - need maximum brightness |
| 60 lux | ~94% | Linear interpolation |
| 75 lux | 85% | Getting brighter - reduce slightly |
| 87.5 lux | ~77% | Linear interpolation |
| 100 lux | 70% | Mid-range - comfortable brightness |
| 112.5 lux | ~62% | Linear interpolation |
| 125 lux | 55% | Getting bright - further reduction |
| 137.5 lux | ~47% | Linear interpolation |
| 150 lux | 40% | At OFF threshold - minimum brightness |
| >150 lux | 40% (then OFF) | Too bright - lights will turn off after debounce |

### Linear Interpolation Example

**Scenario**: Current lux = 87.5 (between 75 and 100)

**Living Room Blueprint**:
```yaml
{% set range_pct = (87.5 - 75) / 25 %}  # = 0.5
{{ (85 - (85 - 70) * 0.5) | int }}       # = 85 - 7.5 = 77.5 → 77%
```

**Adjacent Zone Blueprint**:
```yaml
{% set range_pct = (87.5 - 75) / 25 %}  # = 0.5
{{ (85 - (85 - 70) * 0.5) | int }}       # = 85 - 7.5 = 77.5 → 77%
```

**Result**: ✅ **IDENTICAL** - Both calculate 77% brightness

### Why This Works for Consistency

1. **Same breakpoints**: 50/75/100/125/150 lux
2. **Same percentages**: 100/85/70/55/40%
3. **Same interpolation**: Linear between points
4. **Same source**: Both read from lux sensor entity

**Result**: At any lux level, both rooms have same brightness!

---

## Lux Hysteresis System

### Hysteresis Band Visualization

```
                    Lux Level
    0      80      100     150     200     300
    |------|-------|-------|-------|-------|------>

    LIGHTS OFF STATE:
    [------][ON OK ][HYST. BAND   ][STAYS OFF    ]
              ↑                     ↑
              Turn ON < 80 lux      Won't turn on > 80 lux

    LIGHTS ON STATE:
    [STAYS ON      ][HYST. BAND   ][OFF OK]
                     ↑               ↑
                     Won't turn off  Turn OFF > 150 lux
                     < 150 lux
```

### State Transitions

**Living Room Blueprint**:
```yaml
# Turn ON conditions:
- Current state: Lights OFF
- Lux drops below 80 lux
- Wait 2 minutes (debounce)
- If still < 80 lux → Turn ON

# Turn OFF conditions:
- Current state: Lights ON
- Lux rises above 150 lux
- Wait 5 minutes (debounce, 10 min during dawn/dusk)
- If still > 150 lux → Turn OFF
```

**Adjacent Zone Blueprint**:
```yaml
# Turn ON conditions:
- Current state: Lights OFF
- Lux drops below 80 lux
- Wait 2 minutes (debounce)
- If still < 80 lux → Turn ON

# Turn OFF conditions:
- Current state: Lights ON
- Lux rises above 150 lux
- Wait 5 minutes (debounce, 10 min during dawn/dusk)
- If still > 150 lux → Turn OFF
```

**Comparison**: ✅ **100% IDENTICAL**

### Hysteresis Gap

The 70-lux gap (150 - 80 = 70) prevents flicker:

**Without Hysteresis** (single threshold at 100 lux):
```
Lux: 98 → 102 → 98 → 102 → 98 → 102
Lights: ON → OFF → ON → OFF → ON → OFF  ❌ FLICKER!
```

**With Hysteresis** (80/150 thresholds):
```
Lux: 98 → 102 → 98 → 102 → 98 → 102
Lights: ON → ON → ON → ON → ON → ON  ✅ STABLE!
```

---

## Staged Turn-Off Warnings

### Warning Sequence

Both blueprints use IDENTICAL staged turn-off behavior:

**Stage 1 (First Warning)**:
- Brightness: 40% (60% reduction)
- Color: 1800K warm (556 mireds)
- Transition: 3 seconds
- Purpose: Noticeable warning, still functional

**Stage 2 (Second Warning)**:
- Brightness: 20% (80% reduction)
- Color: 1800K warm (556 mireds)
- Transition: 3 seconds
- Purpose: Strong warning, lights turning off soon

**Stage 3 (Turn-Off)**:
- Brightness: 0% (complete OFF)
- Transition: 3 seconds
- Purpose: Gentle fade to complete darkness

### Timing Differences

**Living Room Blueprint** (default):
- Stage 1: 30 minutes after presence clears
- Stage 2: 40 minutes total
- Stage 3: 45 minutes total

**Adjacent Zone Blueprint** (Quick preset):
- Stage 1: 5 minutes after motion clears
- Stage 2: 10 minutes total
- Stage 3: 15 minutes total

**Color/Brightness**: ✅ **IDENTICAL** (only timing differs)

### Why 1800K for Warnings?

1. **Warm color is calming**: Less jarring than cool color
2. **Distinct from circadian**: Clearly indicates warning state
3. **Consistent across zones**: Same warning color everywhere
4. **Philips Hue compatible**: 1800K is warmest most bulbs support

---

## Update Timing and Transitions

### Circadian Update Cycle

**Living Room Blueprint**:
```yaml
- platform: time_pattern
  minutes: "/1"  # Every minute
  id: circadian_update

# Then in action:
circadian_update_sec: 60  # Actually update every 60 seconds
```

**Adjacent Zone Blueprint**:
```yaml
- platform: time_pattern
  minutes: "/1"  # Every minute
  id: circadian_update

# Then in action:
circadian_update_sec: 60  # Actually update every 60 seconds
```

**Comparison**: ✅ **IDENTICAL**

### Transition Duration

**Living Room Blueprint**:
```yaml
transition: "{{ circadian_transition_sec }}"  # Default: 4 seconds
```

**Adjacent Zone Blueprint**:
```yaml
transition: "{{ circadian_transition_sec }}"  # Default: 4 seconds
```

**Comparison**: ✅ **IDENTICAL**

### Why 60-Second Updates?

1. **Responsive enough**: Sun elevation changes slowly
2. **Not excessive**: Avoids overwhelming automation engine
3. **Battery-friendly**: For battery-powered lights
4. **Balanced**: Good compromise between accuracy and efficiency

### Why 4-Second Transitions?

1. **Smooth but not sluggish**: Noticeable but not slow
2. **Philips Hue optimized**: Hue bulbs handle 4-sec well
3. **Eye-friendly**: Gradual enough to avoid eye strain
4. **Energy-efficient**: Shorter transitions use less power

---

## Testing Color Consistency

### Test 1: Visual Comparison

**Setup**:
1. Stand between living room and hallway
2. Look at both sets of lights simultaneously
3. Note any color difference

**Expected Result**:
- ✅ PASS: Colors appear identical
- ❌ FAIL: Colors noticeably different

**If FAIL**:
- Check both automations have circadian enabled
- Verify both use same update frequency (60 sec)
- Confirm both use same transition duration (4 sec)

### Test 2: Developer Tools Check

**Setup**:
1. Open **Developer Tools** → **States**
2. Find living room light: `light.living_room_ceiling`
3. Find hallway light: `light.hallway_ceiling`
4. Compare `color_temp` attribute

**Expected Result**:
- ✅ PASS: color_temp values match (within ±5 mireds)
- ❌ FAIL: color_temp differs by >5 mireds

**Example**:
```
light.living_room_ceiling:
  color_temp: 250  # 4000K

light.hallway_ceiling:
  color_temp: 250  # 4000K (or 245-255)
```

### Test 3: Sunrise/Sunset Transition

**Setup**:
1. Wait for sunrise or sunset
2. Watch both rooms during transition
3. Note if colors change together

**Expected Result**:
- ✅ PASS: Both rooms transition together (within 60 sec)
- ❌ FAIL: One room transitions, other doesn't

**Timing**:
- Updates every 60 seconds
- May be offset by up to 60 seconds
- After 60 seconds, should be in sync

### Test 4: Brightness at Same Lux

**Setup**:
1. Ensure both rooms have similar lux levels
2. Check brightness_pct attribute
3. Compare values

**Expected Result**:
- ✅ PASS: brightness_pct values match (within ±5%)
- ❌ FAIL: brightness differs significantly

**Note**: This assumes lux sensors are similarly calibrated

### Test 5: Staged Warning Color

**Setup**:
1. Clear presence/motion from living room
2. Clear motion from hallway
3. Wait for Stage 1 warning (both rooms)
4. Compare color_temp

**Expected Result**:
- ✅ PASS: Both show 556 mireds (1800K)
- ❌ FAIL: Different warning colors

---

## Troubleshooting Mismatches

### Problem: Colors Don't Match Visually

**Possible Causes**:
1. One automation has circadian disabled
2. Different update frequencies
3. Different transition durations
4. Bulb calibration differences

**Solutions**:

**Check Circadian Settings**:
```
Living Room automation:
- Enable Circadian Rhythm: ON ✅

Adjacent Zone automation:
- Enable Circadian Rhythm: ON ✅
```

**Check Update Frequency**:
```
Living Room automation:
- Circadian Update Frequency: 60 seconds ✅

Adjacent Zone automation:
- Circadian Update Frequency: 60 seconds ✅
```

**Check Transition Duration**:
```
Living Room automation:
- Circadian Transition Duration: 4 seconds ✅

Adjacent Zone automation:
- Circadian Transition Duration: 4 seconds ✅
```

**Check Bulb Types**:
- Philips Hue bulbs have best color accuracy
- IKEA Tradfri bulbs may have slight color cast
- Mixing brands can cause visual differences
- All bulbs should be same model for best match

### Problem: color_temp Values Differ in Developer Tools

**Possible Causes**:
1. Automations updating at different times (offset)
2. One automation not running
3. Formula modified in one blueprint

**Solutions**:

**Check Update Offset**:
- Automations update every 60 seconds
- May be offset by up to 60 seconds
- Wait 60 seconds and check again
- Values should match after sync

**Check Automation Status**:
```
Settings → Automations:
- Living Room - Intelligent Lighting: ✅ ENABLED
- Hallway - Motion Circadian: ✅ ENABLED
```

**Check Trace/Logs**:
- Settings → Automations → Select automation → ⋮ → Traces
- Find recent "circadian_update" trigger
- Check if color_temp_mireds calculated correctly

### Problem: Brightness Doesn't Match

**Possible Causes**:
1. Different lux sensor readings
2. Lux sensors in different locations
3. One sensor behind glass (affects readings)
4. Dynamic brightness disabled in one automation

**Solutions**:

**Check Lux Sensor Placement**:
- Both sensors should measure natural light
- Place near windows but not in direct sun
- Not behind glass (affects readings)
- Similar distance from windows

**Check Sensor Readings**:
```
Developer Tools → States:
- sensor.living_room_illuminance: 95 lux
- sensor.hallway_illuminance: 98 lux  (similar ✅)
```

**Check Dynamic Brightness**:
- Living Room: Dynamic brightness ENABLED ✅
- Adjacent Zone: Always enabled (hardcoded) ✅

**Check Brightness Curve** (Living Room):
```
Brightness at 50 lux: 100% (default ✅)
Brightness at 75 lux: 85% (default ✅)
Brightness at 100 lux: 70% (default ✅)
Brightness at 125 lux: 55% (default ✅)
Brightness at 150 lux: 40% (default ✅)
```

### Problem: Warnings Don't Match

**Possible Causes**:
1. Staged turn-off disabled in one automation
2. Different brightness percentages (Living Room customized)
3. Different warning color (formula modified)

**Solutions**:

**Check Staged Turn-Off**:
```
Living Room automation:
- Enable Staged Turn-Off: ON ✅

Adjacent Zone automation:
- Enable Staged Turn-Off: ON ✅
```

**Check Warning Brightness** (Living Room):
```
Stage 1 Brightness: 40% (default ✅)
Stage 2 Brightness: 20% (default ✅)
```

**Check Warning Color**:
- Both should use 1800K (556 mireds)
- Check during warning stage
- Should be warm amber color

### Problem: One Room Updates, Other Doesn't

**Possible Causes**:
1. One automation disabled
2. One automation in error state
3. Trigger not firing
4. Lights offline

**Solutions**:

**Check Automation Status**:
- Settings → Automations
- Both should show "ENABLED"
- Check last triggered time (should be recent)

**Check Automation Traces**:
- Settings → Automations → Select automation → ⋮ → Traces
- Look for errors (red indicators)
- Check if "circadian_update" trigger firing

**Check Light Status**:
```
Developer Tools → States:
- light.living_room_ceiling: state = "on", available = true ✅
- light.hallway_ceiling: state = "on", available = true ✅
```

**Reload Automations**:
- Developer Tools → YAML → Reload Automations
- Wait 60 seconds for next update cycle

---

## Consistency Checklist

Use this checklist to verify color consistency:

### Blueprint Settings

- [ ] Both automations use latest blueprint versions
- [ ] Adjacent Zone blueprint is `adjacent_zone_motion_circadian_lighting.yaml`
- [ ] Living Room blueprint is `intelligent_living_room_mmwave_lux_aware.yaml`

### Circadian Settings

- [ ] Living Room: **Circadian Rhythm** = ENABLED
- [ ] Adjacent Zone: **Circadian Rhythm** = ENABLED
- [ ] Living Room: **Update Frequency** = 60 seconds
- [ ] Adjacent Zone: **Update Frequency** = 60 seconds
- [ ] Living Room: **Transition Duration** = 4 seconds
- [ ] Adjacent Zone: **Transition Duration** = 4 seconds

### Brightness Settings

- [ ] Living Room: **Dynamic Brightness** = ENABLED
- [ ] Living Room: **Brightness at 50 lux** = 100% (default)
- [ ] Living Room: **Brightness at 75 lux** = 85% (default)
- [ ] Living Room: **Brightness at 100 lux** = 70% (default)
- [ ] Living Room: **Brightness at 125 lux** = 55% (default)
- [ ] Living Room: **Brightness at 150 lux** = 40% (default)
- [ ] Adjacent Zone: Brightness curve is hardcoded (always matches)

### Lux Hysteresis Settings

- [ ] Living Room: **Turn-ON Threshold** = 80 lux (default)
- [ ] Living Room: **Turn-OFF Threshold** = 150 lux (default)
- [ ] Living Room: **Turn-ON Debounce** = 120 sec (default)
- [ ] Living Room: **Turn-OFF Debounce** = 300 sec (default)
- [ ] Living Room: **Sunrise/Sunset Debounce** = 600 sec (default)
- [ ] Adjacent Zone: All lux settings are hardcoded (always match)

### Staged Turn-Off Settings

- [ ] Living Room: **Staged Turn-Off** = ENABLED
- [ ] Adjacent Zone: **Staged Turn-Off** = ENABLED
- [ ] Living Room: **Stage 1 Brightness** = 40% (default)
- [ ] Living Room: **Stage 2 Brightness** = 20% (default)
- [ ] Adjacent Zone: Stage brightness is hardcoded (always matches)

### Hardware

- [ ] All lights support color_temp (tunable white)
- [ ] Preferably all Philips Hue or all IKEA Tradfri (same brand)
- [ ] Lux sensors similarly placed (measure natural light)
- [ ] Lux sensors not behind glass
- [ ] Lux sensors calibrated similarly

### Automation Status

- [ ] Both automations ENABLED
- [ ] Both automations triggering regularly (check traces)
- [ ] No errors in automation traces
- [ ] Lights responding to commands

### Visual Verification

- [ ] Colors appear identical when viewing both rooms
- [ ] Brightness appears similar at same lux levels
- [ ] Warning color is same warm amber (1800K)
- [ ] Transitions happen smoothly (no jarring shifts)

---

## Summary

### What is IDENTICAL Between Blueprints

✅ **Circadian Color Temperature**:
- Sun elevation to Kelvin formula (lines 718-750)
- Kelvin to mireds conversion
- Range: 1800K to 5500K
- Update every 60 seconds
- Transition in 4 seconds

✅ **Dynamic Brightness**:
- Lux to brightness curve (50/75/100/125/150 breakpoints)
- Percentages: 100/85/70/55/40%
- Linear interpolation between points

✅ **Lux Hysteresis**:
- Turn-ON threshold: 80 lux
- Turn-OFF threshold: 150 lux
- Debounce: 2 min ON, 5 min OFF, 10 min sunrise/sunset

✅ **Staged Turn-Off**:
- Warning brightness: 40% / 20%
- Warning color: 1800K (556 mireds)
- Transition: 3 seconds

### What is DIFFERENT Between Blueprints

🔄 **Turn-Off Timing**:
- Living Room: 30/40/45 min (default, configurable)
- Adjacent Zone: Preset-based (3/5/7 min to 10/15/20 min)

🔄 **Trigger Type**:
- Living Room: Continuous presence detection (FP2)
- Adjacent Zone: Motion sensor (binary on/off)

🔄 **Feature Set**:
- Living Room: Scene cycling, approach detection, target count
- Adjacent Zone: Simplified (motion + override only)

🔄 **Configuration**:
- Living Room: User-configurable lux/brightness/timing
- Adjacent Zone: Hardcoded consistency parameters

### Result: PERFECT Color Matching

When you walk from living room to hallway:
1. Same sun elevation reading
2. Same Kelvin calculation
3. Same mireds conversion
4. Same brightness percentage
5. Same update timing

**ZERO color shift** - seamless professional experience!

---

**Questions?** See:
- [Adjacent Zone Quick Start Guide](./ADJACENT_ZONE_QUICK_START.md)
- [Adjacent Zone Setup Guide](./ADJACENT_ZONE_SETUP_GUIDE.md)
- [Living Room Setup Guide](./INTELLIGENT_LIVING_ROOM_SETUP_GUIDE.md)
