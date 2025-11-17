# Intelligent Living Room Blueprint v1.0 - Quick Start Guide

**Get up and running in 5 minutes**

This guide will help you configure the Intelligent Living Room blueprint with minimal fuss. For detailed configuration and advanced features, see the [Complete Setup Guide](./INTELLIGENT_LIVING_ROOM_SETUP_GUIDE.md).

## Prerequisites Checklist

Before you start, make sure you have:

- [ ] **Presence sensor** - Aqara FP2, mmWave, or PIR sensor (integrated in Home Assistant)
- [ ] **Lux sensor** - FP2's built-in lux sensor or separate illuminance sensor
- [ ] **Tunable white lights** - Philips Hue, IKEA Tradfri, or similar (must support color_temp)
- [ ] **Optional: Override trigger** - Button, switch, or input_boolean helper (for movie mode)

## Step 1: Install the Blueprint (30 seconds)

### Option A: Import via URL
1. Open Home Assistant
2. Navigate to **Settings** → **Automations & Scenes** → **Blueprints**
3. Click **Import Blueprint** (bottom right)
4. Paste this URL:
   ```
   https://github.com/martinlevie/HomeAssistant-Repo/blueprints/intelligent_living_room_mmwave_lux_aware.yaml
   ```
5. Click **Preview** → **Import**

### Option B: Manual Copy
1. Copy the blueprint YAML file to: `/config/blueprints/automation/`
2. Restart Home Assistant (or reload automations)

## Step 2: Create the Automation (2 minutes)

1. Navigate to **Settings** → **Automations & Scenes** → **Blueprints**
2. Find "Intelligent Living Room - mmWave + Lux-Aware v1.0"
3. Click **Create Automation**
4. Fill in the **required** fields:

### Required Inputs

| Field | What to Select | Example |
|-------|---------------|---------|
| **Target Lights** | Your living room lights | `light.living_room_ceiling` |
| **Primary Presence Sensor** | FP2 or mmWave sensor | `binary_sensor.fp2_living_room_presence` |
| **Lux/Illuminance Sensor** | FP2 lux or separate sensor | `sensor.fp2_living_room_light_sensor` |

### Leave Defaults For Now

All other settings have sensible defaults that work out-of-the-box:
- Lux thresholds: 80 lux (ON) / 150 lux (OFF)
- Debounce: 2 min (ON) / 5 min (OFF)
- Dynamic brightness: ENABLED
- Circadian rhythm: ENABLED
- Staged turn-off: 30/40/45 minutes

You can customize these later!

## Step 3: Save and Test (2 minutes)

1. Click **Save** (bottom right)
2. Name your automation: `Living Room - Intelligent Lighting`
3. Test it:
   - **Walk into room (low light)** → Lights turn on
   - **Open curtains (bright sunlight)** → Lights dim or turn off (after debounce)
   - **Walk out of room** → Staged turn-off begins (30/40/45 min warnings)

## What Just Happened?

You now have a living room that:
- ✅ Turns on lights when you enter (if room is dark enough)
- ✅ Automatically dims lights as natural light increases
- ✅ Won't flicker on/off during cloudy days (hysteresis + debounce)
- ✅ Adjusts color temperature throughout the day (warmer at night, cooler during day)
- ✅ Gives you progressive warnings before turning off (30/40/45 min stages)
- ✅ Extended debounce during sunrise/sunset to prevent oscillation

## Next Steps (Optional)

### Add Manual Override (Movie Mode)
1. Create an input_boolean helper:
   - **Settings** → **Devices & Services** → **Helpers** → **Create Helper**
   - Choose **Toggle** → Name: `Living Room Override`
2. Return to your automation → **Edit**
3. Enable **Manual Override System** → Set to **ON**
4. Select **Override Trigger Entity** → Choose `input_boolean.living_room_override`
5. Save

Now you can toggle movie mode from your dashboard! When active, lights stay on regardless of lux level.

### Add Approach Detection (FP2 Only)
If you have an Aqara FP2:
1. Configure approach detection in the Aqara app (see FP2_FEATURES_REFERENCE.md)
2. Return to your automation → **Edit**
3. Set **Approach Detection Sensor** → Choose `binary_sensor.fp2_living_room_approach`
4. Save

Now lights turn on instantly when you walk toward the room!

### Fine-Tune Lux Thresholds
If lights turn on/off too early or too late:
1. Check your current lux reading:
   - **Developer Tools** → **States** → Search for your lux sensor
   - Note the lux value in different lighting conditions
2. Adjust thresholds:
   - **Morning (curtains closed)**: ~40-60 lux
   - **Afternoon (curtains open, cloudy)**: ~100-150 lux
   - **Afternoon (curtains open, sunny)**: ~300-500 lux
3. Set **Turn-ON Threshold** below your "too dark" point
4. Set **Turn-OFF Threshold** above your "bright enough" point
5. Maintain ~70 lux gap between thresholds (prevents flicker)

## Troubleshooting

### Lights flicker on/off during cloudy days
**Solution**: Increase debounce timers
- Turn-OFF Debounce: Increase to 10 minutes (600 sec)
- Turn-ON Debounce: Increase to 5 minutes (300 sec)

### Lights turn on when room is already bright
**Solution**: Lower the Turn-ON threshold
- Current: 80 lux
- Try: 50-60 lux

### Lights turn off too quickly when curtains open
**Solution**: Increase Turn-OFF threshold or debounce
- Turn-OFF Threshold: Increase to 200-250 lux
- Turn-OFF Debounce: Increase to 10 minutes

### Lights don't turn off during sunrise/sunset
**Solution**: This is intentional (extended debounce)
- During dawn/dusk: 10-minute debounce (prevents oscillation)
- Normal times: 5-minute debounce
- If you want faster response, reduce "Sunrise/Sunset Extended Debounce"

### Staged turn-off happens too quickly
**Solution**: Increase stage delays
- Stage 1 (First Warning): Increase from 30 to 45 minutes
- Stage 2 (Second Warning): Increase from 40 to 55 minutes
- Stage 3 (Complete Off): Increase from 45 to 60 minutes

## Advanced Features (For Later)

Once you're comfortable with the basics, explore:
- **Dynamic Brightness Curve**: Customize brightness at specific lux levels
- **Circadian Rhythm Tuning**: Adjust update frequency and transition smoothness
- **Staged Turn-Off Customization**: Change warning brightness levels
- **Physical Button Override**: Use Aqara wireless button or Hue dimmer

See `INTELLIGENT_LIVING_ROOM_SETUP_GUIDE.md` for complete documentation.

## Need Help?

- **Technical Details**: Read `ANTI_FLICKER_TECHNICAL_GUIDE.md`
- **FP2 Configuration**: Read `FP2_FEATURES_REFERENCE.md`
- **Full Setup Guide**: Read `INTELLIGENT_LIVING_ROOM_SETUP_GUIDE.md`
- **Issues**: Open a GitHub issue with automation logs

## Pro Tips

1. **Lux sensor placement**: Place near window but not in direct sunlight (represents room's overall brightness)
2. **Hysteresis gap**: Keep 50-100 lux between ON and OFF thresholds (prevents flickering)
3. **Debounce tuning**: Longer debounce = more stable, shorter = more responsive
4. **Override for parties**: Create a scene that activates override + sets specific brightness/color
5. **Test during cloudy days**: That's when anti-flicker is most important!

---

**You're all set!** Your living room now has professional-grade intelligent lighting. Enjoy the seamless automation!
