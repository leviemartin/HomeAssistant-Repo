# Circadian Rhythm Lighting - Original vs Sun-Aware Comparison

## Quick Decision Guide

**Use Original Blueprint if:**
- You want consistent, predictable lighting schedules
- Your daily routine is fixed regardless of season
- You work night shifts or non-standard hours
- You prefer simplicity over seasonal adaptation

**Use Sun-Aware Blueprint if:**
- You live in Netherlands or similar high-latitude location (> 45°N or S)
- You experience extreme seasonal daylight variations
- You want authentic circadian rhythm following natural sunlight
- You have kids with fixed bedtimes during long summer days
- You want to balance natural light cycles with social schedules

## Feature Comparison Table

| Feature | Original Blueprint | Sun-Aware Blueprint |
|---------|-------------------|---------------------|
| **Time-based phases** | 6 fixed phases | 6 fixed phases + sun tracking |
| **Color temperature** | Fixed times only | Sun elevation OR fixed times |
| **Seasonal adaptation** | Manual adjustment needed | Automatic via sun position |
| **Bedtime overrides** | Not available | Kids + Parents configurable |
| **Evening intelligence** | Fixed 6pm-9pm schedule | Lux-aware summer/winter logic |
| **Time boundaries** | Fixed only | Min/max limits for sun tracking |
| **Configuration complexity** | Simple (6 time inputs) | Flexible (29 inputs total) |
| **Backward compatibility** | N/A | 100% compatible (sun tracking can be disabled) |
| **Netherlands summer** | 10pm still energizing | Bedtime overrides + lux awareness |
| **Netherlands winter** | 4:30pm darkness needs manual tweaks | Auto-adjusts to early sunset |
| **Motion sensor support** | Yes | Yes (unchanged) |
| **Dim warning** | Yes | Yes (unchanged) |
| **Lux sensor basic** | Yes (prevent daytime activation) | Yes (unchanged) |
| **Lux sensor advanced** | No | Yes (evening context awareness) |
| **Brightness adjustment** | Optional, time-based | Optional, sun-aware OR time-based |

## Color Temperature Behavior Comparison

### Scenario 1: June 21 (Summer Solstice) at 7:30pm

**Original Blueprint:**
- Uses "evening" phase (6pm-9pm)
- Color temperature: 3000K (warm white)
- No consideration that sun is still high (elevation ~20°)
- May be too cool for pre-bedtime routine

**Sun-Aware Blueprint (Hybrid Mode):**
- Kids bedtime override: 2200K (if enabled, recommended)
- Without override: Checks sun elevation (~20°)
  - If lux sensor shows bright: Progressive cooling 4000K → 3000K
  - If lux sensor shows dark: Force 3000K warm white
- Adapts to actual environmental conditions

### Scenario 2: December 21 (Winter Solstice) at 4:45pm

**Original Blueprint:**
- Uses "afternoon" phase (3pm-6pm)
- Color temperature: 4500K (neutral white)
- Sun set 15 minutes ago, but lights still energizing
- Conflicts with body's darkness response

**Sun-Aware Blueprint (Hybrid Mode):**
- Detects sun elevation: -2° (below horizon)
- Transition zone logic activates
- Color temperature: Progressive 3500K → 2200K
- Matches actual darkness outside

### Scenario 3: March 21 (Equinox) at 1:00pm

**Original Blueprint:**
- Uses "midday" phase (10am-3pm)
- Color temperature: 5500K (cool daylight)
- Works perfectly for this scenario

**Sun-Aware Blueprint (Sun Tracking Mode):**
- Detects sun elevation: ~37° (Netherlands equinox noon)
- Calculates: 4500 + (37-30)*50 = 4850K
- Slightly less aggressive than fixed mode
- More authentic to actual sunlight at this elevation

Both work well here, sun-aware is slightly warmer and more natural.

## Configuration Examples

### Original Blueprint - Family Configuration
```yaml
target_lights: light.living_room
motion_sensors: binary_sensor.living_room_motion
light_timeout: 300
adjust_brightness: true
dim_warning_enabled: true

# Fixed time schedule
dawn_start: "05:00:00"
morning_start: "07:00:00"
midday_start: "10:00:00"
afternoon_start: "15:00:00"
evening_start: "18:00:00"
night_start: "21:00:00"

lux_sensor: sensor.living_room_illuminance
lux_threshold: 100
```

**Pros:** Simple, predictable, easy to understand
**Cons:** Need seasonal manual adjustment, no bedtime logic

