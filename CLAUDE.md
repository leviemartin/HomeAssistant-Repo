# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is a Home Assistant blueprint repository containing **5 production blueprints** for intelligent lighting automation. All blueprints are scientifically-backed, optimized for Philips Hue compatibility, and designed to promote better sleep, health, and wellbeing through optimized light control.

## Repository Structure

```
/blueprints/                              # Production blueprint files
  circadian_rhythm_lighting.yaml          # Original fixed-time circadian (v1.0)
  circadian_rhythm_lighting_sun_aware.yaml # Sun-tracking circadian (v2.0)
  intelligent_living_room_mmwave_lux_aware.yaml # mmWave + lux-aware (v1.2)
  adjacent_zone_motion_circadian_lighting.yaml # Adjacent zone motion (v1.1)
  child_night_light.yaml                  # Child-friendly night light (v1.0)

  # Blueprint documentation
  *.md                                    # Quick starts, setup guides, technical docs

/docs/                                    # Legacy documentation (original blueprint)
/examples/                                # Example automation configurations
README.md                                 # Main repository documentation
```

## Blueprint Architecture Overview

### 1. Original Circadian Rhythm Lighting (Fixed-Time)
- **File**: `circadian_rhythm_lighting.yaml`
- **Version**: 1.0 (stable, preserved for backward compatibility)
- **Color Range**: 2000K-5500K across 6 fixed time phases
- **Triggers**: Motion sensors with optional lux sensor
- **Turn-off**: 3-stage considerate shutdown (dim warning → wait → fade)
- **Mode**: `restart` (motion resets timer)

### 2. Sun-Aware Circadian Rhythm Lighting
- **File**: `circadian_rhythm_lighting_sun_aware.yaml`
- **Version**: 2.0
- **Color Range**: 2200K-5500K based on actual sun elevation
- **Special Features**:
  - Real sun position tracking via `sun.sun` entity
  - Bedtime overrides (kids 7pm, parents 10pm) force 2200K
  - Smart time boundaries for Netherlands seasonal extremes
  - 4 operating modes: Hybrid, Full Sun, Conservative, Fixed-Time
- **Critical Formula**:
  ```yaml
  # Sun elevation → Kelvin mapping
  elevation < -6°  → 2200K (deep twilight)
  -6° to 0°        → 2200-3000K (civil twilight)
  0° to 10°        → 3000-3500K (low sun)
  10° to 30°       → 3500-4500K (rising/falling sun)
  30° to 50°       → 4500-5500K (high sun)
  > 50°            → 5500K (maximum alertness)
  ```

### 3. Intelligent Living Room Lighting (mmWave + Lux Aware)
- **File**: `intelligent_living_room_mmwave_lux_aware.yaml`
- **Version**: 1.3 (SIMPLIFIED - removed debounce wait + optimized timing)
- **Color Range**: 1800K-5500K (warmest baseline in the system)
- **Special Features**:
  - Aqara FP2 mmWave presence detection
  - Anti-flicker hysteresis (150 lux OFF / 80 lux ON, 50 lux dead band)
  - Dynamic brightness scaling (100% @ ≤50 lux → 40% @ 150 lux)
  - Scene cycling system (3 Philips Hue scenes via input_number helper)
  - Optimized turn-off timing (day: 15/20/25 min, night: 5/10/15 min)
- **CRITICAL BUG FIX (v1.3)**: Removed debounce wait_template that could block turn-on
  - v1.2 bug: wait_template with continue_on_timeout: false exits on lux fluctuations
  - v1.3 fix: Immediate turn-on/off when lux thresholds crossed (same fix as Adjacent Zone v1.4)
  - Result: Reliable turn-on, instant response, hysteresis alone provides anti-flicker
- **Previous Fix (v1.2)**: Variables recalculated inside continuous presence loop
  - Fresh `loop_*` variables recalculated every 60 seconds for circadian/brightness updates

### 4. Adjacent Zone Motion Sensor
- **File**: `adjacent_zone_motion_circadian_lighting.yaml`
- **Version**: 1.4 (FIXED - turn-on now works + optimized timing)
- **Color Range**: 1800K-5500K (IDENTICAL to living room for color consistency)
- **Design Philosophy**: **100% color consistency with living room blueprint**
  - Extracted EXACT same sun elevation → Kelvin formula
  - Hardcoded lux thresholds (150/80) - NOT configurable
  - Hardcoded brightness curve - exact match to living room
  - Result: Adjacent zones always match living room color temperature
- **Special Features**:
  - Zone preset system (Very Quick 3/5/7 min, Quick 5/10/15 min, Medium 10/15/20 min)
  - Shorter turn-off intervals than living room (optimized for hallways/bathrooms)
  - Configurable brightness curve (v1.1 update)

