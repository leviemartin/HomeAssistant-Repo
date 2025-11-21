# Home Assistant Blueprints - Research-Backed Lighting Automation

Scientifically-backed Home Assistant blueprints for intelligent lighting automation. Each blueprint is built on peer-reviewed research to promote better sleep, health, and wellbeing through optimized light control.

## Available Blueprints

### 1. Circadian Rhythm Lighting with Motion Control
Automatically adjusts light color temperature throughout the day to mimic natural sunlight patterns, promoting better sleep, alertness, and overall wellbeing.

**Import URL:**
```
https://raw.githubusercontent.com/leviemartin/HomeAssistant-Repo/main/blueprints/circadian_rhythm_lighting.yaml
```

### 2. Sun-Aware Circadian Rhythm Lighting (Netherlands Optimized) ⭐ NEW
Enhanced version that tracks actual sun position for locations with extreme seasonal daylight variations. Perfect for Netherlands where summer sun sets at 10pm and winter sun barely rises. Includes bedtime overrides to maintain family routines regardless of sun position.

**Import URL:**
```
https://raw.githubusercontent.com/leviemartin/HomeAssistant-Repo/main/blueprints/circadian_rhythm_lighting_sun_aware.yaml
```

[📖 Quick Start Guide](blueprints/QUICK_START.md) | [🔬 Scientific Details](blueprints/SUN_AWARE_UPGRADE_GUIDE.md) | [⚖️ Feature Comparison](blueprints/FEATURE_COMPARISON.md)

### 3. Intelligent Living Room Lighting (mmWave + Lux Aware) ⭐ v1.5
Advanced living room automation with Aqara FP2 mmWave presence detection, natural light awareness, and anti-flicker protection. Features sun-aware circadian rhythm integration, dynamic brightness scaling, optimized turn-off timing (day: 15/20/25 min, night: 5/10/15 min), scene cycling (cycle through 3 Philips Hue scenes). **CRITICAL FIX v1.5:** Fixed string vs boolean bug that prevented automation from executing any actions! **v1.4:** Added lux numeric_state trigger for immediate turn-off when bright.

**Import URL:**
```
https://raw.githubusercontent.com/leviemartin/HomeAssistant-Repo/main/blueprints/intelligent_living_room_mmwave_lux_aware.yaml
```

[📖 Quick Start](blueprints/INTELLIGENT_LIVING_ROOM_QUICK_START.md) | [📘 Complete Setup Guide](blueprints/INTELLIGENT_LIVING_ROOM_SETUP_GUIDE.md) | [🔬 Anti-Flicker Technical Guide](blueprints/ANTI_FLICKER_TECHNICAL_GUIDE.md) | [📡 FP2 Features Reference](blueprints/FP2_FEATURES_REFERENCE.md)

### 4. Adjacent Zone Motion Sensor with Circadian Lighting ⭐ v1.6
Motion-activated lighting for hallways, bathrooms, and stairs with GUARANTEED color consistency to the living room blueprint. Uses identical circadian formulas (1800K-5500K) with hardcoded Quick timing (5/10/15 min = 15 min total) optimized for quick-transition spaces. **CRITICAL FIX v1.6:** Adds lux numeric_state trigger so lights turn off when room becomes bright! **Finally works reliably in sunny conditions!**

**Import URL:**
```
https://raw.githubusercontent.com/leviemartin/HomeAssistant-Repo/main/blueprints/adjacent_zone_motion_circadian_lighting.yaml
```

[📖 Quick Start](blueprints/ADJACENT_ZONE_QUICK_START.md) | [📘 Complete Setup Guide](blueprints/ADJACENT_ZONE_SETUP_GUIDE.md) | [🎨 Color Consistency Guide](blueprints/COLOR_CONSISTENCY_GUIDE.md)

### 5. Child-Friendly Night Light Automation (Ages 1-5)
Research-based night light automation specifically designed for young children's sensitive sleep physiology. Ultra-low brightness and warm amber light minimize melatonin suppression while providing safe nighttime visibility.

**Import URL:**
```
https://raw.githubusercontent.com/leviemartin/HomeAssistant-Repo/main/blueprints/child_night_light.yaml
```

---

## Circadian Rhythm Lighting