### Sun-Aware Blueprint - Recommended Family Configuration
```yaml
target_lights: light.living_room
motion_sensors: binary_sensor.living_room_motion
light_timeout: 300
adjust_brightness: true
dim_warning_enabled: true

# Sun tracking enabled
sun_tracking_enabled: true
sun_tracking_mode: "hybrid"

# Bedtime overrides (KEY FEATURE!)
kids_bedtime_enabled: true
kids_bedtime: "19:00:00"
kids_bedtime_temp: 2200

parents_bedtime_enabled: true
parents_bedtime: "22:00:00"
parents_bedtime_temp: 2200

# Smart boundaries (prevent extreme schedules)
morning_earliest: "07:00:00"  # No 5:30am summer wake-ups
morning_latest: "09:00:00"    # Start by 9am even in winter
evening_earliest: "18:00:00"  # Evening routine not before 6pm
evening_latest: "20:30:00"    # Force wind-down by 8:30pm

# Advanced lux sensor usage
lux_sensor: sensor.living_room_illuminance
lux_threshold: 100
lux_evening_override: true    # Summer evening intelligence
lux_evening_threshold: 50

# Backward compatible fixed times (used for dawn/as fallbacks)
dawn_start: "05:00:00"
morning_start: "07:00:00"
midday_start: "10:00:00"
afternoon_start: "15:00:00"
evening_start: "18:00:00"
night_start: "21:00:00"
```

**Pros:** Adapts to seasons, bedtime logic, lux-aware, comprehensive
**Cons:** More inputs to configure initially (but defaults are sensible)

## Migration Path

### Step 1: Install Sun-Aware Blueprint (Side by Side)
1. Download `circadian_rhythm_lighting_sun_aware.yaml`
2. Place in `/config/blueprints/automation/`
3. Restart Home Assistant or reload automations
4. Keep original automation running

### Step 2: Create Test Automation
1. Settings → Automations & Scenes → Create Automation → Use Blueprint
2. Select "Sun-Aware Circadian Rhythm Lighting with Motion Control"
3. Configure same lights and sensors as original
4. **Initially set:** `sun_tracking_enabled: false`
5. This makes it behave identically to original
6. Save as "Circadian Lighting (Sun-Aware Test)"

### Step 3: Gradual Transition
1. Test for 24 hours with sun tracking disabled
2. If working: Enable sun tracking (`sun_tracking_enabled: true`)
3. Set mode to "hybrid"
4. Test for several days across different times
5. Enable bedtime overrides if you have kids
6. Fine-tune time boundaries for your preferences

### Step 4: Full Cutover
1. Once satisfied, disable original automation
2. Rename sun-aware automation to primary name
3. Delete original automation after 1 week confidence period

### Rollback Plan
If you need to revert:
1. Disable sun-aware automation
2. Re-enable original automation
3. No data loss, instant rollback

## Performance Impact

| Aspect | Original | Sun-Aware | Notes |
|--------|----------|-----------|-------|
| **Trigger frequency** | Motion only | Motion only | Identical |
| **Template complexity** | Simple (6 conditions) | Moderate (15-20 conditions) | Still executes in milliseconds |
| **CPU usage** | Minimal | Minimal | Only calculates on motion trigger |
| **Memory** | Minimal | Minimal | No persistent storage |
| **Sun entity dependency** | None | Yes (`sun.sun`) | Built-in HA integration |
| **Execution time** | ~50ms | ~100ms | Imperceptible difference |

## Troubleshooting Comparison

### Original Blueprint Issues
**Problem:** Lights too bright at 5pm in winter (sun already set)
**Solution:** Manually adjust `afternoon_start` to 14:00 in winter, revert in summer

**Problem:** Lights too warm at 8pm in summer (sun still up)
**Solution:** Manually adjust `evening_start` to 20:00 in summer, revert in winter

**Problem:** Kids can't sleep due to bright light at bedtime
**Solution:** No built-in solution, need separate automation or manual override

### Sun-Aware Blueprint Solutions
**Problem:** Lights too bright at 5pm in winter
**Solution:** Automatic! Sun elevation-based calculation detects sunset

**Problem:** Lights too warm at 8pm in summer
**Solution:** Bedtime override forces sleep-friendly temperature regardless of sun

**Problem:** Kids can't sleep due to bright light
**Solution:** Enable kids_bedtime_override with 7pm cutoff

## Real-World User Scenarios

### Scenario A: Amsterdam Family with Young Kids
**Challenges:**
- Summer: Kids bedtime at 7pm, sun sets at 10pm
- Winter: Dinner at 6pm, sun set at 4:30pm already
- Want circadian benefits but need consistent bedtimes

