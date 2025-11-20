# Intelligent Living Room Blueprint - v1.2 Changelog

## Version 1.2 - Critical Bug Fixes and Night Mode

**Release Date:** 2025-11-20

### CRITICAL BUGS FIXED

#### Problem 1: Circadian Colors NOT Updating During Occupancy (FIXED)

**Issue:** When the room was occupied all day, lights were NOT changing color temperature throughout the day. Colors stayed static despite the circadian rhythm being enabled.

**Root Cause:**
- The continuous presence monitoring loop (lines 1125-1188 in v1.1) was using stale variable values
- Variables like `current_lux`, `sun_elevation`, `color_temp_mireds`, and `calculated_brightness` were calculated ONCE when the automation triggered
- These values were NEVER recalculated during the repeat loop
- Result: Lights would turn on with correct initial color/brightness, but never update as sun position changed

**Fix Applied:**
- Added a new `variables` block INSIDE the repeat loop (lines 1259-1329 in v1.2)
- All critical values are now recalculated on each iteration (every 60 seconds):
  - `loop_current_lux` - Fresh lux reading
  - `loop_sun_elevation` - Current sun position
  - `loop_color_temp_kelvin` - Recalculated circadian color (1800K-5500K)
  - `loop_color_temp_mireds` - Converted to Philips Hue format
  - `loop_calculated_brightness` - Dynamic brightness based on current lux
  - `loop_override_active` - Current override state
  - `loop_scene_is_active` - Current scene state
- Light update commands now use these fresh loop variables instead of stale top-level variables

**Result:** Circadian colors and dynamic brightness now update continuously every 60 seconds while room is occupied.

---

#### Problem 2: Dynamic Brightness NOT Updating During Occupancy (FIXED)

**Issue:** Brightness should scale inversely with natural light (100% at ≤50 lux → 40% at 150 lux), but was not updating during occupancy.

**Root Cause:** Same as Problem 1 - stale `calculated_brightness` variable.

**Fix Applied:** Same as Problem 1 - `loop_calculated_brightness` is now recalculated every 60 seconds inside the repeat loop.

**Result:** Brightness now dynamically adjusts throughout the day based on changing natural light levels.

---

#### Problem 3: Branch 4 Time Pattern Trigger Had Broken Condition (FIXED)

**Issue:** Line 1261 in v1.1 attempted to check if lights were on using invalid syntax: `is_state_attr(target_lights.entity_id[0]...)`

**Root Cause:** `target_lights` is a selector input, not directly accessible as an entity_id reference.

**Fix Applied:**
- Simplified Branch 4 to only trigger when presence is detected (line 1461-1463)
- Added proper variable recalculation inside Branch 4 (lines 1466-1521)
- Branch 4 now serves as a complementary update mechanism to the main presence loop
- Uses fresh variables with `time_` prefix to avoid confusion with other scopes

**Result:** Branch 4 now works correctly as a backup circadian update mechanism.

---

### NEW FEATURE: Night Mode Fast Shutdown

#### Problem 3: Slow Shutdown After 10pm (SOLVED)

**Issue:** After 10pm when going to bed, the 45-minute staged shutdown (30/40/45 min) was too long. User wanted faster response to no presence during late evening hours.

**Solution:** Added configurable "Night Mode" system with time-based conditional logic.

**New Inputs Added:**
1. **Enable Night Mode Fast Shutdown** (default: true)
   - Toggle to enable/disable night mode feature

2. **Night Mode Start Time** (default: 22:00)
   - Time when night mode begins (24-hour format)

3. **Night Mode End Time** (default: 06:00)
   - Time when night mode ends (24-hour format)

4. **Night Mode Stage Delays:**
   - Stage 1: 5 minutes (default) - First warning
   - Stage 2: 7 minutes (default) - Second warning
   - Stage 3: 10 minutes (default) - Complete turn-off

**How It Works:**
- Blueprint automatically detects if current time is within night mode window
- Handles midnight crossing correctly (e.g., 22:00 to 06:00 next day)
- When night mode is active, uses night stage delays instead of normal delays
- When night mode is inactive, uses normal stage delays (30/40/45 min)
- Seamless switching - no user intervention required

**Variables Added:**
- `is_night_mode_active` - Boolean check if current time is in night window
- `effective_stage_1_delay` - Switches between normal/night delay
- `effective_stage_2_delay` - Switches between normal/night delay
- `effective_stage_3_delay` - Switches between normal/night delay

