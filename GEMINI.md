# GEMINI.md

This file provides guidance to Google Gemini when working with code in this repository.

## Repository Purpose

This is a Home Assistant blueprint repository containing **6 production blueprints** for intelligent lighting automation. All blueprints are scientifically-backed, optimized for Philips Hue compatibility, and designed to promote better sleep, health, and wellbeing through optimized light control.

## Repository Structure

```
/blueprints/                              # Production blueprint files
  circadian_rhythm_lighting.yaml          # Original fixed-time circadian (v1.0)
  circadian_rhythm_lighting_sun_aware.yaml # Sun-tracking circadian (v2.0)
  intelligent_living_room_mmwave_lux_aware.yaml # mmWave + lux-aware (v1.10)
  adjacent_zone_motion_circadian_lighting.yaml # Adjacent zone motion (v1.7)
  child_night_light.yaml                  # Child-friendly night light (v1.0)
  child_wake_indicator_night_light.yaml   # OK-to-Wake indicator light (v1.0)

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
- **Version**: 1.10 (CRITICAL FIX - responsive presence detection in loop for immediate turn-off)
- **Color Range**: 1800K-5500K (warmest baseline in the system)
- **Special Features**:
  - Aqara FP2 mmWave presence detection
  - Anti-flicker hysteresis (150 lux OFF / 80 lux ON, 50 lux dead band)
  - Dynamic brightness scaling (100% @ ≤50 lux → 40% @ 150 lux)
  - Scene cycling system (3 Philips Hue scenes via input_number helper)
  - Optimized turn-off timing (day: 15/20/25 min, night: 5/10/15 min)
- **CRITICAL FIX (v1.10)**: Added responsive presence detection in continuous monitoring loop
  - v1.9 bug: Loop only checked "until" condition after each 60-second iteration completed
  - If presence cleared during delay, loop wouldn't detect it until after delay + recalculation + updates
  - This created a window (up to 60 seconds) where triggers could restart automation and cancel turn-off
  - Result: Turn-off sequence never ran, lights stayed on even with no presence
  - v1.10 fix: Added presence check immediately after delay (lines 1223-1230)
  - If presence is "off", skip all variable recalculation and light updates
  - Loop immediately proceeds to "until" check, which exits cleanly
  - Result: Turn-off triggers within 1 second instead of up to 60 seconds after presence clears!
  - User report fixed: "For some reason the turn off steps never get triggered" - SOLVED!
- **Previous Fix (v1.9)**: Added presence guard to prevent lux_dropped from interrupting turn-off
  - v1.8 bug: lux_dropped trigger could fire during turn-off delay (lux fluctuations from clouds/curtains)
  - With mode:restart, this CANCELLED the running automation and restarted it
  - Branch 3 conditions failed (presence was "off"), automation ended without turn-off
  - Lux fluctuations repeatedly interrupted turn-off → lights stayed on forever
  - v1.9 fix: Added presence state check to Branch 3 conditions (lines 1166-1168)
  - Now lux_dropped only runs Branch 3 if presence is actually "on"
  - When presence is "off", lux_dropped won't restart automation or interrupt turn-off
  - Result: Turn-off sequence completes successfully even with lux sensor noise
- **Previous Fix (v1.8)**: Fixed staged turn-off not running when presence clears
  - v1.7 bug: Condition inside repeat loop checked if presence was "on"
  - When presence cleared to "off", condition FAILED, causing loop to error out
  - Automation exited WITHOUT reaching staged turn-off sequence (lights stayed on forever)
  - v1.8 fix: Removed blocking condition from inside repeat sequence
  - Now uses ONLY the "until" condition to cleanly exit loop
  - Result: Loop exits properly when presence clears, staged turn-off runs correctly
- **Previous Fix (v1.7)**: Fixed continuous monitoring loop not running + loop variable boolean bugs
  - v1.6 bug: Branch 2C (lux_dropped) turned on lights then EXITED - no continuous updates
  - v1.6 bug: loop_override_active, loop_scene_is_active, time_override returned STRING "false"
  - v1.7 fix: Merged Branch 2C into Branch 3 using OR condition (presence OR lux_dropped)
  - v1.7 fix: Updated loop variables to use AND chain syntax → returns actual booleans
  - Result: Continuous loop now runs for both presence_detected and lux_dropped triggers
  - Result: Circadian colors update every 60 seconds, brightness scales with lux
  - User report fixed: "Stuck in Option 6, no circadian/brightness changes" - SOLVED!
  - **CRITICAL**: Branch 3 handles BOTH presence_detected AND lux_dropped triggers via OR condition
  - **CRITICAL**: Continuous monitoring loop MUST be inside Branch 3 to run for all entry points
- **Previous Feature (v1.6)**: Added lux_dropped trigger for complete lux monitoring
  - v1.5 had only lux_exceeded trigger (turn OFF when bright)
  - v1.6 adds lux_dropped trigger (turn ON when dark)
  - Trigger 6: `numeric_state below lux_turn_on_threshold`
  - Lights turn ON immediately when room darkens during presence (sunset, curtains closed)
  - Safety: Only activates if presence detected (won't light empty room)
- **Previous Fix (v1.5)**: Fixed string vs boolean bug in override_active and scene_is_active variables
  - v1.4 bug: Variables returned STRING "false" instead of BOOLEAN false
  - Impact: `not "false"` = FALSE (any string is truthy in Jinja2!)
  - Branch 4 condition `{{ not scene_is_active }}` always failed, blocking ALL automation actions
  - v1.5 fix: Simplified templates using AND chain syntax to return actual booleans
  - Result: Automation now executes actions properly when presence detected
- **Previous Fix (v1.4)**: Added lux_exceeded trigger for proper lux monitoring
  - v1.3 bug: Only checked lux in 60-second loop, could miss rapid changes or have delays
  - v1.4 fix: Added trigger that fires when lux > 150 (OFF threshold)
  - Result: Lights turn off immediately when room becomes bright (sunrise, curtains opened)
  - Branch 2B handles lux_exceeded trigger, respects override/scene settings
- **Previous Fix (v1.3)**: Removed debounce wait_template that could block turn-on
  - Immediate turn-on/off when lux thresholds crossed
- **Previous Fix (v1.2)**: Variables recalculated inside continuous presence loop
  - Fresh `loop_*` variables recalculated every 60 seconds for circadian/brightness updates

### 4. Adjacent Zone Motion Sensor
- **File**: `adjacent_zone_motion_circadian_lighting.yaml`
- **Version**: 1.7 (NEW FEATURE - lux sensor is now optional, perfect for windowless bathrooms)
- **Color Range**: 1800K-5500K (IDENTICAL to living room for color consistency)
- **Design Philosophy**: **100% color consistency with living room blueprint**
  - Extracted EXACT same sun elevation → Kelvin formula
  - Hardcoded lux thresholds (150/80) when sensor configured - NOT configurable
  - Optional lux sensor - can be left empty for windowless rooms
  - Result: Adjacent zones always match living room color temperature
- **Special Features**:
  - Hardcoded Quick timing (5/10/15 min = 15 min total) - optimized for hallways/bathrooms
  - Configurable brightness curve (v1.1 update)
  - PIR motion sensor compatible (Philips Hue, Aqara P1)
- **NEW FEATURE (v1.7)**: Optional lux sensor for windowless rooms
  - Lux sensor input now has `default: {}` (optional)
  - Added `lux_sensor_enabled` variable to check if sensor configured
  - Turn-on logic: `{{ not lux_sensor_enabled or current_lux < lux_on_threshold }}`
  - Without sensor: Motion ALWAYS turns on lights (perfect for bathrooms)
  - With sensor: Uses hardcoded 150/80 lux thresholds (living room consistency)
  - Removed lux_exceeded trigger (can't be conditional in YAML)
  - Removed Branch 2 (lux_exceeded action handler)
- **Previous Fix (v1.6)**: Added lux numeric_state trigger for proper lux monitoring
  - v1.4 bug: Only checked lux at motion trigger time, never monitored during wait periods
  - v1.5 bug: Import error - referenced non-existent `!input lux_turn_off_threshold`
  - v1.6 fix: Uses hardcoded value `above: 150` (matches hardcoded variable lux_off_threshold)
  - Result: Lights turn off immediately when room becomes bright (sunrise, curtains opened)
  - Branch 2 handles lux_exceeded trigger, respects override settings
- **Previous Fix (v1.4)**: Removed debounce wait_template that could block turn-on
  - Immediate turn-on, faster timing (15 min total)

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

### 6. Child Wake-Up Indicator Night Light (OK-to-Wake)
- **File**: `child_wake_indicator_night_light.yaml`
- **Version**: 1.1
- **Color Mode**: RGB (not color_temp) for Third Reality compatibility
- **Design Philosophy**: "Red means bed, Green means GO!" stoplight system
- **Special Features**:
  - Per-day configurable wake-up times (7 inputs, one per day)
  - Sleep color (default: warm red [255, 50, 0]) signals "stay in bed"
  - Wake-up color (default: green [0, 255, 0]) signals "OK to get up"
  - Optional warning color (yellow [255, 180, 0]) 5-30 min before wake-up
  - Gradual wake-up transition (10-300 seconds configurable)
  - Motion-activated brightness boost during sleep time only
  - Wake-up auto-off after configurable duration (15-180 min)
  - **Manual activation (v1.1)**: Click "Run Actions" to activate based on current time
- **Mode**: `queued` (max: 3) - ensures reliable operation
  - Prevents restart mode issues (motion cancelling wake-up sequence)
  - All time-based triggers processed in order
- **Key Implementation Details**:
  - Uses `trigger_variables` for wake-up time calculations at trigger evaluation
  - Template trigger for warning time with 59-second window for reliability
  - Day validation prevents wrong day's trigger from running
  - State recalculation after motion boost handles time boundary crossings
- **Research Basis**:
  - Red/amber minimizes melatonin suppression (Nagare et al., 2019)
  - Green for wake-up follows pediatric sleep consultant recommendations
  - Gradual transitions reduce startle response
  - Ultra-low brightness (1-5%) protects developing eyes

## Critical Technical Patterns

### Continuous Monitoring Loop Architecture (Living Room Blueprint)

**CRITICAL PATTERN**: The continuous monitoring loop MUST be accessible from ALL entry points that turn on lights.

**Architecture:**
```yaml
action:
  - choose:
      # BRANCH 1: Override toggle (no loop needed)

      # BRANCH 2B: lux_exceeded (turn OFF only, no loop needed)

      # BRANCH 3: MAIN ENTRY POINT - Handles multiple triggers
      - conditions:
          - or:  # ← CRITICAL: OR condition for multiple entry points
              - condition: trigger
                id: presence_detected
              - condition: trigger
                id: lux_dropped
        sequence:
          # Turn on lights (with lux/override checks)
          - choose: [...]

          # === CONTINUOUS MONITORING LOOP ===
          - repeat:
              sequence:
                - delay: { seconds: 60 }

                # CRITICAL: Recalculate ALL dynamic variables
                - variables:
                    loop_current_lux: "{{ states(lux_sensor) }}"
                    loop_sun_elevation: "{{ state_attr('sun.sun', 'elevation') }}"
                    loop_color_temp_kelvin: "{{ calculate_circadian() }}"
                    loop_calculated_brightness: "{{ calculate_brightness() }}"
                    # MUST use AND chain for booleans!
                    loop_override_active: "{{ enabled and entity and is_state(...) }}"
                    loop_scene_is_active: "{{ enabled and entity and (states(...) > 0) }}"

                # Update lights with fresh values
                - choose:
                    - conditions: "{{ loop_scene_is_active }}"
                      sequence: [skip update]
                    - conditions: "{{ loop_override_active or (loop_current_lux < threshold) }}"
                      sequence:
                        - service: light.turn_on
                          data:
                            brightness_pct: "{{ loop_calculated_brightness }}"
                            color_temp: "{{ loop_color_temp_mireds }}"
              until:
                - condition: state
                  entity_id: !input presence_sensor
                  state: "off"