### Features

- **6 Scientifically-Backed Circadian Phases** - From warm 2000K sunrise to cool 5500K midday
- **Motion-Activated** - Lights respond to motion sensors with configurable timeout
- **Pleasant Smooth Transitions** - Lights fade in over 1.5 seconds (not jarring instant-on)
- **Considerate Turn-Off** - Dim warning before turning off (inspired by Lutron Maestro)
  - Dims to 15% as gentle warning
  - Waits 15 seconds (time to re-trigger motion)
  - Fades off smoothly over 3 seconds
- **Lux Sensor Support** - Optional ambient light detection to prevent daytime activation
- **Brightness Control** - Optional automatic brightness adjustment (20% night to 100% midday)
- **Fully Customizable** - Adjust all time ranges and settings to match your schedule
- **Optimized for Philips Hue** - Uses mireds color temperature for maximum compatibility
- **User-Friendly** - Simple dropdown selectors, no YAML editing required

## Color Temperature Schedule

| Phase | Time | Kelvin | Effect |
|-------|------|--------|---------|
| **Dawn** | 5am-7am | 2000K | Gentle wake-up with warm sunrise glow |
| **Morning** | 7am-10am | 3000K | Comfortable start to the day |
| **Midday** | 10am-3pm | 5500K | Maximum alertness and productivity |
| **Afternoon** | 3pm-6pm | 4500K | Sustained energy levels |
| **Evening** | 6pm-9pm | 3000K | Wind-down begins |
| **Night** | 9pm-5am | 2200K | Sleep-friendly amber light |

*Based on peer-reviewed research on circadian lighting and blue light's effect on melatonin production*

## Quick Start

### Import via URL (Recommended)

1. In Home Assistant, go to **Settings** → **Automations & Scenes** → **Blueprints**
2. Click the **Import Blueprint** button
3. Paste this URL:
   ```
   https://raw.githubusercontent.com/leviemartin/HomeAssistant-Repo/main/blueprints/circadian_rhythm_lighting.yaml
   ```
4. Click **Preview** then **Import**

### Manual Installation

1. Download [`blueprints/circadian_rhythm_lighting.yaml`](blueprints/circadian_rhythm_lighting.yaml)
2. Copy to `/config/blueprints/automation/` in your Home Assistant installation
3. Restart Home Assistant or reload automations

## Configuration

Once imported, create a new automation:

1. Go to **Settings** → **Automations & Scenes** → **Create Automation**
2. Select **Use a blueprint** → **Circadian Rhythm Lighting with Motion Control**
3. Configure:
   - **Target Lights**: Select lights that support color temperature
   - **Motion Sensors**: Select one or more motion sensors
   - **Lux Sensor** (Optional): Prevent activation when room is already bright
   - **Time Ranges** (Optional): Customize phase timings
   - **Brightness Control** (Optional): Enable automatic brightness adjustment
4. Save and test!

## Compatible Devices

Works with any lights supporting `color_temp` or `kelvin` attributes:

- Philips Hue (White Ambiance, White & Color)
- IKEA TRADFRI (Tunable White)
- Sengled (Color Changing)
- LIFX (White to Warm, Color)
- And many more...

## Documentation

### Circadian Rhythm Lighting
- [**Installation Guide**](docs/INSTALLATION_GUIDE.md) - Step-by-step setup instructions
- [**Circadian Lighting README**](docs/CIRCADIAN_LIGHTING_README.md) - Scientific background and detailed documentation
- [**Quick Reference**](docs/QUICK_REFERENCE.md) - One-page cheat sheet
- [**Example Automations**](examples/EXAMPLE_AUTOMATION.yaml) - 6 real-world configuration examples
- [**Project Summary**](docs/PROJECT_SUMMARY.md) - Complete project overview

### Child Night Light
- [**Child Night Light Installation**](docs/CHILD_NIGHT_LIGHT_INSTALLATION.md) - Quick start guide with templates
- [**Child Night Light README**](docs/CHILD_NIGHT_LIGHT_README.md) - Comprehensive documentation with research citations
- [**Child Night Light Examples**](examples/child_night_light_examples.yaml) - 8 detailed real-world scenarios

## Examples