**Recommended:** Sun-Aware Blueprint
- Enable sun tracking (hybrid mode)
- Set kids_bedtime: 19:00 with 2200K override
- Set evening_latest: 20:30 to force parental wind-down
- Use lux_evening_override for context awareness
- Results: Authentic daytime circadian rhythm, guaranteed sleep-friendly evenings

### Scenario B: Rotterdam Single Professional
**Challenges:**
- Flexible schedule, no kids
- Wants maximum circadian benefit
- Okay with seasonal variation in evening timing

**Recommended:** Sun-Aware Blueprint (Full Sun Mode)
- Enable sun tracking (full_sun mode)
- Disable bedtime overrides
- Looser time boundaries (morning 6am-10am, evening 5pm-11pm)
- Let sun guide naturally
- Results: Most authentic natural light simulation

### Scenario C: Eindhoven Shift Worker
**Challenges:**
- Works 2pm-10pm shift
- Sleeps 11pm-7am
- Seasons don't matter, need consistency

**Recommended:** Original Blueprint OR Sun-Aware (Fixed Mode)
- Original blueprint works perfectly
- OR sun-aware with sun_tracking_enabled: false
- Customize fixed times to match shift schedule
- Results: Predictable, shift-appropriate lighting

### Scenario D: Maastricht Elderly Couple
**Challenges:**
- Fixed routines year-round
- Early bedtime (8pm)
- Confusion with changing settings

**Recommended:** Original Blueprint
- Simplicity is key
- Fixed 8pm evening phase works well
- No seasonal adjustment needed (prefer consistency)
- Results: Reliable, understandable, low-maintenance

## Scientific Accuracy Comparison

### Original Blueprint Approach
- Based on: General circadian rhythm research
- Assumes: Fixed daily light cycles (works for equatorial regions)
- Strengths: Predictable, well-tested, scientifically sound for stable climates
- Limitations: Doesn't account for seasonal variation in natural light

### Sun-Aware Blueprint Approach
- Based on: Solar elevation angle correlation to natural CCT (correlated color temperature)
- Matches: Actual sunlight characteristics at your latitude
- Strengths: Authentic circadian mimicry, adapts to photoperiod changes
- Research backed: Multiple studies on sun elevation and color temperature correlation
- Limitations: More complex, requires understanding of hybrid modes

Both are scientifically valid. Sun-aware is more authentic to natural sunlight cycles.

## Recommendations by Latitude

| Latitude Range | Location Examples | Recommended Blueprint | Reason |
|---------------|-------------------|----------------------|--------|
| 0° - 30° | Singapore, Mumbai, Miami | Original | Minimal seasonal variation |
| 30° - 45° | Los Angeles, Tokyo, Sydney | Either | Moderate seasons, personal preference |
| 45° - 55° | Montreal, Netherlands, Berlin | Sun-Aware | Significant seasonal variation |
| 55° - 66° | Scotland, Alaska, Scandinavia | Sun-Aware (Essential!) | Extreme seasonal variation |
| > 66° | Arctic/Antarctic | Sun-Aware + Custom | Polar day/night periods |

**Netherlands is at 52°N - Solidly in the "Sun-Aware Recommended" zone.**

## Final Recommendations

### Use Original Blueprint If:
1. You live below 40° latitude (minimal seasonal change)
2. You prefer simplicity and predictability above all else
3. You have non-standard schedules (shift work, elderly care)
4. You're just starting with circadian lighting

### Use Sun-Aware Blueprint If:
1. You live in Netherlands or similar high-latitude location
2. You have kids with fixed bedtimes during summer
3. You experience "winter blues" and want maximum seasonal adaptation
4. You have lux sensors and want intelligent evening decisions
5. You're a circadian lighting enthusiast wanting maximum authenticity

### Best of Both Worlds:
**Install sun-aware blueprint with `sun_tracking_enabled: false` initially.**
- Start with original behavior
- Graduate to sun tracking when comfortable
- Keep fallback option always available

---

## Summary

The sun-aware blueprint is a **comprehensive enhancement** that:
- ✓ Preserves all original features
- ✓ Adds intelligent sun tracking for authentic circadian rhythm
- ✓ Solves Netherlands-specific seasonal extreme challenges
- ✓ Provides bedtime override solutions for families
- ✓ Maintains backward compatibility
- ✓ Offers flexible configuration for different user needs

**For Netherlands residents with families: The sun-aware blueprint is specifically designed for your use case.**

---

**Document Version:** 1.0
**Last Updated:** 2025-11-17
**Comparison Based On:** Original v1.0 vs Sun-Aware v2.0