```

**Why This Matters:**
- v1.6 bug: Branch 2C (lux_dropped) turned on lights then EXITED - no loop ran
- v1.7 fix: Merged Branch 2C into Branch 3 using OR condition
- Result: BOTH presence_detected AND lux_dropped trigger the continuous loop
- Circadian colors and brightness now update every 60 seconds for ALL entry points

**Common Mistake:**
```yaml
# BAD - Each trigger has separate branch
- conditions:
    - condition: trigger
      id: presence_detected
  sequence:
    - [turn on lights]
    - repeat: [continuous loop]  # ← Loop runs for presence

- conditions:
    - condition: trigger
      id: lux_dropped
  sequence:
    - [turn on lights]  # ← Loop does NOT run for lux_dropped!
```

**Best Practice:**
- Use OR condition to handle multiple triggers in ONE branch
- Place continuous monitoring loop AFTER initial turn-on logic
- Always recalculate variables inside loop (don't reuse top-level variables)
- Use AND chain syntax for boolean variables in loop scope
- **CRITICAL**: Do NOT use conditions inside repeat sequence to check exit criteria
- Use ONLY the "until" condition to exit the loop cleanly

**Common Mistake #2: Blocking Condition Inside Loop**
```yaml
# BAD - Condition inside sequence blocks exit
- repeat:
    sequence:
      - delay: { seconds: 60 }
      - condition: state  # ← BLOCKS when presence clears!
        entity_id: !input presence_sensor
        state: "on"
      - [update lights]
    until:
      - condition: state
        entity_id: !input presence_sensor
        state: "off"