### Bedroom Setup
```yaml
# Motion-activated with lux sensor
# Only activates in dark conditions
# Sleep-optimized schedule
```

### Home Office
```yaml
# Extended midday phase for productivity
# Higher brightness during work hours
# Quick motion timeout
```

See [`examples/EXAMPLE_AUTOMATION.yaml`](examples/EXAMPLE_AUTOMATION.yaml) for complete configurations.

---

## Sun-Aware Circadian Rhythm Lighting ⭐ NEW

### Key Features

- **Real Sun Position Tracking** - Uses Home Assistant's sun.sun entity for authentic natural lighting
- **Netherlands Optimized** - Handles extreme seasonal variations (5:30am-10pm summer, 8:45am-4:30pm winter)
- **Bedtime Override System**:
  - Kids bedtime (~7pm) → Forces 2200K warm light even if sun still up
  - Parents bedtime (~10pm) → Forces 2200K regardless of season
- **Smart Time Boundaries** - Prevents 5:30am summer wake-ups, ensures 9am winter routines
- **Hybrid Operating Modes**:
  - **Hybrid** (recommended): Sun-aware during day, fixed schedule for evening
  - **Full Sun**: Always follows sun elevation
  - **Conservative**: Sun hints with strict boundaries
  - **Fixed-Time**: Original blueprint behavior (backward compatible)
- **Scientific Sun Elevation Mapping**:
  - Below -6°: 2200K (deep twilight, sleep-friendly)
  - -6° to 0°: 2200-3000K (civil twilight)
  - 0° to 10°: 3000-3500K (low sun, energizing morning)
  - 10° to 30°: 3500-4500K (rising/falling sun)
  - 30° to 50°: 4500-5500K (high sun, peak alertness)
  - Above 50°: 5500K (maximum alertness)
- **Advanced Lux Intelligence** - Summer evening detection (outdoor bright, indoor dark with curtains)

### Quick Start

See [📖 QUICK_START.md](blueprints/QUICK_START.md) for 5-minute setup guide.

**Recommended Configuration for Netherlands Families:**
```yaml
sun_tracking_enabled: true
sun_tracking_mode: "hybrid"
kids_bedtime: "19:00:00"  # 7pm
parents_bedtime: "22:00:00"  # 10pm
morning_earliest: "07:00:00"
evening_latest: "20:30:00"
```

### Documentation

- [📖 Quick Start Guide](blueprints/QUICK_START.md) - 5-minute setup
- [🔬 Scientific Details & Upgrade Guide](blueprints/SUN_AWARE_UPGRADE_GUIDE.md) - Research basis, edge cases
- [⚖️ Feature Comparison](blueprints/FEATURE_COMPARISON.md) - Original vs sun-aware
- [🔧 Technical Implementation](blueprints/CHANGES_SUMMARY.md) - Complete technical details

---

## Intelligent Living Room Lighting ⭐ v1.4

### Key Features

- **Anti-Flicker Hysteresis** - 150 lux OFF / 80 lux ON with 50 lux dead band prevents oscillation
- **Immediate Response** - No debounce wait, lights turn on/off instantly when thresholds crossed
- **Simplified Turn-On Logic** - Removed wait_template that could block activation
- **Aqara FP2 mmWave Integration**:
  - Approach detection (lights on when approaching)
  - Built-in lux sensor support
  - Future-ready for zones, target count, fall detection
- **Dynamic Brightness Scaling** - Inverse relationship with natural light (100% @ ≤50 lux → 40% @ 150 lux)
- **Sun-Aware Circadian Rhythm**:
  - 1800K-5500K based on sun elevation
  - 60-second continuous updates
  - 3-5 second smooth transitions
  - Adapted from sun-aware blueprint
- **Optimized Staged Turn-Off** (v1.3):
  - Day mode: 15/20/25 min (Stage 1 @ 15 min: 40% + 1800K, Stage 2 @ 20 min: 20% + 1800K, OFF @ 25 min)
  - Night mode: 5/10/15 min (Stage 1 @ 5 min: 40% + 1800K, Stage 2 @ 10 min: 20% + 1800K, OFF @ 15 min)
  - Perfect balance: responsive without being jarring
