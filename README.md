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

### 3. Intelligent Living Room Lighting (mmWave + Lux Aware) ⭐ NEW v1.1
Advanced living room automation with Aqara FP2 mmWave presence detection, natural light awareness, and anti-flicker protection. Features sun-aware circadian rhythm integration, dynamic brightness scaling, staged turn-off warnings, and **NEW: Scene Cycling** (cycle through 3 Philips Hue scenes with one button).

**Import URL:**
```
https://raw.githubusercontent.com/leviemartin/HomeAssistant-Repo/main/blueprints/intelligent_living_room_mmwave_lux_aware.yaml
```

[📖 Quick Start](blueprints/INTELLIGENT_LIVING_ROOM_QUICK_START.md) | [📘 Complete Setup Guide](blueprints/INTELLIGENT_LIVING_ROOM_SETUP_GUIDE.md) | [🔬 Anti-Flicker Technical Guide](blueprints/ANTI_FLICKER_TECHNICAL_GUIDE.md) | [📡 FP2 Features Reference](blueprints/FP2_FEATURES_REFERENCE.md)

### 4. Child-Friendly Night Light Automation (Ages 1-5)
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

## Intelligent Living Room Lighting ⭐ NEW

### Key Features

- **Anti-Flicker Hysteresis** - 150 lux OFF / 80 lux ON with 50 lux dead band prevents oscillation
- **Time-Based Debouncing** - 5 min stable before OFF (cloud protection), 2 min before ON
- **Extended Sunrise/Sunset Debounce** - 10 minutes during transitions (1 hour window)
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
- **Staged Turn-Off Warnings**:
  - 30 min: Dim to 40% + warm to 1800K (clear warning)
  - 40 min: Dim to 20% + warm to 1800K (strong warning)
  - 45 min: Turn off with 3-second fade
- **Easy Override System** - Toggle via button/switch/input_boolean to keep lights on regardless of lux
- **Scene Cycling (NEW v1.1)** - Single button cycles through 3 Hue scenes (Movie → Reading → Party → Off)
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
- **Debouncing**: Light must be stable 5 minutes before turning off, 2 minutes before turning on
- **Extended transitions**: 10-minute debounce during sunrise/sunset prevents oscillation

Result: No flicker, smooth operation, natural feel.

### Documentation

- [📖 Quick Start](blueprints/INTELLIGENT_LIVING_ROOM_QUICK_START.md) - 5-minute setup
- [📘 Complete Setup Guide](blueprints/INTELLIGENT_LIVING_ROOM_SETUP_GUIDE.md) - FP2 configuration, advanced features
- [🔬 Anti-Flicker Technical Guide](blueprints/ANTI_FLICKER_TECHNICAL_GUIDE.md) - Deep dive into hysteresis logic
- [📡 FP2 Features Reference](blueprints/FP2_FEATURES_REFERENCE.md) - Aqara FP2 capabilities

### Recent Enhancements (v1.1)

- ✅ **Scene Cycling** - Cycle through 3 Philips Hue scenes with single button
- ✅ **State Persistence** - Scene state survives HA restarts via input_number helper
- ✅ **Smart Skipping** - Automatically skips disabled scenes in cycle

### Future Enhancements (v1.2+)

- Night Lights Mode (separate late-night settings)
- Adaptive Lighting Detection (pause on manual changes)
- Auto-Off Safety Timer (maximum on-time)
- Multi-zone support (different FP2 zones)
- Target count awareness (brighter with more people)

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