# Problem: When presence clears, inner condition FAILS, loop errors out
# Never reaches the "until" check or the staged turn-off code after loop

# GOOD - No condition inside sequence
- repeat:
    sequence:
      - delay: { seconds: 60 }
      - [recalculate variables]
      - [update lights]
    until:
      - condition: state
        entity_id: !input presence_sensor
        state: "off"
# Result: Loop completes iteration, checks "until", exits cleanly
```

**Why This Matters:**
- v1.7 bug: Condition inside loop prevented staged turn-off from running
- When presence cleared, condition failed, automation exited early
- Lights stayed on forever (no turn-off sequence ran)
- v1.8 fix: Removed condition from inside sequence
- Result: Loop exits cleanly via "until", staged turn-off runs properly

### Optional Sensor Pattern (Adjacent Zone v1.7)

**Use Case**: Making sensors optional for different room types (e.g., windowless bathrooms don't need lux sensors).

**Pattern:**
```yaml
# INPUT - Make sensor optional with default: {}
inputs:
  lux_sensor:
    name: Lux Sensor (Optional)
    default: {}
    selector:
      entity:
        filter:
          - domain: sensor
            device_class: illuminance

# VARIABLES - Check if sensor is configured
variables:
  lux_sensor_entity: !input lux_sensor

  # Boolean check for sensor availability
  lux_sensor_enabled: >
    {{ lux_sensor_entity not in [none, '', 'unavailable', 'unknown'] and lux_sensor_entity | length > 0 }}

  # Conditional sensor reading
  current_lux: >
    {% if lux_sensor_enabled %}
      {{ states(lux_sensor_entity) | float(0) }}
    {% else %}
      0
    {% endif %}