- **Easy Override System** - Toggle via button/switch/input_boolean to keep lights on regardless of lux
- **Scene Cycling** - Single button cycles through 3 Hue scenes (Movie → Reading → Party → Off)
  - Smart scene skipping (disabled scenes automatically skipped)
  - State persists across HA restarts
  - Optional presence timeout bypass
- **Philips Hue Compatible** - Color_temp mode (mireds) + Scene integration

### Quick Start

See [📖 Quick Start Guide](blueprints/INTELLIGENT_LIVING_ROOM_QUICK_START.md) for 5-minute setup.

**Recommended Configuration:**
```yaml
lux_sensor: sensor.aqara_fp2_illuminance
presence_sensor: binary_sensor.aqara_fp2_presence
approach_sensor: binary_sensor.aqara_fp2_approach  # Optional
override_entity: input_boolean.living_room_movie_mode
lux_turn_off_threshold: 150
lux_turn_on_threshold: 80
```

### Anti-Flicker Protection Explained

**The Problem:** Natural light constantly fluctuates from clouds, reflections, and gradual sunrise/sunset. Without protection, lights flicker on/off annoyingly.

**The Solution:**
- **Hysteresis**: 50 lux gap between ON (80) and OFF (150) creates "dead band"
- **Immediate Response**: Lights respond instantly when thresholds crossed (no wait timers)
- **Reliable Operation**: Dual thresholds prevent oscillation from brief lux spikes

Result: No flicker, instant response, reliable operation.

### Documentation

- [📖 Quick Start](blueprints/INTELLIGENT_LIVING_ROOM_QUICK_START.md) - 5-minute setup
- [📘 Complete Setup Guide](blueprints/INTELLIGENT_LIVING_ROOM_SETUP_GUIDE.md) - FP2 configuration, advanced features
- [🔬 Anti-Flicker Technical Guide](blueprints/ANTI_FLICKER_TECHNICAL_GUIDE.md) - Deep dive into hysteresis logic
- [📡 FP2 Features Reference](blueprints/FP2_FEATURES_REFERENCE.md) - Aqara FP2 capabilities

### Recent Enhancements (v1.5) ⭐ AUTOMATION FINALLY WORKING

- ✅ **CRITICAL FIX: String vs Boolean Bug** - Fixed override_active and scene_is_active returning strings instead of booleans
- ✅ **Automation Now Executes Actions** - Branch 4 (circadian_update) conditions now evaluate correctly
- ✅ **Proper Boolean Logic** - `not "false"` (FALSE) → `not false` (TRUE) - automation now responds to presence!
- ✅ **Root Cause Identified** - Variables used if/else returning string "false" instead of boolean false
- ✅ **Simplified Templates** - Using AND chain syntax for cleaner boolean returns

### Previous Enhancements (v1.4)

- ✅ **CRITICAL FIX: Lux Numeric State Trigger** - Automation now fires when lux exceeds threshold
- ✅ **Immediate Turn-Off When Bright** - Lights turn off as soon as room becomes bright (sunrise, curtains)
- ✅ **Independent of Presence Loop** - No longer relies only on 60-second loop checks
- ✅ **Respects Override/Scene** - Won't turn off if override or scene is active
- ✅ **Fixes Sunny Day Issue** - Lights no longer stay on all day in bright rooms

### Previous Enhancements (v1.3)

- ✅ **Removed Debounce Wait** - Lights turn on/off instantly when lux thresholds crossed
- ✅ **Fixed Turn-On Blocking** - Removed wait_template with continue_on_timeout: false
- ✅ **Optimized Turn-Off Timing** - Day: 15/20/25 min (was 30/40/45), Night: 5/10/15 min
- ✅ **Cleaner Configuration** - Removed 3 unused debounce inputs

### Previous Enhancements (v1.2)

- ✅ **CRITICAL FIX: Circadian Updates** - Colors now properly update every 60 seconds during occupancy
- ✅ **CRITICAL FIX: Brightness Updates** - Brightness now properly adjusts with changing lux
- ✅ **Night Mode Fast Shutdown** - Configurable 5/10/15 min turn-off after 22:00 (vs 15/20/25 min day mode)
- ✅ **Midnight Crossing Support** - Night mode properly handles 22:00-06:00 time windows
- ✅ **Bug Fix: Branch 4 Trigger** - Fixed broken time_pattern condition