**Implementation:**
- All three staged turn-off sequences updated to use `effective_stage_X_delay` variables:
  - Approach detection branch (lines 1147-1185)
  - Main presence detection branch (lines 1406-1444)
  - Already correctly calculated with midnight crossing logic (lines 709-745)

**Result:** After 10pm, lights turn off in 5/7/10 minutes instead of 30/40/45 minutes. Perfect for bedtime routines.

---

## Technical Changes Summary

### Files Modified
- `/Users/martinlevie/Documents/GitHub/HomeAssistant-Repo/blueprints/intelligent_living_room_mmwave_lux_aware.yaml`

### Lines Changed
- **Version:** Line 2 (v1.1 → v1.2)
- **Description:** Line 9 (updated changelog reference)
- **New Inputs:** Lines 560-631 (night mode configuration section)
- **New Variables:** Lines 699-745 (night mode logic and effective delays)
- **Fixed Loop:** Lines 1257-1376 (recalculation of all dynamic values)
- **Fixed Delays:** Lines 1147, 1159, 1170, 1179, 1407, 1418, 1429, 1439 (use effective delays)
- **Fixed Branch 4:** Lines 1449-1535 (recalculate variables, simplified conditions)

### Maintained Compatibility
- 100% color consistency with adjacent zone blueprint (same 1800K-5500K curve)
- Hardcoded lux thresholds preserved (150/80 lux)
- Anti-flicker hysteresis protection intact
- Scene cycling feature preserved
- Manual override system preserved
- All existing configuration options preserved

---

## Testing Recommendations

### Test 1: Circadian Color Updates During Occupancy
**Steps:**
1. Stay in room for 2+ hours during daytime
2. Ensure lux is below turn-off threshold (< 150 lux)
3. Watch lights every 60 seconds

**Expected Result:**
- Lights should gradually shift color temperature as sun moves
- Morning: Warm (1800K-3000K)
- Midday: Cool (5000K-5500K)
- Evening: Warm (1800K-3000K)
- Changes should be smooth and continuous

**How to Monitor:**
- Check light entity color_temp attribute in Developer Tools
- Should see mireds value changing every 60 seconds
- Formula: kelvin = 1,000,000 / mireds

---

### Test 2: Dynamic Brightness Updates During Occupancy
**Steps:**
1. Stay in room during changing light conditions
2. Close/open curtains to change natural light levels
3. Monitor brightness changes

**Expected Result:**
- As lux increases (more natural light), brightness decreases
- As lux decreases (less natural light), brightness increases
- Updates should happen every 60 seconds
- Smooth interpolation between brightness levels

**Brightness Scale:**
- ≤50 lux: 100% brightness
- 75 lux: 85% brightness
- 100 lux: 70% brightness
- 125 lux: 55% brightness
- 150 lux: 40% brightness

---

### Test 3: Night Mode Fast Shutdown
**Steps:**
1. Set night mode start time to current time + 2 minutes
2. Ensure presence is detected
3. Leave room after night mode starts
4. Time the staged turn-off sequence

**Expected Result:**
- Before night mode: 30/40/45 minute delays (if tested before night mode)
- After night mode starts: 5/7/10 minute delays
- Should see faster dimming sequence
- Lights should turn off after 10 minutes (vs 45 minutes normally)

**How to Verify:**
- Check automation trace in Developer Tools
- Look for delay durations in trace
- Should show 300/420/600 seconds instead of 1800/2400/2700

---

### Test 4: Night Mode Midnight Crossing
**Steps:**
1. Set night mode: 22:00 to 06:00
2. Test leaving room at 23:00 (during night mode)
3. Test leaving room at 07:00 (after night mode)
4. Test leaving room at 05:00 (still in night mode)

**Expected Result:**
- 23:00 test: Fast shutdown (5/7/10 min)
- 07:00 test: Normal shutdown (30/40/45 min)
- 05:00 test: Fast shutdown (5/7/10 min)

---

### Test 5: Scene Override Behavior (Regression Test)
**Steps:**
1. Activate a scene via scene cycle button
2. Stay in room for 2+ hours
3. Verify colors DON'T change (scene preserved)
4. Leave room and verify scene bypass settings

**Expected Result:**
- While scene active: NO circadian updates (colors stay as scene defined)
- Scene bypass enabled: Lights stay on when leaving
- Scene bypass disabled: Staged turn-off still works
- Logs should show "Skipping circadian update - scene is active"

