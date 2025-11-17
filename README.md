# Home Assistant Blueprints - Research-Backed Lighting Automation

Scientifically-backed Home Assistant blueprints for intelligent lighting automation. Each blueprint is built on peer-reviewed research to promote better sleep, health, and wellbeing through optimized light control.

## Available Blueprints

### 1. Circadian Rhythm Lighting with Motion Control
Automatically adjusts light color temperature throughout the day to mimic natural sunlight patterns, promoting better sleep, alertness, and overall wellbeing.

### 2. Child-Friendly Night Light Automation (Ages 1-5)
Research-based night light automation specifically designed for young children's sensitive sleep physiology. Ultra-low brightness and warm amber light minimize melatonin suppression while providing safe nighttime visibility.

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

## Child-Friendly Night Light (Ages 1-5)

### Key Features

- **Research-Backed Defaults**: Based on peer-reviewed studies of children's light sensitivity
- **Ultra-Low Brightness** (1-5%): Minimizes melatonin suppression in highly sensitive young children
- **Warm Amber Color** (2000K): Studies show minimal circadian impact even at higher intensities
- **3 Configurable Time Slots**: Nighttime + 2 optional nap periods (each can be enabled/disabled)
- **Motion-Activated Brightness Boost**: Temporarily increases brightness for parent checks, then returns to base
- **Smooth Transitions**: Gentle changes prevent startling sleeping children
- **Age-Specific Recommendations**: Detailed guidance for ages 1-2, 2-3, 3-4, and 4-5

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