### Previous Enhancements (v1.1)

- ✅ **Scene Cycling** - Cycle through 3 Philips Hue scenes with single button
- ✅ **State Persistence** - Scene state survives HA restarts via input_number helper
- ✅ **Smart Skipping** - Automatically skips disabled scenes in cycle

### Future Enhancements (v1.4+)

- Night Lights Mode (separate late-night settings)
- Adaptive Lighting Detection (pause on manual changes)
- Auto-Off Safety Timer (maximum on-time)
- Multi-zone support (different FP2 zones)
- Target count awareness (brighter with more people)

---

## Adjacent Zone Motion Sensor with Circadian Lighting ⭐ v1.6

### Key Features

- **100% Color Consistency** - Extracted IDENTICAL circadian formulas from living room blueprint
- **Hardcoded Lux Thresholds** - 150/80 lux (not configurable) for guaranteed matching
- **Hardcoded Quick Timing** - 5/10/15 minutes (simplified, no presets)
  - 5 min initial wait (motion must clear completely)
  - 5 min Stage 1 delay (40% brightness, 1800K warm)
  - 5 min Stage 2 delay (20% brightness, 1800K warm)
  - Total: 15 minutes from last motion to lights off
  - Optimized for hallways and bathrooms
- **Sun-Aware Circadian Rhythm**:
  - 1800K-5500K based on sun elevation
  - Identical mapping to living room
  - Set at light turn-on (not continuous updates)
- **Anti-Flicker Protection**:
  - Hysteresis: 150 lux OFF / 80 lux ON (50 lux dead band)
  - Debouncing: 2 min before ON
  - Prevents lights flickering during cloud cover
- **Configurable Brightness Controls**:
  - Dynamic brightness enable/disable toggle
  - Adjustable brightness curve at 5 lux breakpoints (50/75/100/125/150 lux)
  - Customizable stage warning brightness (defaults: 40%/20%)
  - Same functionality as living room blueprint
- **Staged Turn-Off Warnings**:
  - Stage 1: Configurable brightness (default 40%) + warm to 1800K
  - Stage 2: Configurable brightness (default 20%) + warm to 1800K
  - Stage 3: Turn off with 3-second fade
- **Manual Override** - Toggle via button/switch/input_boolean to keep lights on
- **CRITICAL FIX (v1.4)** ⚠️ TURN-ON NOW WORKS:
  - ✅ **v1.4 Fix**: Removed debounce wait that prevented lights from turning on
  - ✅ **Immediate Response**: Lights turn on instantly when motion detected (if lux < 80)
  - ✅ **No Timeouts**: Simple condition check can't fail or timeout
  - ✅ **Faster Turn-Off**: 15 minutes total (reduced from 25 min in v1.3)
- **Previous Fixes (v1.3)**:
  - ✅ **ROOT CAUSE FIXED**: v1.1/v1.2 used monitoring loop designed for mmWave sensors
  - ✅ **PIR Sensors Now Work**: wait_for_trigger ignores PIR 1-2 min hardware cooldown
  - ✅ **Simplified Code**: 24% reduction (861 → 655 lines)
  - ❌ v1.1/v1.2 bug: PIR cooldown caused infinite restart loop
  - ❌ v1.3 bug: Debounce wait prevented lights from turning on
- **Philips Hue Compatible** - Color_temp mode (mireds)
- **PIR Motion Sensor Compatible** ⭐ - Philips Hue, Aqara P1, generic PIR (also works with mmWave)

### Design Philosophy

This blueprint solves a critical UX problem: **adjacent zones must have identical light color** to avoid jarring transitions. When you walk from the living room (managed by the Intelligent Living Room blueprint) into a hallway (managed by this blueprint), both lights must show the exact same color temperature at the exact same time.

**How We Guarantee Consistency:**
- Extracted the EXACT sun elevation → Kelvin formula from living room blueprint
- Hardcoded lux thresholds (150/80) - users cannot change them
- Hardcoded brightness curve - identical scaling
- Hardcoded debounce times (120/300/600 sec)
- Result: **Living room at 3500K = Hallway at 3500K** (always)

