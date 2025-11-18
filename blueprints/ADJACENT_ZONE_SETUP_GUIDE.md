# Adjacent Zone Motion - Circadian Lighting v1.0 - Complete Setup Guide

Comprehensive guide for setting up motion-activated circadian lighting for hallways, bathrooms, stairs, and adjacent zones with MAXIMUM color consistency to the Intelligent Living Room blueprint.

**Version**: 1.0.0
**Last Updated**: 2025-11-18
**Compatibility**: Home Assistant 2024.1+

## Table of Contents

1. [Overview](#overview)
2. [Hardware Requirements](#hardware-requirements)
3. [Installation](#installation)
4. [Basic Configuration](#basic-configuration)
5. [Zone Preset System](#zone-preset-system)
6. [Color Consistency Technical Details](#color-consistency-technical-details)
7. [Manual Override Setup](#manual-override-setup)
8. [Troubleshooting](#troubleshooting)
9. [Example Configurations](#example-configurations)
10. [Advanced Customization](#advanced-customization)

---

## Overview

This blueprint creates motion-activated intelligent lighting for adjacent zones (hallways, bathrooms, stairs, closets, garages) with PERFECT color consistency to the Intelligent Living Room blueprint.

### Key Features

- **Motion Sensor Integration**: Works with any PIR or mmWave motion sensor
- **Zone Preset System**: Quick-select timing for common zone types (Very Quick/Quick/Medium/Custom)
- **Color Consistency**: EXACT same circadian sun elevation formula as living room (1800K-5500K)
- **Lux-Aware Control**: Hardcoded thresholds (80/150 lux) match living room for consistency
- **Dynamic Brightness**: IDENTICAL brightness curve as living room (50/75/100/125/150 lux breakpoints)
- **Staged Turn-Off**: Progressive warnings with SAME dimming percentages as living room (40%/20%)
- **Manual Override**: Button/switch/input_boolean keeps lights on regardless of lux
- **Sunrise/Sunset Intelligence**: Extended 10-minute debounce during dawn/dusk (matches living room)

### What's Different from Living Room Blueprint?

**EXCLUDED Features (not needed for adjacent zones)**:
- ❌ Aqara FP2 approach detection
- ❌ Scene cycling system
- ❌ Target count awareness
- ❌ Configurable lux thresholds (hardcoded instead)
- ❌ Configurable brightness curve (hardcoded instead)
- ❌ Configurable debounce times (hardcoded instead)

**INCLUDED Features (same as living room)**:
- ✅ Exact circadian sun elevation logic
- ✅ Dynamic brightness based on lux
- ✅ Staged turn-off warnings (with warm color shift)
- ✅ Manual override toggle
- ✅ Extended sunrise/sunset debounce
- ✅ Anti-flicker hysteresis
- ✅ Continuous circadian monitoring

**SIMPLIFIED Features (optimized for adjacent zones)**:
- 🔄 Motion sensor instead of continuous presence detection
- 🔄 Preset-based timing instead of configurable delays
- 🔄 Hardcoded consistency parameters

---

## Hardware Requirements

### Required Components

1. **Tunable White Lights**
   - Philips Hue (White Ambiance, White & Color Ambiance)
   - IKEA Tradfri (tunable white bulbs)
   - Any smart bulb with `color_temp` support
   - **Note**: Must support mireds (micro reciprocal degrees) color temperature

2. **Motion Sensor**
   - PIR motion sensors (most common)
   - mmWave presence sensors (more accurate)
   - Occupancy sensors (any type)
   - **Integration**: Must expose `binary_sensor.xxx_motion` entity
   - **Examples**:
     - Philips Hue Motion Sensor
     - Aqara Motion Sensor (RTCGQ11LM)
     - Shelly Motion (SHMOS-01)
     - Z-Wave motion sensors
     - Aqara FP2 (overkill but works)

3. **Lux/Illuminance Sensor**
   - Motion sensor's built-in lux sensor (if available)
   - Separate Xiaomi/Aqara lux sensor
   - Philips Hue Motion Sensor (includes lux)
   - Any sensor with `sensor.xxx_illuminance` entity

### Optional Components

4. **Override Trigger** (For Manual Control)
   - Aqara Wireless Button (WXKG11LM, WXKG12LM, WXKG13LM)
   - Philips Hue Dimmer Switch
   - IKEA Tradfri Remote
   - Virtual `input_boolean` helper
   - Any `binary_sensor` or `switch` entity

### Recommended Hardware Combinations

**Budget Option**:
- IKEA Tradfri tunable white bulbs ($15-20 each)
- Any ZigBee PIR motion sensor ($15-25)
- Same motion sensor's built-in lux (if available)
- Total per zone: ~$30-45

**Premium Option**:
- Philips Hue White Ambiance bulbs ($25-30 each)
- Philips Hue Motion Sensor ($40, includes lux)
- Philips Hue Dimmer Switch for override ($20)
- Total per zone: ~$85-90

**Best Value Option**:
- Philips Hue White Ambiance bulbs ($25-30 each)
- Aqara Motion Sensor ($15-20, includes lux)
- Virtual input_boolean for override (FREE)
- Total per zone: ~$40-50

---

## Installation

### Method 1: Import via URL (Recommended)

1. Open Home Assistant web interface
2. Navigate to **Settings** → **Automations & Scenes** → **Blueprints**
3. Click **Import Blueprint** (bottom right corner)
4. Paste the blueprint URL:
   ```
   https://github.com/martinlevie/HomeAssistant-Repo/blueprints/adjacent_zone_motion_circadian_lighting.yaml
   ```
5. Click **Preview Blueprint**
6. Review the blueprint details
7. Click **Import Blueprint**

### Method 2: Manual Installation

1. Download `adjacent_zone_motion_circadian_lighting.yaml`
2. Copy to Home Assistant config directory:
   ```
   /config/blueprints/automation/adjacent_zone_motion_circadian_lighting.yaml
   ```
3. Restart Home Assistant or reload automations:
   - **Developer Tools** → **YAML** → **Reload Automations**

### Verify Installation

1. Navigate to **Settings** → **Automations & Scenes** → **Blueprints**
2. Look for "Adjacent Zone Motion - Circadian Lighting v1.0"
3. If visible, installation successful!

---

## Basic Configuration

### Step 1: Create the Automation

1. Navigate to **Settings** → **Automations & Scenes** → **Blueprints**
2. Find the blueprint and click **Create Automation**
3. Fill in the required fields:

### Required Inputs

| Input | Description | Example |
|-------|-------------|---------|
| **Target Lights** | Lights to control in this zone | `light.hallway_ceiling` |
| **Motion Sensor** | PIR or mmWave motion sensor | `binary_sensor.hallway_motion` |
| **Lux/Illuminance Sensor** | Ambient light sensor | `sensor.hallway_illuminance` |
| **Zone Type Preset** | Quick-select for zone type | `Quick (5/10/15 min)` |

### Optional Inputs (with Defaults)

| Input | Default | Description |
|-------|---------|-------------|
| **Enable Manual Override System** | ON | Allow override toggle |
| **Override Trigger Entity** | None | Button/switch for override |
| **Enable Circadian Rhythm** | ON | Sun-aware color temperature |
| **Circadian Update Frequency** | 60 sec | How often to update color |
| **Circadian Transition Duration** | 4 sec | Smoothness of transitions |
| **Enable Staged Turn-Off** | ON | Progressive warnings before off |

### Hardcoded Settings (NOT Configurable)

These values are LOCKED to ensure consistency with living room:

| Setting | Value | Reason |
|---------|-------|--------|
| **Lux Turn-ON Threshold** | 80 lux | Matches living room |
| **Lux Turn-OFF Threshold** | 150 lux | Matches living room |
| **Turn-ON Debounce** | 2 minutes | Matches living room |
| **Turn-OFF Debounce** | 5 minutes | Matches living room |
| **Sunrise/Sunset Debounce** | 10 minutes | Matches living room |
| **Stage 1 Brightness** | 40% | Matches living room |
| **Stage 2 Brightness** | 20% | Matches living room |
| **Brightness at 50 lux** | 100% | Matches living room |
| **Brightness at 75 lux** | 85% | Matches living room |
| **Brightness at 100 lux** | 70% | Matches living room |
| **Brightness at 125 lux** | 55% | Matches living room |
| **Brightness at 150 lux** | 40% | Matches living room |

---

## Zone Preset System

The zone preset system provides quick-select timing configurations for common zone types.

### Preset Comparison Table

| Preset | Stage 1 | Stage 2 | Stage 3 | Best For |
|--------|---------|---------|---------|----------|
| **Very Quick** | 3 min (40%) | 5 min (20%) | 7 min (OFF) | Closets, Stairs |
| **Quick** | 5 min (40%) | 10 min (20%) | 15 min (OFF) | Hallways, Bathrooms |
| **Medium** | 10 min (40%) | 15 min (20%) | 20 min (OFF) | Utility, Garage |
| **Custom** | Your choice | Your choice | Your choice | Unique needs |

### Very Quick (3/5/7 min)

**Target Zones**: Closets, Stairs, Short-transit areas

**Timing Breakdown**:
- **Motion clears → 3 minutes**: Lights dim to 40% + warm to 1800K (first warning)
- **5 minutes total**: Lights dim to 20% (second warning)
- **7 minutes total**: Lights turn off completely

**Use Cases**:
- Walk-in closets (quick access, grab item, leave)
- Stairways (ascending/descending takes seconds)
- Pantries (quick food grab)
- Any zone where you spend < 5 minutes typically

**Why This Timing?**:
- Short enough to not waste energy
- Long enough to provide warning if still in zone
- 3-minute first warning gives time to react

### Quick (5/10/15 min) - DEFAULT

**Target Zones**: Hallways, Bathrooms, Powder Rooms

**Timing Breakdown**:
- **Motion clears → 5 minutes**: Lights dim to 40% + warm to 1800K (first warning)
- **10 minutes total**: Lights dim to 20% (second warning)
- **15 minutes total**: Lights turn off completely

**Use Cases**:
- Main hallways (transitional spaces)
- Guest bathrooms (short visits)
- Powder rooms (quick use)
- Mudrooms (entering/leaving home)

**Why This Timing?**:
- 5-minute first warning is safe for most bathroom visits
- 15-minute total gives plenty of warning time
- Balances energy savings with user comfort

### Medium (10/15/20 min)

**Target Zones**: Utility Areas, Garages, Workshops

**Timing Breakdown**:
- **Motion clears → 10 minutes**: Lights dim to 40% + warm to 1800K (first warning)
- **15 minutes total**: Lights dim to 20% (second warning)
- **20 minutes total**: Lights turn off completely

**Use Cases**:
- Laundry rooms (folding clothes, loading machines)
- Garages (parking, retrieving items)
- Workshops (quick projects)
- Storage rooms (organizing, searching)

**Why This Timing?**:
- Longer delays accommodate work/tasks
- 10-minute first warning for active work
- 20-minute total provides generous time

### Custom

**Target Zones**: Unique zones with specific requirements

**Configuration**:
- **Custom Stage 1 Delay**: Set your own (1-60 minutes)
- **Custom Stage 2 Delay**: Set your own (1-60 minutes)
- **Custom Stage 3 Delay**: Set your own (1-60 minutes)

**Use Cases**:
- Server room (15/20/25 min - extended access)
- Attic (20/25/30 min - organizing storage)
- Pet room (8/12/18 min - feeding/cleaning)
- Any zone with unique timing needs

**Examples**:
- **Server Room**: 15/20/25 min (need time to work on equipment)
- **Attic**: 20/25/30 min (organizing takes time)
- **Pet Room**: 8/12/18 min (feeding + cleaning routine)

---

## Color Consistency Technical Details

This section explains HOW color consistency is achieved between the living room and adjacent zones.

### Circadian Sun Elevation Formula (EXACT MATCH)

Both blueprints use IDENTICAL sun elevation to Kelvin mapping:

```yaml
# EXACT COPY from living room blueprint (lines 718-750)
color_temp_kelvin: >
  {% if circadian_rhythm %}
    {% set elevation = sun_elevation | float %}

    {# Deep night (sun well below horizon) #}
    {% if elevation < -6 %}
      1800

    {# Civil twilight (sun just below horizon) - gentle warm transition #}
    {% elif elevation >= -6 and elevation < 0 %}
      {{ (1800 + (elevation + 6) * 200) | int }}  {# 1800K to 3000K #}

    {# Low sun (sunrise/sunset) - warming up or cooling down #}
    {% elif elevation >= 0 and elevation < 10 %}
      {{ (3000 + (elevation * 50)) | int }}  {# 3000K to 3500K #}

    {# Rising/falling sun - energizing transition #}
    {% elif elevation >= 10 and elevation < 30 %}
      {{ (3500 + (elevation - 10) * 75) | int }}  {# 3500K to 5000K #}

    {# High sun - peak alertness #}
    {% elif elevation >= 30 and elevation < 50 %}
      {{ (5000 + (elevation - 30) * 25) | int }}  {# 5000K to 5500K #}

    {# Very high sun (summer midday) - maximum cool daylight #}
    {% else %}
      5500
    {% endif %}
  {% else %}
    3000  {# Default warm white if circadian disabled #}
  {% endif %}
```

### Kelvin to Mireds Conversion (EXACT MATCH)

Both blueprints use IDENTICAL conversion formula:

```yaml
# Formula: mireds = 1,000,000 / kelvin
color_temp_mireds: "{{ (1000000 / (color_temp_kelvin | int)) | int }}"
```

### Warning Color Temperature (EXACT MATCH)

Both blueprints use IDENTICAL warning color (1800K warm):

```yaml
# Always 1800K warm during staged warnings (converted to mireds)
warning_color_temp_mireds: "{{ (1000000 / 1800) | int }}"  # = 556 mireds
```

### Dynamic Brightness Curve (EXACT MATCH)

Both blueprints use IDENTICAL lux-to-brightness mapping:

```yaml
calculated_brightness: >
  {% set lux = current_lux | float %}
  {% if lux <= 50 %}
    100                                    # Very dark: 100%
  {% elif lux <= 75 %}
    {% set range_pct = (lux - 50) / 25 %}
    {{ (100 - (100 - 85) * range_pct) | int }}  # Linear interpolation
  {% elif lux <= 100 %}
    {% set range_pct = (lux - 75) / 25 %}
    {{ (85 - (85 - 70) * range_pct) | int }}    # Linear interpolation
  {% elif lux <= 125 %}
    {% set range_pct = (lux - 100) / 25 %}
    {{ (70 - (70 - 55) * range_pct) | int }}    # Linear interpolation
  {% elif lux <= 150 %}
    {% set range_pct = (lux - 125) / 25 %}
    {{ (55 - (55 - 40) * range_pct) | int }}    # Linear interpolation
  {% else %}
    40                                     # At/above OFF threshold: 40%
  {% endif %}
```

### Lux Hysteresis System (EXACT MATCH)

Both blueprints use IDENTICAL anti-flicker thresholds:

| Parameter | Living Room | Adjacent Zone | Match? |
|-----------|-------------|---------------|--------|
| Turn-ON threshold | 80 lux | 80 lux | ✅ IDENTICAL |
| Turn-OFF threshold | 150 lux | 150 lux | ✅ IDENTICAL |
| Turn-ON debounce | 120 sec (2 min) | 120 sec (2 min) | ✅ IDENTICAL |
| Turn-OFF debounce | 300 sec (5 min) | 300 sec (5 min) | ✅ IDENTICAL |
| Sunrise/sunset debounce | 600 sec (10 min) | 600 sec (10 min) | ✅ IDENTICAL |

### Update Frequency and Transitions (EXACT MATCH)

Both blueprints use IDENTICAL timing parameters:

| Parameter | Living Room | Adjacent Zone | Match? |
|-----------|-------------|---------------|--------|
| Circadian update interval | 60 seconds | 60 seconds | ✅ IDENTICAL |
| Circadian transition duration | 4 seconds | 4 seconds | ✅ IDENTICAL |
| Stage 1 brightness | 40% | 40% | ✅ IDENTICAL |
| Stage 2 brightness | 20% | 20% | ✅ IDENTICAL |

### Result: PERFECT Color Matching

When you walk from living room (with Intelligent Living Room blueprint) to hallway (with Adjacent Zone blueprint):

1. **Sun elevation is same** → Both use `state_attr('sun.sun', 'elevation')`
2. **Kelvin calculation is identical** → Both use exact same formula
3. **Mireds conversion is identical** → Both use `1000000 / kelvin`
4. **Brightness scaling is identical** → Both use same lux breakpoints
5. **Update timing is identical** → Both update every 60 seconds

**Result**: ZERO color shift between rooms! Seamless experience.

---

## Manual Override Setup

Manual override allows you to keep lights on regardless of lux level - useful for cleaning, extended bathroom use, or maintenance.

### Option 1: Virtual Input Boolean (Easiest)

**Step 1: Create Helper**
1. Navigate to **Settings** → **Devices & Services** → **Helpers**
2. Click **Create Helper** (bottom right)
3. Choose **Toggle** (input_boolean)
4. Configure:
   - **Name**: `Hallway Override` (or zone name)
   - **Icon**: `mdi:lightbulb` or `mdi:hand-back-left`
5. Click **Create**

**Step 2: Add to Automation**
1. Edit your Adjacent Zone automation
2. **Enable Manual Override System**: ON
3. **Override Trigger Entity**: Select `input_boolean.hallway_override`
4. Save

**Step 3: Add to Dashboard**
1. Edit your dashboard
2. Add **Entities Card**
3. Add entity: `input_boolean.hallway_override`
4. Toggle ON to activate override, OFF to deactivate

### Option 2: Physical Button (More Convenient)

**Supported Buttons**:
- Aqara Wireless Button (WXKG11LM, WXKG12LM, WXKG13LM)
- Philips Hue Dimmer Switch
- IKEA Tradfri Remote
- Any Zigbee/Z-Wave button

**Step 1: Pair Button**
1. Pair button with Home Assistant (Zigbee integration)
2. Note the entity ID (e.g., `sensor.aqara_button_action`)

**Step 2: Create Input Boolean**
1. Create input_boolean helper (see Option 1)

**Step 3: Create Button Automation**
1. Create new automation (separate from blueprint)
2. **Trigger**: Button pressed (e.g., `sensor.aqara_button_action` = `single`)
3. **Action**: Toggle input_boolean
   ```yaml
   service: input_boolean.toggle
   target:
     entity_id: input_boolean.hallway_override
   ```

**Step 4: Link to Blueprint**
1. Edit your Adjacent Zone automation
2. **Enable Manual Override System**: ON
3. **Override Trigger Entity**: Select `input_boolean.hallway_override`
4. Save

Now pressing the physical button toggles override mode!

### Option 3: Switch Entity (Direct Integration)

If your button/switch exposes a switch entity directly:

1. Edit your Adjacent Zone automation
2. **Enable Manual Override System**: ON
3. **Override Trigger Entity**: Select your switch entity
4. Save

The switch state (on/off) controls override mode.

### Override Behavior

When override is ACTIVE (ON):
- ✅ Lights turn on when motion detected (regardless of lux)
- ✅ Lights stay on even if lux > 150
- ✅ Circadian color updates continue normally
- ✅ Dynamic brightness updates continue normally
- ❌ Lux-based turn-off is disabled

When override is INACTIVE (OFF):
- Normal lux-aware behavior resumes
- Lights turn off if lux > 150 (after debounce)
- Staged turn-off occurs after motion clears

### Use Cases

- **Bathroom cleaning**: Activate override, clean, deactivate when done
- **Extended bathroom use**: Activate for long shower, auto-off after
- **Garage projects**: Activate for car work, deactivate when finished
- **Maintenance work**: Activate for repairs, deactivate when complete

---

## Troubleshooting

### Motion Detection Issues

**Problem**: Lights don't turn on when motion detected

**Solutions**:
1. Check motion sensor entity state:
   - **Developer Tools** → **States** → Search for motion sensor
   - Walk in front of sensor → State should change to "on"
   - If state doesn't change, check sensor integration
2. Check lux level:
   - Lights only turn on if lux < 80
   - View lux sensor state in Developer Tools
   - If lux > 80, lights won't turn on (expected behavior)
3. Enable override temporarily to test:
   - This bypasses lux check
   - If lights turn on with override, lux is the issue

**Problem**: Motion sensor too sensitive (triggers when passing by)

**Solutions**:
1. Adjust sensor sensitivity (if available in device settings)
2. Change sensor mounting angle (aim away from door)
3. Use "occupancy" mode instead of "motion" mode (if sensor supports it)
4. Move sensor to better location (center of room, not near door)

### Staged Turn-Off Issues

**Problem**: Lights turn off too quickly

**Solutions**:
1. Change zone preset to longer timing:
   - Current: Very Quick (3/5/7 min)
   - Try: Quick (5/10/15 min) or Medium (10/15/20 min)
2. Use Custom preset for exact timing

**Problem**: Lights turn off too slowly

**Solutions**:
1. Change zone preset to shorter timing:
   - Current: Quick (5/10/15 min)
   - Try: Very Quick (3/5/7 min)
2. Disable staged turn-off entirely:
   - **Enable Staged Turn-Off**: OFF
   - Lights will turn off immediately when motion clears

**Problem**: Staged warnings not visible (don't notice dimming)

**Solutions**:
1. This is expected behavior - warnings are subtle
2. First warning: 40% brightness (60% reduction)
3. Second warning: 20% brightness (80% reduction)
4. Percentages are hardcoded to match living room
5. Move when you notice dimming to reset timer

### Color Consistency Issues

**Problem**: Colors don't match living room

**Solutions**:
1. Verify both automations have **Circadian Rhythm** = ENABLED
2. Verify both have **Update Frequency** = 60 seconds
3. Verify both have **Transition Duration** = 4 seconds
4. Check if lights support color_temp:
   - Developer Tools → States → Check light attributes
   - Must have `color_temp` attribute
5. Verify lights are Philips Hue or IKEA Tradfri (best compatibility)

**Problem**: Adjacent zone color seems delayed vs living room

**Solutions**:
1. Both automations update every 60 seconds
2. Updates may occur at different times (offset by up to 60 sec)
3. This is normal and barely noticeable
4. After 60 seconds, colors will match perfectly

### Lux Threshold Issues

**Problem**: Lights turn on when room seems bright enough

**Solutions**:
1. Lux thresholds are HARDCODED (80/150) - not configurable
2. Check lux sensor placement:
   - Should measure room's natural light
   - Place near window but not in direct sunlight
   - Should not be behind glass (affects readings)
3. Check current lux level:
   - Developer Tools → States → View lux sensor
   - If lux < 80, lights WILL turn on (expected)
4. If sensor placement is optimal but threshold feels wrong:
   - Thresholds are intentionally matched to living room
   - Cannot be changed for consistency reasons

**Problem**: Lights don't turn off even though room is bright

**Solutions**:
1. Check if override is active:
   - Developer Tools → States → Check override entity
   - If state = "on", override is active (deactivate it)
2. Wait for debounce period:
   - Turn-OFF requires 5 minutes above 150 lux
   - During sunrise/sunset: 10 minutes required
3. Check lux level:
   - Must be > 150 lux to trigger turn-off
   - View lux sensor in Developer Tools

### Flicker Issues

**Problem**: Lights flicker on/off during cloudy days

**Solutions**:
1. This should NOT happen - hardcoded debounce prevents it
2. Check lux sensor:
   - Sensor may be in direct sunlight (rapid changes)
   - Sensor may be behind glass (affects readings)
   - Move sensor to measure ambient room light
3. Check sensor entity for rapid state changes:
   - Developer Tools → States → View history
   - If lux jumps rapidly (50 → 200 → 50), sensor placement issue
4. If still flickering after sensor repositioning:
   - Check sensor integration (may have issues)
   - Try different lux sensor

---

## Example Configurations

### Example 1: Master Bathroom

**Hardware**:
- Philips Hue White Ambiance bulbs (ceiling)
- Aqara Motion Sensor (includes lux)
- Virtual input_boolean for override

**Configuration**:
- **Zone Preset**: Quick (5/10/15 min)
- **Override**: `input_boolean.master_bath_override`
- **Circadian**: ENABLED
- **Staged Turn-Off**: ENABLED

**Use Case**:
- Short visits: Lights auto-off after 15 min
- Long showers: Activate override before shower
- Morning routine: Auto circadian color (warm early, cool later)

**Dashboard Setup**:
- Add toggle for `input_boolean.master_bath_override`
- Label: "Bathroom Extended Mode"

### Example 2: Hallway

**Hardware**:
- IKEA Tradfri tunable white bulbs
- Philips Hue Motion Sensor (includes lux)
- No override needed

**Configuration**:
- **Zone Preset**: Quick (5/10/15 min)
- **Override**: Disabled (not needed)
- **Circadian**: ENABLED
- **Staged Turn-Off**: ENABLED

**Use Case**:
- Transitional space (just passing through)
- Lights turn on when walking through
- Auto-off after 15 min if no motion
- Perfect color match to living room

### Example 3: Stairs

**Hardware**:
- Philips Hue White Ambiance bulbs
- Aqara Motion Sensor
- No override needed

**Configuration**:
- **Zone Preset**: Very Quick (3/5/7 min)
- **Override**: Disabled
- **Circadian**: ENABLED
- **Staged Turn-Off**: ENABLED

**Use Case**:
- Short transit time (ascending/descending)
- Lights turn on when detected
- Quick turn-off (7 min total) saves energy
- Warnings give time to react if still on stairs

### Example 4: Walk-in Closet

**Hardware**:
- IKEA Tradfri tunable white bulb
- Aqara Motion Sensor
- Physical button for override

**Configuration**:
- **Zone Preset**: Very Quick (3/5/7 min)
- **Override**: `input_boolean.closet_override` (toggled by button)
- **Circadian**: ENABLED
- **Staged Turn-Off**: ENABLED

**Use Case**:
- Quick access: Auto-off after 7 min
- Organizing clothes: Press button for override
- Circadian color matches bedroom

### Example 5: Garage

**Hardware**:
- Philips Hue White Ambiance bulbs
- mmWave occupancy sensor
- Virtual input_boolean for override

**Configuration**:
- **Zone Preset**: Medium (10/15/20 min)
- **Override**: `input_boolean.garage_override`
- **Circadian**: ENABLED
- **Staged Turn-Off**: ENABLED

**Use Case**:
- Parking car: Auto-off after 20 min
- Working on car: Activate override for extended time
- Retrieving items: Normal 20-min timing is sufficient

### Example 6: Laundry Room

**Hardware**:
- IKEA Tradfri tunable white bulbs
- PIR motion sensor with lux
- Virtual input_boolean for override

**Configuration**:
- **Zone Preset**: Medium (10/15/20 min)
- **Override**: `input_boolean.laundry_override`
- **Circadian**: ENABLED
- **Staged Turn-Off**: ENABLED

**Use Case**:
- Quick load: Auto-off after 20 min
- Folding clothes: Activate override
- Circadian color reduces eye strain during folding

### Example 7: Powder Room (Custom Timing)

**Hardware**:
- Philips Hue White Ambiance bulb
- Aqara Motion Sensor
- No override needed

**Configuration**:
- **Zone Preset**: Custom
- **Custom Stage 1**: 3 minutes
- **Custom Stage 2**: 6 minutes
- **Custom Stage 3**: 10 minutes
- **Override**: Disabled
- **Circadian**: ENABLED
- **Staged Turn-Off**: ENABLED

**Use Case**:
- Shorter timing than "Quick" preset
- Guests typically spend < 10 min
- Custom timing balances comfort + energy savings

---

## Advanced Customization

### Disable Circadian Rhythm

If you prefer constant color temperature:

1. Edit automation
2. **Enable Circadian Rhythm**: OFF
3. Save

Lights will use 3000K warm white (constant) instead of sun-aware color.

**Note**: This breaks color consistency with living room!

### Disable Staged Turn-Off

If you prefer instant turn-off:

1. Edit automation
2. **Enable Staged Turn-Off**: OFF
3. Save

Lights will turn off immediately when motion clears (based on Stage 3 timing).

### Change Update Frequency

If you want faster/slower circadian updates:

1. Edit automation
2. **Circadian Update Frequency**: Change from 60 seconds
   - Faster (30 sec): More responsive but more automation triggers
   - Slower (120 sec): Less responsive but fewer triggers
3. Save

**Warning**: If you change this, living room must match for color consistency!

### Change Transition Duration

If you want smoother/snappier transitions:

1. Edit automation
2. **Circadian Transition Duration**: Change from 4 seconds
   - Smoother (6-8 sec): Gentler color changes
   - Snappier (1-2 sec): Faster color changes
3. Save

**Warning**: If you change this, living room must match for color consistency!

### Multiple Zones with Same Blueprint

Create separate automations for each zone:

1. Create automation for Zone 1 (e.g., Hallway)
   - Name: `Hallway - Motion Circadian`
   - Configure for hallway
2. Create automation for Zone 2 (e.g., Bathroom)
   - Name: `Bathroom - Motion Circadian`
   - Configure for bathroom
3. Repeat for all zones

Each automation is independent but all share color consistency!

### Combine with Living Room Blueprint

For complete home coverage:

1. **Living Room**: Use `intelligent_living_room_mmwave_lux_aware.yaml`
2. **All Adjacent Zones**: Use `adjacent_zone_motion_circadian_lighting.yaml`

Result: Entire home has consistent circadian color!

---

## Appendix: Hardcoded Values Reference

For technical users, here are all hardcoded consistency parameters:

```yaml
# Lux thresholds (matches living room)
lux_on_threshold: 80          # Turn ON below 80 lux
lux_off_threshold: 150        # Turn OFF above 150 lux

# Debounce times (matches living room)
lux_on_debounce_sec: 120      # 2 minutes
lux_off_debounce_sec: 300     # 5 minutes
sunrise_sunset_debounce_sec: 600  # 10 minutes

# Staged turn-off brightness (matches living room)
stage_1_brightness_pct: 40    # First warning: 40%
stage_2_brightness_pct: 20    # Second warning: 20%

# Dynamic brightness curve (matches living room)
brightness_at_50_lux: 100     # 100% at/below 50 lux
brightness_at_75_lux: 85      # 85% at 75 lux
brightness_at_100_lux: 70     # 70% at 100 lux
brightness_at_125_lux: 55     # 55% at 125 lux
brightness_at_150_lux: 40     # 40% at 150 lux

# Circadian color (matches living room)
# See "Circadian Sun Elevation Formula" section for full code
# Range: 1800K (elevation < -6°) to 5500K (elevation > 50°)

# Warning color (matches living room)
warning_color_temp_kelvin: 1800  # 1800K warm (556 mireds)
```

These values CANNOT be changed in this blueprint - they are intentionally hardcoded for maximum consistency with the living room blueprint.

---

**Need more help?** See:
- [Quick Start Guide](./ADJACENT_ZONE_QUICK_START.md)
- [Color Consistency Guide](./COLOR_CONSISTENCY_GUIDE.md)
- [Living Room Setup Guide](./INTELLIGENT_LIVING_ROOM_SETUP_GUIDE.md)
- [Anti-Flicker Technical Guide](./ANTI_FLICKER_TECHNICAL_GUIDE.md)
