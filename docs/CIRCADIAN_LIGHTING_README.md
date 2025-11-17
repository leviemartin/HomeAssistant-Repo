# Circadian Rhythm Lighting Blueprint

A scientifically-backed Home Assistant blueprint that automatically adjusts light color temperature throughout the day to support your natural circadian rhythm. Motion sensors trigger lights to turn on with the perfect color temperature for the current time, helping you wake up naturally, stay alert during the day, and wind down in the evening.

## Features

- **6 Circadian Phases**: Dawn, Morning, Midday, Afternoon, Evening, and Night
- **Scientifically-Backed Color Temperatures**: Based on research about circadian rhythm and light exposure
- **Motion-Activated**: Lights turn on automatically when motion is detected
- **Optional Brightness Control**: Can adjust both color temperature AND brightness throughout the day
- **Lux Sensor Integration**: Prevent lights from turning on when there's sufficient natural light
- **Fully Customizable**: Adjust time ranges for each phase to match your schedule
- **Maximum Compatibility**: Works with lights supporting either Kelvin or Mireds color temperature

## Scientific Background

This blueprint implements color temperature ranges based on peer-reviewed research on circadian lighting:

- **Blue-enriched light (cool, 5000K+)** during the day suppresses melatonin and promotes alertness
- **Warm light (2000-3000K)** in the morning and evening reduces circadian disruption
- **Extra warm/amber light (2200K)** at night minimizes impact on sleep quality
- Natural sunlight varies from 2000K at sunrise to 6600K at noon, then back to warm tones

## Color Temperature Schedule

| Phase | Time Range (Default) | Color Temp | Description |
|-------|---------------------|------------|-------------|
| **Dawn** | 5:00 AM - 7:00 AM | 2000K | Soft warm light, gentle wake-up |
| **Morning** | 7:00 AM - 10:00 AM | 3000K | Warm white, comfortable morning light |
| **Midday** | 10:00 AM - 3:00 PM | 5500K | Cool daylight, maximum alertness |
| **Afternoon** | 3:00 PM - 6:00 PM | 4500K | Neutral white, sustained energy |
| **Evening** | 6:00 PM - 9:00 PM | 3000K | Warm white, beginning wind-down |
| **Night** | 9:00 PM - 5:00 AM | 2200K | Extra warm/amber, minimal disruption |

## Brightness Levels (Optional)

When brightness adjustment is enabled:

| Phase | Brightness |
|-------|-----------|
| Night | 20% |
| Dawn | 40% |
| Morning | 70% |
| Midday | 100% |
| Afternoon | 80% |
| Evening | 50% |

## Installation

### Method 1: Direct Import (Recommended)

1. In Home Assistant, navigate to **Settings** → **Automations & Scenes** → **Blueprints**
2. Click the **Import Blueprint** button
3. Enter the URL to the blueprint YAML file
4. Click **Preview** and then **Import**

### Method 2: Manual Installation

1. Copy the `circadian_rhythm_lighting.yaml` file to your Home Assistant blueprints folder:
   ```
   /config/blueprints/automation/circadian_rhythm_lighting.yaml
   ```
2. Restart Home Assistant or reload automations
3. The blueprint will appear in **Settings** → **Automations & Scenes** → **Blueprints**

## Configuration

### Required Inputs

1. **Target Lights**: Select the lights that should follow circadian rhythm
   - Must support color temperature adjustment (tunable white or white ambiance bulbs)
   - Can select individual lights, groups, or use device/area selectors

2. **Motion Sensors**: Select one or more motion sensors that will trigger the lights
   - Any sensor detecting motion will activate the lights
   - Lights turn on with appropriate color temperature for current time

### Optional Inputs

3. **Light Timeout Duration** (default: 300 seconds / 5 minutes)
   - How long lights stay on after motion stops

4. **Adjust Brightness Throughout Day** (default: disabled)
   - Enable to also adjust brightness based on time of day
   - Disable to only adjust color temperature

5. **Lux Sensor** (optional)
   - Select an illuminance sensor
   - When configured, lights only activate if ambient light is below threshold

6. **Lux Threshold** (default: 100 lux)
   - Lights only turn on if lux sensor reads below this value
   - Only applies when lux sensor is configured

### Time Customization

All phase start times are fully customizable:

- **Dawn Phase Start Time** (default: 5:00 AM)
- **Morning Phase Start Time** (default: 7:00 AM)
- **Midday Phase Start Time** (default: 10:00 AM)
- **Afternoon Phase Start Time** (default: 3:00 PM)
- **Evening Phase Start Time** (default: 6:00 PM)
- **Night Phase Start Time** (default: 9:00 PM)

## Usage Example

### Basic Setup

1. Create a new automation from the blueprint
2. Select your bedroom lights as "Target Lights"
3. Select your bedroom motion sensor as "Motion Sensors"
4. Leave other settings at defaults
5. Save the automation

Now when you walk into your bedroom:
- At 6:00 AM: Lights turn on at warm 2000K (gentle dawn)
- At 11:00 AM: Lights turn on at cool 5500K (energizing midday)
- At 8:00 PM: Lights turn on at warm 3000K (relaxing evening)
- At 10:00 PM: Lights turn on at extra warm 2200K (sleep-friendly)

### Advanced Setup with Lux Sensor

