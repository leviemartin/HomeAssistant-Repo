# Adjacent Zone Motion - Circadian Lighting v1.0 - Quick Start Guide

**Get up and running in 5 minutes**

This guide will help you configure motion-activated circadian lighting for hallways, bathrooms, stairs, and other adjacent zones with PERFECT color consistency to your living room. For detailed configuration and advanced features, see the [Complete Setup Guide](./ADJACENT_ZONE_SETUP_GUIDE.md).

## Prerequisites Checklist

Before you start, make sure you have:

- [ ] **Motion sensor** - PIR, mmWave, or any occupancy sensor (integrated in Home Assistant)
- [ ] **Lux sensor** - Built-in motion sensor lux or separate illuminance sensor
- [ ] **Tunable white lights** - Philips Hue, IKEA Tradfri, or similar (must support color_temp)
- [ ] **Optional: Override trigger** - Button, switch, or input_boolean helper (for extended use)

## Step 1: Install the Blueprint (30 seconds)

### Option A: Import via URL
1. Open Home Assistant
2. Navigate to **Settings** → **Automations & Scenes** → **Blueprints**
3. Click **Import Blueprint** (bottom right)
4. Paste this URL:
   ```
   https://github.com/martinlevie/HomeAssistant-Repo/blueprints/adjacent_zone_motion_circadian_lighting.yaml
   ```
5. Click **Preview** → **Import**

### Option B: Manual Copy
1. Copy the blueprint YAML file to: `/config/blueprints/automation/`
2. Restart Home Assistant (or reload automations)

## Step 2: Create the Automation (2 minutes)

1. Navigate to **Settings** → **Automations & Scenes** → **Blueprints**
2. Find "Adjacent Zone Motion - Circadian Lighting v1.0"
3. Click **Create Automation**
4. Fill in the **required** fields:

### Required Inputs

| Field | What to Select | Example |
|-------|---------------|---------|
| **Target Lights** | Your zone's lights | `light.hallway_ceiling` |
| **Motion Sensor** | PIR or mmWave motion sensor | `binary_sensor.hallway_motion` |
| **Lux/Illuminance Sensor** | Motion sensor lux or separate | `sensor.hallway_illuminance` |
| **Zone Type Preset** | Select zone type | `Quick (5/10/15 min) - Hallways, Bathrooms` |

### Leave Defaults For Now

All other settings have sensible defaults that work out-of-the-box:
- Lux thresholds: **80 lux (ON) / 150 lux (OFF)** - HARDCODED for consistency
- Debounce: **2 min (ON) / 5 min (OFF)** - HARDCODED for consistency
- Circadian rhythm: **ENABLED** - matches living room color
- Staged turn-off: **ENABLED** - 40%/20% dimming (matches living room)

You can customize timing presets and enable/disable features, but lux thresholds are LOCKED for color consistency!

## Step 3: Save and Test (2 minutes)

1. Click **Save** (bottom right)
2. Name your automation: `Hallway - Motion Circadian Lighting`
3. Test it:
   - **Walk into hallway (low light)** → Lights turn on with circadian color
   - **Walk into hallway (bright daylight)** → Lights stay off (lux too high)
   - **Leave hallway** → Staged turn-off begins (timing based on preset)

## What Just Happened?

You now have a hallway/bathroom/zone that:
- ✅ Turns on lights when motion detected (if dark enough)
- ✅ Uses EXACT same color temperature as living room (1800K-5500K)
- ✅ Automatically adjusts brightness based on ambient light
- ✅ Won't flicker on/off during cloudy days (same hysteresis as living room)
- ✅ Gives progressive warnings before turning off (40% → 20% → OFF)
- ✅ Extended debounce during sunrise/sunset to prevent oscillation
- ✅ Seamless color matching when walking between rooms

## Zone Preset Guide

Choose the preset that matches your zone type:

### Very Quick (3/5/7 min) - Closets, Stairs
- **Stage 1 (3 min)**: Dim to 40% + warm to 1800K
- **Stage 2 (5 min)**: Dim to 20%
- **Stage 3 (7 min)**: Turn off
- **Best for**: Quick-access zones, closets, stairways

### Quick (5/10/15 min) - Hallways, Bathrooms (DEFAULT)
- **Stage 1 (5 min)**: Dim to 40% + warm to 1800K
- **Stage 2 (10 min)**: Dim to 20%
- **Stage 3 (15 min)**: Turn off
- **Best for**: Hallways, bathrooms, powder rooms

### Medium (10/15/20 min) - Utility, Garage
- **Stage 1 (10 min)**: Dim to 40% + warm to 1800K
- **Stage 2 (15 min)**: Dim to 20%
- **Stage 3 (20 min)**: Turn off
- **Best for**: Laundry rooms, garages, utility areas

### Custom
- **Configure manually**: Set your own timing for each stage
- **Best for**: Unique zones with specific requirements

## Next Steps (Optional)

### Add Manual Override (Extended Use Mode)
1. Create an input_boolean helper:
   - **Settings** → **Devices & Services** → **Helpers** → **Create Helper**
   - Choose **Toggle** → Name: `Hallway Override`
2. Return to your automation → **Edit**
3. Enable **Manual Override System** → Set to **ON**
4. Select **Override Trigger Entity** → Choose `input_boolean.hallway_override`
5. Save

Now you can toggle override for extended bathroom use, cleaning, or maintenance!

### Set Up Multiple Zones
Repeat this process for each adjacent zone:
- **Upstairs hallway**: Quick preset (5/10/15 min)
- **Downstairs hallway**: Quick preset (5/10/15 min)
- **Bathroom**: Quick preset (5/10/15 min)
- **Stairs**: Very Quick preset (3/5/7 min)
- **Closet**: Very Quick preset (3/5/7 min)
- **Garage**: Medium preset (10/15/20 min)