**What's Different:**
- Shorter turn-off intervals (5/10/15 min vs 30/40/45 min in living room)
- Optimized for quick-transition spaces (not long-dwell spaces)
- Zone preset system for easy configuration

### Quick Start

See [📖 Quick Start Guide](blueprints/ADJACENT_ZONE_QUICK_START.md) for 5-minute setup.

**Recommended Configuration for Hallways:**
```yaml
zone_preset: "quick"  # 5/10/15 min intervals
lux_sensor: sensor.hallway_illuminance
motion_sensor: binary_sensor.hallway_motion
override_entity: input_boolean.hallway_override
```

### Why Color Consistency Matters

**Without Consistency:**
- Living room shows 5500K cool white (midday sun)
- Hallway shows 3000K warm white (different blueprint/settings)
- Result: Jarring color shift when transitioning between rooms

**With This Blueprint:**
- Living room shows 5500K at 2pm (sun at 37°)
- Hallway shows 5500K at 2pm (same formula, same sun position)
- Result: Seamless, natural transition

### Documentation

- [📖 Quick Start](blueprints/ADJACENT_ZONE_QUICK_START.md) - 5-minute setup
- [📘 Complete Setup Guide](blueprints/ADJACENT_ZONE_SETUP_GUIDE.md) - Zone configuration, advanced features
- [🎨 Color Consistency Guide](blueprints/COLOR_CONSISTENCY_GUIDE.md) - Technical details on color matching

### Hardcoded Timing (No Configuration Needed)

**Quick preset permanently configured for hallways and bathrooms:**

| Event | Timing | What Happens |
|-------|--------|--------------|
| **Motion Detected** | T+0 | Lights turn ON (100%, circadian color) |
| **Motion Clears** | T+0 to T+5 | Wait for new motion (or timeout) |
| **Stage 1 Warning** | T+5 | Dim to 40% + warm to 1800K |
| **Stage 2 Warning** | T+10 | Dim to 20% + warm to 1800K |
| **Lights OFF** | T+15 | Turn off with 3-second fade |

**If motion detected during any stage:** Automation restarts from T+0 (lights back to 100%)

### Recent Enhancements (v1.6) ⭐ LUX MONITORING FIXED

- ✅ **CRITICAL FIX: Lux Numeric State Trigger** - Automation now fires when lux exceeds threshold (150 lux hardcoded)
- ✅ **Import Error Fixed** - v1.5 had bug referencing non-existent input, v1.6 uses hardcoded value
- ✅ **Immediate Turn-Off When Bright** - Lights turn off as soon as room becomes bright (sunrise, curtains)
- ✅ **No More Guessing** - v1.4 only checked lux at motion trigger, never monitored afterward
- ✅ **Respects Override** - Won't turn off if manual override is active
- ✅ **Fixes Sunny Day Issue** - Lights no longer stay on all day in bright hallways/bathrooms

### Previous Enhancements (v1.4)

- ✅ **CRITICAL FIX**: Removed debounce wait that prevented lights from turning on
- ✅ **Immediate Response**: Lights now turn on instantly when motion detected (if lux < 80)
- ✅ **No Timeouts**: Simple condition check can't fail or timeout like v1.3
- ✅ **Faster Turn-Off**: 15 minutes total (reduced from 25 min in v1.3)
- ✅ **User-Requested Timing**: 5 min initial wait instead of 15 min
- ✅ **Actually Reliable**: Both turn-on AND turn-off work perfectly with PIR sensors

### Previous Enhancements (v1.3) - COMPLETE REWRITE

- ✅ **ROOT CAUSE IDENTIFIED**: v1.1/v1.2 used monitoring loop designed for mmWave sensors
- ✅ **PIR Cooldown Handling**: PIR sensors go "OFF" after 1-2 min (hardware behavior)
- ✅ **New Architecture**: wait_for_trigger pattern ignores PIR cooldown, watches for NEW triggers
- ✅ **Simplified Code**: Removed preset system (Very Quick/Medium/Custom)
- ✅ **Hardcoded Timing**: Quick preset only (5/10/15 min)
- ✅ **24% Smaller**: 861 lines → 655 lines
- ❌ **v1.3 Bug**: Debounce wait prevented lights from turning on reliably

