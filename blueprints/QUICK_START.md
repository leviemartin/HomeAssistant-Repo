# Sun-Aware Circadian Lighting - Quick Start Guide

## 5-Minute Setup for Netherlands Families

### Step 1: Install Blueprint (30 seconds)
1. Blueprint already in: `/Users/martinlevie/Documents/GitHub/HomeAssistant-Repo/blueprints/`
2. File: `circadian_rhythm_lighting_sun_aware.yaml`
3. Copy to your Home Assistant: `/config/blueprints/automation/`
4. Restart HA or reload automations

### Step 2: Create Automation (2 minutes)
1. Settings → Automations & Scenes → Create Automation
2. Use Blueprint → "Sun-Aware Circadian Rhythm Lighting with Motion Control"
3. Fill in these fields:

```yaml
# REQUIRED (30 seconds)
target_lights: light.living_room  # Your lights
motion_sensors: binary_sensor.living_room_motion  # Your sensor(s)

# RECOMMENDED FOR NETHERLANDS FAMILIES (1 minute)
sun_tracking_enabled: true  # ← Enable sun awareness
sun_tracking_mode: "hybrid"  # ← Sun during day, fixed evening

kids_bedtime_enabled: true
kids_bedtime: "19:00:00"  # 7pm
kids_bedtime_temp: 2200  # Warm for sleep

parents_bedtime_enabled: true
parents_bedtime: "22:00:00"  # 10pm
parents_bedtime_temp: 2200  # Warm for sleep

# SMART BOUNDARIES (1 minute)
morning_earliest: "07:00:00"  # No 5:30am summer wake-ups!
morning_latest: "09:00:00"  # Start by 9am even in winter
evening_earliest: "18:00:00"  # Evening not before 6pm
evening_latest: "20:30:00"  # Force wind-down by 8:30pm

# OPTIONAL (if you have lux sensor)
lux_sensor: sensor.living_room_illuminance
lux_threshold: 100
lux_evening_override: true  # Summer intelligence
lux_evening_threshold: 50
```

4. Save as "Circadian Lighting - Living Room"

### Step 3: Test (2 minutes)
1. Trigger motion sensor
2. Watch light turn on with appropriate color
3. Check Developer Tools → States → sun.sun
4. Verify elevation angle makes sense for time of day
5. Try again at different times (morning, afternoon, evening)

Done! Your lights now follow the sun.

---

## What You Get

### Summer (June 21)
- **5:30am sunrise** → Morning phase waits until 7am (your boundary)
- **1:30pm** → Sun at 60°, lights at 5500K (cool energizing)
- **7pm** → Sun still up at 15°, but kids bedtime → 2200K warm
- **10pm** → Sun setting, parents bedtime → 2200K warm

### Winter (December 21)
- **8:45am sunrise** → Morning phase starts by 9am (your boundary)
- **12:30pm** → Sun only at 13°, lights at ~3650K (moderate)
- **4:30pm sunset** → Evening transition starts
- **7pm** → Dark for hours, kids bedtime → 2200K warm
- **10pm** → Deep night, parents bedtime → 2200K warm

### Equinox (March/September)
- **7am sunrise** → Natural morning start
- **1pm** → Sun at 37°, lights at ~4850K (balanced)
- **7pm sunset** → Natural evening transition
- **Bedtimes** → Override to 2200K as configured

---

## Troubleshooting (1 Minute Fixes)

### "Lights too bright at bedtime"
✓ Check: bedtime_enabled is `true`
✓ Check: bedtime time is correct (19:00 for kids, 22:00 for parents)

### "Morning starts too early in summer"
✓ Adjust: `morning_earliest` to later time (e.g., 08:00)

### "Evening too energizing in summer"
✓ Adjust: `evening_latest` to earlier (e.g., 20:00)
✓ Enable: `lux_evening_override: true`

### "Not following sun"
✓ Check: `sun_tracking_enabled: true`
✓ Check: sun.sun entity exists (Developer Tools → States)
✓ Check: Not in bedtime override period

### "Too complex, want simple"
✓ Set: `sun_tracking_enabled: false`
✓ Result: Works exactly like original blueprint

---

## Recommended Settings by Scenario

### Young Kids (Age 0-8)
```yaml
kids_bedtime: "19:00:00"  # Early bedtime
evening_latest: "20:00:00"  # Force warm before kids sleep
lux_evening_override: true  # Smart summer handling
```

### Teens (Age 9-17)
```yaml
kids_bedtime: "21:00:00"  # Later bedtime
evening_latest: "21:30:00"  # More flexible evening
```

### Adults Only
```yaml
kids_bedtime_enabled: false  # No kids override
parents_bedtime: "22:30:00"  # Your preference
sun_tracking_mode: "full_sun"  # More sun tracking
```

### Home Office
```yaml
adjust_brightness: true  # Important for work
sun_tracking_mode: "hybrid"  # Balance natural + productive
morning_earliest: "06:00:00"  # Earlier start option
```

### Shift Worker
```yaml
sun_tracking_enabled: false  # Use fixed times
# Then adjust dawn/morning/evening times to your schedule
```

---

## Advanced: Per-Room Configuration

Create multiple automations with different settings:

### Living Room (Family Space)
- Kids bedtime: 19:00 at 2200K
- Evening latest: 20:30
- Lux override: enabled

### Bedroom (Adults)
- Parents bedtime: 22:00 at 2200K
- Evening latest: 21:30 (earlier wind-down)
- Lux override: enabled

### Home Office
- No bedtime overrides
- Brightness adjustment: enabled
- Sun tracking: full_sun mode
- More energizing during work hours

### Hallway
- Simple: sun tracking disabled
- Fixed times work fine
- Lower brightness overall

---

## Documentation Files

### Just Getting Started?
→ Read this file (QUICK_START.md)

### Want to Understand Features?
→ Read `FEATURE_COMPARISON.md`
→ Compares original vs. sun-aware blueprint

### Want Scientific Details?
→ Read `SUN_AWARE_UPGRADE_GUIDE.md`
→ Sun elevation angles, research, edge cases

### Want Technical Details?
→ Read `CHANGES_SUMMARY.md`
→ Implementation details, variables, logic flow

---

## Success Checklist

After 1 week of use, you should notice:

- [ ] Lights feel more "natural" during daytime
- [ ] No more bright lights at kids' bedtime in summer
- [ ] Smooth adaptation as days get longer/shorter
- [ ] No manual adjustments needed between seasons
- [ ] Better sleep due to proper evening light
- [ ] More alert during day (especially winter)
- [ ] Family routines maintained despite sun position

If all checked: Congratulations, your circadian lighting is optimized for Netherlands!

If some unchecked: See troubleshooting section or read full documentation.

---

## One-Line Summary

**"This blueprint makes your lights follow the sun during the day, but respects your family's bedtime routines—perfect for Netherlands where summer sun sets at 10pm and winter sun barely rises."**

---

**Quick Start Version:** 1.0
**Last Updated:** 2025-11-17
**For Blueprint:** Sun-Aware Circadian Rhythm Lighting v2.0
**Target Users:** Netherlands families with young kids
