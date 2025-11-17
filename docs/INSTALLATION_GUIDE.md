# Quick Installation Guide - Circadian Rhythm Lighting Blueprint

## Step 1: Install the Blueprint

### Option A: File Upload (Recommended for beginners)

1. Download `circadian_rhythm_lighting.yaml` to your computer
2. In Home Assistant, go to **File Editor** addon (install if needed)
3. Navigate to `/config/blueprints/automation/`
4. Create a new folder called `circadian_lighting` (optional but organized)
5. Upload the `circadian_rhythm_lighting.yaml` file
6. Go to **Settings** → **Automations & Scenes** → **Blueprints**
7. You should see "Circadian Rhythm Lighting with Motion Control" listed

### Option B: Manual File Copy

1. Copy `circadian_rhythm_lighting.yaml` to your Home Assistant server:
   ```bash
   # If using SSH/Terminal
   scp circadian_rhythm_lighting.yaml homeassistant@your-ha-ip:/config/blueprints/automation/
   ```

2. Or if you have direct file access:
   ```
   /config/blueprints/automation/circadian_rhythm_lighting.yaml
   ```

3. Restart Home Assistant or reload automations

## Step 2: Create Your First Automation

1. Go to **Settings** → **Automations & Scenes**
2. Click the **Create Automation** button (bottom right)
3. Select **Use a blueprint**
4. Find and click **Circadian Rhythm Lighting with Motion Control**
5. Fill in the configuration (see below)
6. Click **Save**

## Step 3: Basic Configuration

### Minimum Required Settings

```
Name: Bedroom Circadian Lighting
(or whatever you want to call it)

Target Lights:
└─ Select your bedroom lights
   (e.g., light.bedroom_ceiling, light.bedside_lamp)

Motion Sensors:
└─ Select your motion sensor(s)
   (e.g., binary_sensor.bedroom_motion)

(Leave all other settings at defaults to start)
```

### Click SAVE and you're done!

## Step 4: Test It

1. Turn off the room lights manually
2. Walk into the room to trigger motion
3. Lights should turn on with appropriate color temperature:
   - Morning: Warm white (3000K)
   - Midday: Cool daylight (5500K)
   - Evening: Warm white (3000K)
   - Night: Extra warm amber (2200K)
4. Leave the room for 5 minutes
5. Lights should automatically turn off

## Step 5: Optional Enhancements

Once basic setup works, consider adding:

### Add Lux Sensor (prevents daytime activation)

1. Edit your automation
2. Scroll to **Lux Sensor (Optional)**
3. Select your illuminance sensor (e.g., `sensor.bedroom_illuminance`)
4. Set **Lux Threshold** to 50-100 lux
5. Save

### Enable Brightness Adjustment

1. Edit your automation
2. Find **Adjust Brightness Throughout Day**
3. Toggle it ON
4. Save

Now lights will also dim/brighten based on time:
- Night: 20% brightness
- Morning: 70% brightness
- Midday: 100% brightness
- Evening: 50% brightness

### Customize Time Ranges

1. Edit your automation
2. Scroll to the time settings:
   - Dawn Phase Start Time
   - Morning Phase Start Time
   - Midday Phase Start Time
   - Afternoon Phase Start Time
   - Evening Phase Start Time
   - Night Phase Start Time
3. Adjust to match your schedule
4. Save

## Verification Checklist

- [ ] Blueprint appears in Home Assistant blueprints list
- [ ] Created automation from blueprint successfully
- [ ] Selected compatible lights (tunable white/color temperature capable)
- [ ] Selected motion sensor(s)
- [ ] Tested motion detection triggers lights
- [ ] Verified color temperature changes based on time of day
- [ ] (Optional) Lux sensor prevents daytime activation
- [ ] (Optional) Brightness adjusts throughout day
- [ ] Lights turn off after timeout period

## Troubleshooting Quick Fixes

### Blueprint doesn't appear

- Restart Home Assistant
- Check file is in `/config/blueprints/automation/`
- Check YAML file has no syntax errors

### Lights don't change color

- Verify lights support color temperature (not just RGB)
- Check light capabilities in Developer Tools → States
- Look for `color_temp` or `kelvin` in supported features

### Lights don't turn off

- Check motion sensor returns to "off" state
- Verify timeout duration setting
- Check Home Assistant logs for errors

### Wrong color temperature

- Verify current time is in expected phase
- Check phase start times in automation settings
- Some lights have limited color temp ranges (2700-6500K typical)

## Next Steps

1. **Create multiple automations** for different rooms
2. **Customize schedules** for each room's usage pattern
3. **Fine-tune lux thresholds** for optimal daytime behavior
4. **Adjust timeout durations** per room activity level
5. **Experiment with brightness levels** if enabled

## Example Configurations

### Bedroom (Sleep-Focused)
- Extra warm night settings (2200K)
- Dim morning wake-up (40%)
- Lux sensor to prevent daytime activation
- 5-minute timeout

### Home Office (Productivity-Focused)
- Bright cool midday (5500K, 100%)
- Extended afternoon phase for late workers
- No lux sensor (want light even during day)
- 10-minute timeout

### Bathroom (Quick Use)
- Standard circadian schedule
- 3-minute timeout (short visits)
- High lux threshold (often used during day)

### Living Room (Relaxation-Focused)
- Extended warm evening phase
- Moderate brightness levels
- 10-minute timeout (longer activities)

---

**Need Help?** Check the full README.md for detailed troubleshooting, customization ideas, and technical documentation.

**Working perfectly?** Consider creating automations for all your rooms to enjoy full circadian lighting throughout your home!
