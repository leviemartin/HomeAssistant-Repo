# Circadian Rhythm Lighting Blueprint for Home Assistant

A scientifically-backed Home Assistant blueprint that automatically adjusts your lights' color temperature throughout the day to mimic natural sunlight patterns, promoting better sleep, alertness, and overall wellbeing.

## Features

- **6 Scientifically-Backed Circadian Phases** - From warm 2000K sunrise to cool 5500K midday
- **Motion-Activated** - Lights respond to motion sensors with configurable timeout
- **Lux Sensor Support** - Optional ambient light detection to prevent daytime activation
- **Brightness Control** - Optional automatic brightness adjustment (20% night to 100% midday)
- **Fully Customizable** - Adjust all time ranges and settings to match your schedule
- **Maximum Compatibility** - Supports both Kelvin and Mireds color temperature formats
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

- [**Installation Guide**](docs/INSTALLATION_GUIDE.md) - Step-by-step setup instructions
- [**Circadian Lighting README**](docs/CIRCADIAN_LIGHTING_README.md) - Scientific background and detailed documentation
- [**Quick Reference**](docs/QUICK_REFERENCE.md) - One-page cheat sheet
- [**Example Automations**](examples/EXAMPLE_AUTOMATION.yaml) - 6 real-world configuration examples
- [**Project Summary**](docs/PROJECT_SUMMARY.md) - Complete project overview

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

## Scientific Background

This blueprint is based on research into:
- Natural sunlight variation throughout the day (2000K-6600K)
- Blue light's effect on melatonin suppression and alertness
- Melanopsin photopigment response to different wavelengths
- Circadian rhythm optimization for sleep quality

Color temperatures and timing are designed to support your body's natural circadian cycle while providing appropriate lighting for daily activities.

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