### 5. Child-Friendly Night Light
- **File**: `child_night_light.yaml`
- **Version**: 1.0
- **Color Mode**: RGB (not color_temp) for Third Reality compatibility
- **Color Range**: 1700K-2700K (ultra-warm for children's sleep)
- **Special Features**:
  - 3 time slots (nighttime + 2 optional naps)
  - Ultra-low brightness (3% default, ~1-2 lux)
  - Motion-activated brightness boost
  - RGB color conversion for non-color_temp lights

## Critical Technical Patterns

### Color Temperature Conversion
```yaml
# Kelvin → Mireds (for Philips Hue color_temp attribute)
color_temp_mireds: "{{ (1000000 / color_temp_kelvin|int) | int }}"

# NEVER send both kelvin AND color_temp in same service call
# This causes "two or more values in the same group of exclusion" error
```

### Variable Scoping and Recalculation (CRITICAL)
**Problem**: Home Assistant blueprint variables defined at top level are calculated ONCE when automation triggers. They don't auto-update inside loops.

**Solution**: For continuous updates during presence, recalculate variables INSIDE loops:
```yaml
- repeat:
    while:
      - condition: state
        entity_id: !input presence_sensor
        state: "on"
    sequence:
      # CRITICAL: Recalculate variables inside loop
      - variables:
          loop_current_lux: "{{ states(lux_sensor_entity) | float(0) }}"
          loop_sun_elevation: "{{ state_attr('sun.sun', 'elevation') | float(0) }}"
          loop_color_temp_kelvin: >
            {% set elevation = loop_sun_elevation %}
            # ... calculate based on FRESH elevation ...
```

**Common Bug**: Using top-level variables inside loops causes stale data (fixed in living room v1.2).

### Anti-Flicker Hysteresis Pattern
```yaml
# Dual thresholds with dead band
lux_on_threshold: 80    # Turn ON when lux drops below 80
lux_off_threshold: 150  # Turn OFF when lux rises above 150
# Result: 50 lux dead band prevents oscillation

# Time-based debouncing
lux_on_debounce_sec: 120   # 2 minutes stable before ON
lux_off_debounce_sec: 300  # 5 minutes stable before OFF
sunrise_sunset_debounce_sec: 600  # 10 minutes during transitions
```

### Staged Turn-Off Pattern
All blueprints use progressive warnings to avoid jarring instant-off:
```yaml
# Stage 1: Dim to 40% + warm to minimum Kelvin (warning)
# Stage 2: Dim to 20% + maintain warm (strong warning)
# Stage 3: Turn off with 3-second fade
```

Living room v1.2 adds time-conditional delays:
```yaml
# Night mode (22:00-06:00): 5/7/10 minutes
# Normal mode: 30/40/45 minutes
effective_delay: "{{ night_delay if is_night_mode_active else normal_delay }}"
```

## Color Consistency Architecture

**Critical Requirement**: Living room and adjacent zones (hallways, bathrooms) are physically adjacent. They MUST show identical color temperature at all times to avoid jarring transitions.

**Implementation**:
1. **Living room blueprint** = source of truth for circadian formula
2. **Adjacent zone blueprint** = EXACT copy of circadian formula (lines extracted verbatim)
3. **Hardcoded lux thresholds** (150/80) in adjacent zone - NOT configurable
4. **Hardcoded brightness curve** in adjacent zone (v1.0 only; v1.1 made configurable but defaults match)
5. **Hardcoded debounce times** (120/300/600 sec)

**Result**: Living room at 3500K = Adjacent hallway at 3500K (guaranteed)

**Difference**: Only turn-off intervals differ (30/40/45 min living room vs 5/10/15 min hallway)

## Scientific Color Temperature Values

Based on peer-reviewed research on melanopsin photopigment response and melatonin regulation. **DO NOT change arbitrarily**:

- **1800K**: Warmest baseline (living room/adjacent zone only)
- **2000K**: Warm sunrise/sunset (original blueprint)
- **2200K**: Night/sleep-friendly (extra warm amber)
- **3000K**: Warm white (morning/evening)
- **4500K**: Neutral white (afternoon)
- **5500K**: Cool daylight (maximum alertness, midday)

Child night light uses 2000K default (research shows minimal melatonin suppression even at 800 lux with warm amber).

## Git Workflow

This repository uses **GitHub Desktop** for push operations (NOT command line):

```bash
# Commit changes via command line
cd "/Users/martinlevie/Documents/GitHub/HomeAssistant-Repo"
git add blueprints/filename.yaml
git commit -m "Descriptive message with UX impact

- Feature 1
- Feature 2

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>"

# Push via GitHub Desktop app (authentication issues with CLI)
open -a "GitHub Desktop" "/Users/martinlevie/Documents/GitHub/HomeAssistant-Repo"
# User clicks "Push origin" in GitHub Desktop
```

## Documentation Sync Requirements

When modifying blueprints, update these files:

### For Living Room Blueprint
1. `blueprints/intelligent_living_room_mmwave_lux_aware.yaml` - Source of truth
2. `README.md` - Update version number, feature highlights, Recent Enhancements section
3. `blueprints/INTELLIGENT_LIVING_ROOM_SETUP_GUIDE.md` - If new inputs added
4. `blueprints/CHANGELOG_vX.X.md` - Create changelog for major versions
5. Commit message MUST explain what was broken and how it was fixed

### For Adjacent Zone Blueprint
1. `blueprints/adjacent_zone_motion_circadian_lighting.yaml` - Source of truth
2. `README.md` - Update version number if changed
3. `blueprints/ADJACENT_ZONE_SETUP_GUIDE.md` - If new inputs added
4. **CRITICAL**: If circadian formula changes in living room, MUST update adjacent zone to match

### For Other Blueprints
1. Blueprint YAML file
2. `README.md` - Feature list in Available Blueprints section
3. Relevant documentation in `/blueprints/` or `/docs/`

## Testing Workflow

Users test blueprints by re-importing from GitHub:

```
# Import URL format (raw GitHub, not regular page)
https://raw.githubusercontent.com/leviemartin/HomeAssistant-Repo/main/blueprints/{filename}.yaml

# User workflow
1. Make changes, commit, push via GitHub Desktop
2. User goes to Home Assistant → Settings → Automations & Scenes → Blueprints
3. Find blueprint → Three dots → "Re-import Blueprint"
4. Existing automations automatically use updated blueprint
```

## Common Issues and Solutions

### Issue: "Two or more values in the same group of exclusion"
**Cause**: Sending both `kelvin` and `color_temp` in same `light.turn_on` service call
**Fix**: Use ONLY `color_temp` (mireds) for Philips Hue, remove `kelvin`

### Issue: Brightness value is empty
**Cause**: Conditional brightness not properly templated
**Fix**: Use conditional key inclusion:
```yaml
data: >
  {% set data = {'color_temp': color_temp_mireds, 'transition': 4} %}
  {% if dynamic_brightness %}
    {% set data = dict(data, brightness_pct=calculated_brightness) %}
  {% endif %}
  {{ data }}
```

### Issue: Circadian colors not updating during occupancy
**Cause**: Variables calculated once at start, never recalculated in loop (v1.1 bug)
**Fix**: Add `variables:` block INSIDE repeat loop with `loop_*` prefixed fresh values (v1.2 fix)

### Issue: Lights not turning on despite lux being low
**Cause**: Debounce wait_template with `continue_on_timeout: false` exits automation if lux fluctuates
**Fix**: Remove wait_template, use simple condition check + immediate service call (v1.3/v1.4 fix)
```yaml
# BAD (v1.2/v1.3 bug):
- wait_template: "{{ states(lux_sensor) | float < threshold }}"
  timeout: { seconds: 120 }
  continue_on_timeout: false  # ← Exits if lux fluctuates!
- service: light.turn_on  # Never reached if timeout

# GOOD (v1.3/v1.4 fix):
- service: light.turn_on  # Immediate, no wait
  target: !input lights
  data:
    brightness_pct: "{{ brightness }}"
```

### Issue: Transition values causing errors
**Cause**: Passing string instead of number
**Fix**: Use `{{ transition_time | int }}` or define as number in variables

### Issue: Third Reality lights not responding to color_temp
**Cause**: Third Reality uses RGB, not color_temp attribute
**Fix**: Use RGB color conversion:
```yaml
rgb_color_value: >
  {% if color_temp_kelvin|int == 2000 %}
    [255, 147, 41]
  {% elif color_temp_kelvin|int == 2200 %}
    [255, 160, 54]
  # ... etc
```

## Blueprint Versioning

- **Major version** (x.0): Architectural changes, new features, breaking changes
- **Minor version** (1.x): Bug fixes, small enhancements, backward compatible
- Always update `name:` field in blueprint with version number
- Create CHANGELOG file for major versions (see `CHANGELOG_v1.2.md`)
- Update README.md "Recent Enhancements" section with version-specific changes

## User Experience Requirements

- **Smooth transitions**: Never instant (jarring), use 1.5-4 second fades
- **Staged warnings**: Multi-step turn-off prevents surprising darkness
- **User-friendly inputs**: All inputs use selectors (dropdown/slider/time), no manual entity_id typing
- **Sensible defaults**: Blueprints work out-of-box based on scientific research
- **Clear descriptions**: Input descriptions explain WHY (research basis) not just WHAT

## Home Assistant Specific Notes

- Blueprint `mode: restart` means motion detection resets automation (prevents early turn-off)
- Time-based conditions require seconds-since-midnight conversion for numeric comparisons
- Midnight wraparound requires special handling (e.g., 22:00-06:00 spans two days)
- `state_attr('sun.sun', 'elevation')` returns sun angle in degrees (-90 to +90)
- Mireds range: ~153 (6500K cool) to ~500 (2000K warm), smaller number = cooler