### Previous Attempts (v1.1/v1.2) - FAILED

- ❌ **v1.1**: Stale variables + blocking condition → lights stayed on forever
- ❌ **v1.2**: Added variable recalculation → still failed due to PIR cooldown issue
- ❌ **Root Problem**: Monitoring loop pattern incompatible with PIR hardware behavior

---

## Child-Friendly Night Light (Ages 1-5)

### Key Features

- **Research-Backed Defaults**: Based on peer-reviewed studies of children's light sensitivity
- **Ultra-Low Brightness** (1-5%): Minimizes melatonin suppression in highly sensitive young children
- **Warm Amber Color** (2000K): Studies show minimal circadian impact even at higher intensities
- **3 Configurable Time Slots**: Nighttime + 2 optional nap periods (each can be enabled/disabled)
- **Motion-Activated Brightness Boost**: Temporarily increases brightness for parent checks, then returns to base
- **Smooth Transitions**: Gentle changes prevent startling sleeping children
- **Age-Specific Recommendations**: Detailed guidance for ages 1-2, 2-3, 3-4, and 4-5
- **Wide Compatibility**: Works with RGB lights (Third Reality, LIFX) and tunable white lights (Philips Hue, IKEA TRADFRI)

### Scientific Research Summary

This blueprint implements findings from multiple peer-reviewed studies:

**Key Research Finding**: Preschool-aged children experience 87.6% melatonin suppression from bright light and 77.5% suppression even from very dim light (5-40 lux). Melatonin remains suppressed for 50+ minutes after exposure ends.

- **Children are FAR more sensitive than adults**: Blue-enriched light (6200K) significantly suppresses melatonin in children but not adults
- **Amber/red light is safe**: Even 800 lux of amber light has minimal impact on melatonin production
- **Recommended maximum**: 1.5 lux for children's sleep environments
- **Blueprint default**: 3% brightness (~1-2 lux) with 2000K warm amber color

*Full research citations in the detailed documentation*

### Quick Start - Child Night Light

**Import URL:**
```
https://raw.githubusercontent.com/leviemartin/HomeAssistant-Repo/main/blueprints/child_night_light.yaml
```

1. Import blueprint (Settings → Automations & Scenes → Blueprints → Import Blueprint)
2. Create automation and configure:
   - **Night Lights**: Select 1-4 lights (must support color temperature)
   - **Nighttime Schedule**: Set bedtime and wake-up time
   - **Use default settings**: Already optimized based on research (3% brightness, 2000K)
   - **Optional**: Add motion sensor for automatic brightness boost during nighttime checks
3. Done! Research-backed safe lighting for your child

See the [Installation Guide](docs/CHILD_NIGHT_LIGHT_INSTALLATION.md) for detailed setup instructions and configuration templates.

---

## Scientific Background

### Circadian Rhythm Lighting
Based on research into:
- Natural sunlight variation throughout the day (2000K-6600K)
- Blue light's effect on melatonin suppression and alertness
- Melanopsin photopigment response to different wavelengths
- Circadian rhythm optimization for sleep quality

Color temperatures and timing are designed to support your body's natural circadian cycle while providing appropriate lighting for daily activities.

### Child Night Light
Based on peer-reviewed pediatric sleep research:
- Hartstein et al. (2022) - High sensitivity of melatonin suppression in preschool children
- Lee et al. (2018) - Blue-enriched light effects in children vs. adults
- Akacem et al. (2022) - Light exposure timing and duration effects
- Multiple studies on amber/red wavelength safety for children's sleep

All defaults are set to research-backed safe levels for ages 1-5.

## Customization

All time ranges and settings can be customized:
- Adjust phase timings for night shift workers
- Modify color temperatures for personal preference
- Change motion timeout duration
- Set custom lux thresholds
- Enable/disable brightness adjustment

## Contributing

Found a bug or have a suggestion? Please [open an issue](../../issues) or submit a pull request!

## License

MIT License - Feel free to use, modify, and share!

## Acknowledgments

Created with research-backed color temperature values to promote healthy circadian rhythms and improve quality of life through better lighting.

---

**Need help?** Check the [Installation Guide](docs/INSTALLATION_GUIDE.md) or [open an issue](../../issues)!