All zones will have IDENTICAL color temperature - seamless experience!

### Change Zone Preset Later
If timing feels too quick or too slow:
1. Edit automation
2. Change **Zone Type Preset** dropdown
3. Save
4. Test the new timing

## Color Consistency Guarantee

This blueprint uses the EXACT SAME circadian formula as the Intelligent Living Room blueprint:

| Feature | Living Room | Adjacent Zone | Match? |
|---------|-------------|---------------|--------|
| Sun elevation formula | Lines 718-750 | Exact copy | ✅ IDENTICAL |
| Kelvin range | 1800K-5500K | 1800K-5500K | ✅ IDENTICAL |
| Mireds conversion | 1,000,000 / K | 1,000,000 / K | ✅ IDENTICAL |
| Warning color | 1800K (556 mireds) | 1800K (556 mireds) | ✅ IDENTICAL |
| Brightness curve | 50/75/100/125/150 lux | 50/75/100/125/150 lux | ✅ IDENTICAL |
| Lux thresholds | 80/150 lux | 80/150 lux | ✅ IDENTICAL |
| Debounce times | 2/5/10 min | 2/5/10 min | ✅ IDENTICAL |
| Update frequency | 60 seconds | 60 seconds | ✅ IDENTICAL |
| Transition duration | 4 seconds | 4 seconds | ✅ IDENTICAL |

**Result**: Walking from living room to hallway has ZERO color shift!

## Troubleshooting

### Lights turn on when room is already bright
**Solution**: This is expected behavior - lux thresholds are HARDCODED at 80/150 lux
- If motion detected when lux < 80, lights turn on
- If this happens too often, adjust lux sensor placement
- Sensor should measure room's natural light (near window, not direct sun)

### Staged turn-off happens too quickly
**Solution**: Change zone preset
- Current preset: Quick (5/10/15 min)
- Try: Medium (10/15/20 min)
- Or use Custom preset for exact timing

### Staged turn-off happens too slowly
**Solution**: Change zone preset
- Current preset: Quick (5/10/15 min)
- Try: Very Quick (3/5/7 min)
- Perfect for stairs and closets

### Motion sensor too sensitive (lights turn on when passing by)
**Solution**: Adjust motion sensor settings
- Reduce sensor sensitivity (check device settings)
- Adjust sensor mounting angle (aim away from doorway)
- Some sensors have "passing" vs "occupancy" modes

### Lights flicker on/off during cloudy days
**Solution**: This should NOT happen - hardcoded debounce prevents it
- Check lux sensor placement (should not be in direct sunlight)
- Verify sensor is not behind glass (affects lux readings)
- If still flickering, check sensor entity for rapid state changes

### Colors don't match living room
**Solution**: Check circadian rhythm settings
- Both automations must have **Circadian Rhythm** = ENABLED
- Both must have **Update Frequency** = 60 seconds
- Both must have **Transition Duration** = 4 seconds
- If colors still don't match, see COLOR_CONSISTENCY_GUIDE.md

## Advanced Features (For Later)

Once you're comfortable with the basics, explore:
- **Manual Override**: Use physical button or virtual helper
- **Custom Timing**: Fine-tune each stage delay
- **Disable Staged Turn-Off**: Use instant turn-off instead
- **Adjust Update Frequency**: Change how often color updates

See `ADJACENT_ZONE_SETUP_GUIDE.md` for complete documentation.

## Need Help?

- **Color Consistency**: Read `COLOR_CONSISTENCY_GUIDE.md`
- **Full Setup Guide**: Read `ADJACENT_ZONE_SETUP_GUIDE.md`
- **Living Room Setup**: Read `INTELLIGENT_LIVING_ROOM_SETUP_GUIDE.md`
- **Technical Details**: Read `ANTI_FLICKER_TECHNICAL_GUIDE.md`
- **Issues**: Open a GitHub issue with automation logs

## Pro Tips

1. **Zone preset selection**: Choose based on typical occupancy time
2. **Very Quick for stairs**: Prevents lights staying on when just passing through
3. **Quick for bathrooms**: Gives enough warning time before turning off
4. **Medium for garages**: Useful when working on projects
5. **Custom for unique needs**: Laundry room might need 15/20/25 min timing
6. **Override for cleaning**: Activate override before starting, deactivate when done
7. **Consistent sensor placement**: All lux sensors should measure natural light
8. **Test during day and night**: Verify circadian colors change throughout day

## Example Zone Configurations

### Master Bathroom
- **Preset**: Quick (5/10/15 min)
- **Override**: `input_boolean.master_bath_override` (for showers)
- **Typical use**: Short visits get full 15 min, override for longer

### Hallway
- **Preset**: Quick (5/10/15 min)
- **Override**: Not needed (hallways are transitional)
- **Typical use**: Lights turn on when passing, turn off after 15 min

### Stairs
- **Preset**: Very Quick (3/5/7 min)
- **Override**: Not needed
- **Typical use**: Lights turn on when ascending/descending, turn off quickly

### Walk-in Closet
- **Preset**: Very Quick (3/5/7 min)
- **Override**: Optional (for organizing)
- **Typical use**: Quick access, lights turn off fast

### Garage
- **Preset**: Medium (10/15/20 min)
- **Override**: `input_boolean.garage_override` (for projects)
- **Typical use**: Normal parking gets 20 min, override for working on car

---

**You're all set!** Your adjacent zones now have professional-grade motion lighting with PERFECT color consistency to your living room. Enjoy the seamless automation!