1. Create automation from blueprint
2. Select living room lights
3. Select motion sensors
4. **Enable** "Adjust Brightness Throughout Day"
5. Select your illuminance sensor as "Lux Sensor"
6. Set "Lux Threshold" to 50 lux
7. Customize time ranges to match your schedule

Result: Lights only activate when room is dark, with both color temperature and brightness optimized for circadian rhythm.

## Requirements

### Compatible Lights

- Tunable white bulbs (e.g., Philips Hue White Ambiance)
- White spectrum bulbs (e.g., IKEA TRADFRI tunable)
- Any light supporting `color_temp` or `kelvin` attributes

**Not compatible with:**
- RGB-only lights without white spectrum
- Fixed color temperature lights
- Simple on/off lights

### Required Hardware

- At least one motion sensor (binary_sensor with device_class: motion)
- Compatible smart lights (see above)

### Optional Hardware

- Illuminance/Lux sensor for ambient light detection

## Troubleshooting

### Lights don't change color temperature

**Issue**: Lights turn on but stay at default color temperature

**Solutions**:
- Verify your lights support color temperature adjustment (check device specifications)
- Some lights require being turned on first before accepting color temperature changes
- Try manually setting color temperature through Home Assistant to confirm capability

### Lights turn on during daytime when bright

**Issue**: Lights activate even when room has natural daylight

**Solution**:
- Configure a lux sensor in the blueprint settings
- Set appropriate lux threshold (typically 50-100 lux)
- Ensure lux sensor is placed correctly (not in shadows or direct sunlight)

### Motion timeout too short/long

**Issue**: Lights turn off while still in room, or stay on too long after leaving

**Solution**:
- Adjust "Light Timeout Duration" setting
- For high-traffic areas: 3-5 minutes (180-300 seconds)
- For low-traffic areas: 5-10 minutes (300-600 seconds)
- For bathrooms: 10-15 minutes (600-900 seconds)

### Colors don't match expected values

**Issue**: Color temperatures seem different than documented values

**Solution**:
- Some lights have limited color temperature ranges (e.g., 2700K-6500K)
- Blueprint will command the correct value, but lights will use closest supported value
- Check your light's specifications for its actual color temperature range

## Customization Ideas

### Night Shift Workers

Adjust the time ranges to match your schedule:
- Dawn: 5:00 PM (when you wake up)
- Morning: 7:00 PM
- Midday: 10:00 PM - 3:00 AM (your "daytime")
- Evening: 6:00 AM
- Night: 9:00 AM (when you sleep)

### Early Risers

- Dawn: 4:00 AM
- Morning: 6:00 AM
- Adjust other phases earlier to match

### Night Owls

- Extend evening phase to 11:00 PM
- Night phase: 11:00 PM - 7:00 AM
- Dawn phase: 7:00 AM

### Different Rooms, Different Schedules

Create multiple automations from the same blueprint:
- **Bedroom**: Full circadian rhythm with dim nighttime settings
- **Office**: Bright cool light during work hours, warm in evening
- **Living Room**: More moderate settings, longer evening phase

## Technical Details

### Automation Mode

The blueprint uses **restart mode**, which means:
- If motion is detected again while lights are on, the timeout timer resets
- This prevents lights from turning off while you're still active in the room
- Only one instance runs at a time (new motion events restart the automation)

### Transition Behavior

- **Color temperature changes**: Instant (0 second transition)
  - Provides immediate feedback when entering room
  - Color temperature represents current time phase
- **Turn off transition**: 2 seconds
  - Gentle fade to off prevents jarring experience

### Variable Calculations

The blueprint calculates:
1. Current time in seconds since midnight
2. Phase boundary times in seconds
3. Appropriate color temperature for current phase
4. Both Kelvin and Mireds values (for maximum compatibility)
5. Brightness percentage (when enabled)

### Edge Case Handling

- **Midnight wraparound**: Night phase properly handles times before dawn
- **Optional lux sensor**: Safe handling when no sensor is configured
- **Brightness disabled**: Template returns empty string, HA ignores the attribute
- **Multiple motion sensors**: Any sensor triggering will activate lights

## Research References

This blueprint is based on scientific research about circadian lighting:

- Natural sunlight color temperature variation (2000K at sunrise to 6600K at noon)
- Melanopsin response to blue light wavelengths (~490nm)
- Melatonin suppression from blue-enriched cool light (5000K+)
- Sleep quality improvement with warm evening light (2200-3000K)
- Alertness and cognitive performance enhancement with daylight tones (5000-6500K)

## Version History

### v1.0.0 (2025-11-17)
- Initial release
- 6-phase circadian rhythm support
- Motion sensor triggers
- Optional lux sensor integration
- Optional brightness adjustment
- Fully customizable time ranges
- Kelvin and Mireds compatibility

## Support & Feedback

For issues, questions, or suggestions, please:
1. Check the troubleshooting section above
2. Review Home Assistant logs for error messages
3. Verify your lights support color temperature adjustment
4. Test with a single light first before expanding to groups

## License

This blueprint is provided as-is for personal use in Home Assistant installations. Feel free to modify and adapt to your needs.

---

**Created by**: Home Assistant Blueprint Architect
**Last Updated**: November 17, 2025
**Home Assistant Version**: 2024.11+