---

### Test 6: Override System Behavior (Regression Test)
**Steps:**
1. Activate manual override (toggle override switch)
2. Increase lux above OFF threshold (> 150 lux)
3. Verify lights stay on despite high lux
4. Monitor circadian updates continue

**Expected Result:**
- Lights stay on regardless of lux level
- Circadian colors still update every 60 seconds
- Dynamic brightness still adjusts
- Only lux-based turn-off is disabled

---

## Configuration Changes Needed

### For Existing Users (Upgrading from v1.1)

**No configuration changes required!** The blueprint will use default night mode settings:
- Night mode: ENABLED
- Start: 22:00 (10pm)
- End: 06:00 (6am)
- Night delays: 5/7/10 minutes

**Optional Customization:**
If you want different night mode timing:
1. Open automation in Home Assistant
2. Navigate to "Night Mode (Fast Shutdown After Hours)" section
3. Adjust start/end times to your preference
4. Adjust stage delays if 5/7/10 minutes is too fast or too slow
5. Or disable night mode entirely if you prefer consistent timing

---

## Migration Notes

### From v1.1 to v1.2
- **Breaking Changes:** NONE
- **Deprecations:** NONE
- **New Required Helpers:** NONE
- **Behavioral Changes:**
  - Circadian colors now update during occupancy (was broken before)
  - Night mode fast shutdown is active by default (can be disabled)

### Compatibility
- Home Assistant 2023.1 or newer
- Philips Hue, IKEA Tradfri, any tunable white lights
- Aqara FP2/EP1, any mmWave or PIR presence sensors
- All existing configurations from v1.0 and v1.1 are compatible

---

## Known Issues

### None currently identified

If you discover issues, please report with:
- Blueprint version (v1.2)
- Home Assistant version
- Light entity details (brand, model, supported features)
- Automation trace from Developer Tools
- Logs showing the issue

---

## Credits

**Blueprint Author:** Martin Levie
**Repository:** https://github.com/martinlevie/HomeAssistant-Repo
**Blueprint Type:** Living Room Lighting Automation
**License:** MIT (or your chosen license)

---

## Detailed Technical Explanation

### Why Variables Became Stale in v1.1

In Home Assistant blueprints and automations:
- Variables defined in the top-level `variables` section are calculated ONCE when the automation triggers
- These variables are snapshots of entity states at trigger time
- Inside a `repeat` loop, referencing these variables returns the ORIGINAL value from trigger time
- Template expressions like `{{ current_lux }}` inside a loop still reference the stale trigger-time value

**The Fix:**
- Add a NEW `variables` block inside the repeat sequence
- Home Assistant recalculates these variables on each loop iteration
- Use different variable names (e.g., `loop_current_lux`) to avoid confusion
- Update all service calls to use the fresh loop variables

### Why This Is Critical for Circadian Lighting

Sun elevation changes continuously:
- Every 60 seconds, sun moves approximately 0.25 degrees
- Color temperature is calculated from sun elevation
- If using stale elevation value, color never changes

Example:
```yaml
# BROKEN (v1.1):
variables:
  sun_elevation: "{{ state_attr('sun.sun', 'elevation') }}"  # Calculated once at 8am: 10°

repeat:
  sequence:
    - service: light.turn_on
      data:
        color_temp: "{{ calculate_from(sun_elevation) }}"  # Always uses 10° from 8am!
```

```yaml
# FIXED (v1.2):
repeat:
  sequence:
    - variables:
        loop_sun_elevation: "{{ state_attr('sun.sun', 'elevation') }}"  # Fresh value each iteration!
    - service: light.turn_on
      data:
        color_temp: "{{ calculate_from(loop_sun_elevation) }}"  # Updates with actual sun position!
```

---

## Version History

- **v1.0** - Initial release with mmWave, lux hysteresis, circadian rhythm, staged turn-off
- **v1.1** - Added scene cycling feature with single-button control
- **v1.2** - Fixed circadian/brightness updates during occupancy, added night mode fast shutdown

---

## Next Version Plans (v1.3)

Potential features for future release:
- Adaptive timeout: Learn occupancy patterns and adjust delays dynamically
- Seasonal adjustments: Different delays for summer vs winter
- Weather integration: Adjust lux thresholds based on cloud cover
- Multi-zone coordination: Sync colors across adjacent rooms
- Guest mode: Temporarily disable automation for visitors

Feedback welcome!