# TRIGGERS - Cannot conditionally include triggers in YAML
# Solution: Remove lux-based triggers, handle in conditions instead
trigger:
  - platform: state
    entity_id: !input motion_sensor
    to: "on"
  # NOTE: No lux trigger when sensor is optional

# ACTIONS - Use conditional logic
action:
  - choose:
      # Turn on lights if: override OR no lux sensor OR lux below threshold
      - conditions:
          - condition: template
            value_template: "{{ not lux_sensor_enabled or current_lux < lux_on_threshold }}"
        sequence:
          - service: light.turn_on
```

**Why This Works:**
- `default: {}` makes input optional (won't error if empty)
- `lux_sensor_enabled` safely checks if sensor exists
- Conditional logic: `not lux_sensor_enabled or current_lux < threshold`
- Without sensor: Motion ALWAYS turns on lights
- With sensor: Motion turns on lights only if dark enough

**Applied In:**
- Adjacent Zone v1.7 (optional lux for windowless bathrooms)

**Common Mistake #3: Triggers Interrupting Turn-Off with mode:restart**

**The Bug (Living Room v1.8):**
```yaml
mode: restart  # Any new trigger CANCELS currently running automation

trigger:
  - platform: numeric_state
    entity_id: !input lux_sensor
    below: !input lux_turn_on_threshold
    id: lux_dropped  # Fires when lux drops below threshold

action:
  - choose:
      # BRANCH 3: Presence OR lux_dropped
      - conditions:
          - or:
              - condition: trigger
                id: presence_detected
              - condition: trigger
                id: lux_dropped
          # BUG: No presence check! lux_dropped can fire even when presence is "off"
        sequence:
          # ... turn on lights, run continuous loop ...
          # When presence clears, loop exits
          # Start staged turn-off (5 minute delay)
          - delay: { seconds: 300 }  # ← During this delay...
          # lux fluctuates and drops below threshold
          # lux_dropped triggers again!
          # mode:restart CANCELS this automation
          # Turn-off sequence never completes!
          - service: light.turn_off  # ← Never reached!
```

**The Problem:**
1. Presence clears → loop exits → staged turn-off starts
2. During the 5-10 minute delay, lux sensor fluctuates (clouds, curtains, etc.)
3. Lux drops below ON threshold → `lux_dropped` trigger fires
4. With `mode: restart`, automation RESTARTS and CANCELS the turn-off
5. Branch 3 checks conditions → presence is "off" → branch skips
6. Automation ends, no turn-off sequence runs
7. Lights stay on forever!
8. If lux keeps fluctuating, turn-off is interrupted repeatedly

**The Fix (Living Room v1.9):**
```yaml
action:
  - choose:
      # BRANCH 3: Presence OR lux_dropped (with presence guard)
      - conditions:
          - or:
              - condition: trigger
                id: presence_detected
              - condition: trigger
                id: lux_dropped
          # FIX: Add presence check to prevent interruption!
          - condition: state
            entity_id: !input presence_sensor
            state: "on"
        sequence:
          # Now lux_dropped only runs if presence is actually detected
          # When presence is "off", lux_dropped won't restart automation
          # Turn-off sequence can complete without interruption
```

**Why This Matters:**
- With `mode: restart`, ANY trigger will cancel the running automation
- Numeric state triggers (lux, temperature, etc.) can fire frequently due to sensor noise
- Time pattern triggers (circadian updates) fire on schedule regardless of state
- Must add guards to prevent unwanted triggers during critical sequences like turn-off
- Always check if the triggering condition is still valid (e.g., presence "on") before running branch

**Key Lesson:**
When using `mode: restart`, carefully review ALL triggers and their corresponding branch conditions. Ask: "Should this branch run when presence is off or lights are turning off?" If not, add appropriate state checks.

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

### Issue: Lights staying on all day in sunny conditions
**Cause**: Missing lux sensor numeric_state trigger - automation never fires when lux changes
**Fix**: Add numeric_state trigger that fires when lux exceeds OFF threshold (v1.4/v1.5 fix)
```yaml
# BAD (v1.3/v1.4 bug):
trigger:
  - platform: state
    entity_id: !input motion_sensor
    to: "on"
# ← Only motion triggers, lux changes ignored!

# GOOD (v1.4/v1.5 fix):
trigger:
  - platform: state
    entity_id: !input motion_sensor
    to: "on"
  - platform: numeric_state
    entity_id: !input lux_sensor
    above: !input lux_turn_off_threshold
    id: lux_exceeded  # ← Automation fires when bright!

# Then add action branch:
- conditions:
    - condition: trigger
      id: lux_exceeded
  sequence:
    - service: light.turn_off
      target: !input lights
      data:
        transition: 5
```

### Issue: Automation not executing any actions despite presence
**Cause**: Variables returning STRING "false" instead of BOOLEAN false (v1.4/v1.6 bug)
**Fix**: Simplify templates to return actual boolean values using AND chain syntax (v1.5/v1.7 fix)
```yaml
# BAD (v1.4/v1.6 bug):
override_active: >
  {% if override_system_enabled and override_trigger_entity %}
    {{ is_state(override_trigger_entity, 'on') }}
  {% else %}
    false  # ← Returns STRING "false" not boolean!
  {% endif %}

scene_is_active: >
  {% if scene_cycling_system_enabled and scene_tracker_entity %}
    {{ (states(scene_tracker_entity) | int(0)) > 0 }}
  {% else %}
    false  # ← Returns STRING "false" not boolean!
  {% endif %}

# Problem: In conditions
- condition: template
  value_template: "{{ not scene_is_active }}"
  # not "false" = FALSE (any non-empty string is truthy in Jinja2!)
  # Result: Condition ALWAYS fails, blocking all actions

# GOOD (v1.5/v1.7 fix):
override_active: >
  {{ override_system_enabled and override_trigger_entity and is_state(override_trigger_entity, 'on') }}
  # Returns actual BOOLEAN True/False

scene_is_active: >
  {{ scene_cycling_system_enabled and scene_tracker_entity and (states(scene_tracker_entity) | int(0)) > 0 }}
  # Returns actual BOOLEAN True/False

# Result: not false = TRUE (correct boolean logic!)
```

**CRITICAL NOTE**: This bug affected BOTH top-level variables AND loop variables:
- v1.5 fixed top-level variables (override_active, scene_is_active)
- v1.7 fixed loop variables (loop_override_active, loop_scene_is_active, time_override)
- **Always check BOTH scopes** when fixing boolean bugs!

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
