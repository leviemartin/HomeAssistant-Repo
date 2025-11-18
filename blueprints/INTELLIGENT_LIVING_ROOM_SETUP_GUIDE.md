# Intelligent Living Room Blueprint v1.1 - Complete Setup Guide

Comprehensive guide for setting up advanced lux-aware presence lighting with anti-flicker hysteresis, dynamic brightness, circadian rhythm, manual override, and scene cycling.

**Version**: 1.1.0
**Last Updated**: 2025-11-18
**Compatibility**: Home Assistant 2024.1+

## Table of Contents

1. [Overview](#overview)
2. [Hardware Requirements](#hardware-requirements)
3. [Installation](#installation)
4. [Basic Configuration](#basic-configuration)
5. [Advanced Features](#advanced-features)
6. [FP2 Configuration](#fp2-configuration)
7. [Override System Setup](#override-system-setup)
8. [Scene Cycling Override (NEW v1.1)](#scene-cycling-override-new-v11)
9. [Troubleshooting](#troubleshooting)
10. [Example Configurations](#example-configurations)

---

## Overview

This blueprint creates intelligent living room lighting that:

- **Lux-Aware Control**: Only activates when room is dark enough (configurable thresholds)
- **Anti-Flicker Hysteresis**: Dual-threshold system (80/150 lux) with debounce prevents oscillation
- **Dynamic Brightness**: Brightness scales inversely with natural light (brighter outside = dimmer lights)
- **Circadian Rhythm**: Sun-aware color temperature (1800K warm to 5500K cool) based on sun elevation
- **Staged Turn-Off**: Progressive warnings (30/40/45 min) with dimming + warm color before complete off
- **Manual Override**: Movie mode via button/switch/input_boolean keeps lights on regardless of lux
- **Sunrise/Sunset Intelligence**: Extended debounce during dawn/dusk transitions (10 min vs 2-5 min)
- **Approach Detection**: Optional instant activation when approaching room (FP2 feature)

### Key Innovations

**Hysteresis Anti-Flicker System**
```
State: LIGHTS OFF
- Lux < 80 for 2 minutes → Lights can turn ON (if presence detected)

State: LIGHTS ON
- Lux > 150 for 5 minutes → Lights turn OFF

Hysteresis Band (80-150 lux): No automatic changes
```

**Dynamic Brightness Curve**
```
≤50 lux:   100% brightness (very dark)
75 lux:    85% brightness
100 lux:   70% brightness
125 lux:   55% brightness
150 lux:   40% brightness (at OFF threshold)
>150 lux:  Lights turn off (after debounce)
```

**Circadian Color Temperature (Sun-Aware)**
```
Sun elevation < -6°:      1800K (deep night - amber warmth)
Sun elevation -6° to 0°:  1800K to 3000K (civil twilight)
Sun elevation 0° to 10°:  3000K to 3500K (sunrise/sunset)
Sun elevation 10° to 30°: 3500K to 5000K (rising/falling sun)
Sun elevation 30° to 50°: 5000K to 5500K (high sun)
Sun elevation > 50°:      5500K (peak daylight)
```

---

## Hardware Requirements

### Required Components

1. **Tunable White Lights**
   - Philips Hue (White Ambiance, White & Color Ambiance)
   - IKEA Tradfri (tunable white bulbs)
   - Any smart bulb with `color_temp` support
   - **Note**: Must support mireds (micro reciprocal degrees) color temperature

2. **Presence Sensor**
   - Aqara FP2 mmWave sensor (recommended - best accuracy)
   - Any mmWave presence sensor (Screek, HLK-LD2410, etc.)
   - PIR motion sensors (works but less accurate)
   - **Integration**: Must expose `binary_sensor.xxx_presence` entity

3. **Lux/Illuminance Sensor**
   - FP2's built-in lux sensor (convenient if using FP2)
   - Separate Xiaomi/Aqara lux sensor
   - Philips Hue Motion Sensor (includes lux)
   - Any sensor with `sensor.xxx_illuminance` entity

### Optional Components

4. **Approach Detection Sensor** (FP2 Only)
   - Configured via Aqara Home app
   - Exposes `binary_sensor.xxx_approach` entity
   - See [FP2 Configuration](#fp2-configuration) section

5. **Override Trigger** (For Manual Control)
   - Aqara Wireless Button (WXKG11LM, WXKG12LM, WXKG13LM)
   - Philips Hue Dimmer Switch
   - IKEA Tradfri Remote
   - Virtual `input_boolean` helper
   - Any `binary_sensor` or `switch` entity

---

## Installation

### Method 1: Import via URL (Recommended)

1. Open Home Assistant web interface
2. Navigate to **Settings** → **Automations & Scenes** → **Blueprints**
3. Click **Import Blueprint** (bottom right corner)
4. Paste the blueprint URL:
   ```
   https://github.com/martinlevie/HomeAssistant-Repo/blueprints/intelligent_living_room_mmwave_lux_aware.yaml
   ```
5. Click **Preview Blueprint**
6. Review the blueprint details
7. Click **Import Blueprint**

### Method 2: Manual Installation

1. Download `intelligent_living_room_mmwave_lux_aware.yaml`
2. Copy to Home Assistant config directory:
   ```
   /config/blueprints/automation/intelligent_living_room_mmwave_lux_aware.yaml
   ```
3. Restart Home Assistant or reload automations:
   - **Developer Tools** → **YAML** → **Reload Automations**

### Verify Installation

1. Navigate to **Settings** → **Automations & Scenes** → **Blueprints**
2. Look for "Intelligent Living Room - mmWave + Lux-Aware v1.0"
3. If visible, installation successful!

---

## Basic Configuration

### Step 1: Create the Automation

1. Navigate to **Settings** → **Automations & Scenes** → **Blueprints**
2. Find the blueprint and click **Create Automation**
3. Fill in the required fields:

### Required Inputs

#### Target Lights
**What to select**: Your living room lights
```yaml
Examples:
- light.living_room_ceiling
- light.living_room_group
- Multiple lights via area or group
```

**Tips**:
- Use groups for easier management
- All lights must support color_temp
- Verify lights respond to mireds (most do)

#### Primary Presence Sensor
**What to select**: Main room presence detection
```yaml
Examples:
- binary_sensor.fp2_living_room_presence
- binary_sensor.living_room_mmwave_presence
- binary_sensor.living_room_motion (PIR fallback)
```

**Tips**:
- FP2 recommended (no false negatives)
- PIR sensors may miss stationary presence
- Verify sensor updates reliably

#### Lux/Illuminance Sensor
**What to select**: Ambient light measurement
```yaml
Examples:
- sensor.fp2_living_room_light_sensor
- sensor.living_room_lux
- sensor.hue_motion_living_room_illuminance
```

**Tips**:
- Place sensor near window (not direct sunlight)
- Represents overall room brightness
- Check Developer Tools → States for current reading

### Recommended Default Settings (Out-of-Box)

Leave these at defaults for initial testing:

| Setting | Default Value | What It Does |
|---------|--------------|--------------|
| **Lux Turn-ON Threshold** | 80 lux | Lights turn on below this (dark enough) |
| **Lux Turn-OFF Threshold** | 150 lux | Lights turn off above this (bright enough) |
| **Turn-ON Debounce** | 2 minutes | Wait before allowing turn-on |
| **Turn-OFF Debounce** | 5 minutes | Wait before turning off (prevents flicker) |
| **Sunrise/Sunset Debounce** | 10 minutes | Extended wait during dawn/dusk |
| **Dynamic Brightness** | ENABLED | Brightness scales with lux |
| **Circadian Rhythm** | ENABLED | Color temp follows sun elevation |
| **Staged Turn-Off** | ENABLED | 30/40/45 min progressive warnings |
| **Manual Override** | ENABLED | Movie mode support |

### Step 2: Save and Name

1. Click **Save** (bottom right)
2. Enter automation name: `Living Room - Intelligent Lighting`
3. Optionally assign to area: `Living Room`

### Step 3: Initial Testing

#### Test 1: Basic Presence Detection
1. **Setup**: Close curtains (make room dark < 80 lux)
2. **Action**: Walk into room
3. **Expected**: Lights turn on within 2 minutes
4. **Verify**: Brightness appropriate for lux level

#### Test 2: Lux-Aware Turn-Off
1. **Setup**: Lights are on, presence active
2. **Action**: Open curtains (make room bright > 150 lux)
3. **Expected**: Lights dim gradually then turn off after 5 minutes
4. **Verify**: No flickering or oscillation

#### Test 3: Staged Turn-Off
1. **Setup**: Lights are on
2. **Action**: Leave room (clear presence)
3. **Expected Timeline**:
   - **30 minutes**: Lights dim to 40% + shift to 1800K warm
   - **40 minutes**: Lights dim to 20% (still 1800K)
   - **45 minutes**: Lights turn off with 3-second fade
4. **Verify**: Re-entering room before 45 min restarts automation

#### Test 4: Circadian Color Temperature
1. **Setup**: Lights are on
2. **Action**: Observe color temperature throughout day
3. **Expected**:
   - **Morning (sunrise)**: Warm tones (2500-3500K)
   - **Midday**: Cool daylight (5000-5500K)
   - **Evening (sunset)**: Warm tones (3000-3500K)
   - **Night**: Very warm amber (1800-2200K)
4. **Verify**: Smooth transitions every 60 seconds

---

## Advanced Features

### Dynamic Brightness Customization

Fine-tune brightness curve for your specific room and lighting:

#### Understanding the Curve

The blueprint uses 5 control points to create a smooth brightness curve:

```
Lux Level → Brightness
≤50 lux   → 100% (default) - Very dark (night, curtains closed)
75 lux    → 85% (default)  - Dark (overcast day, curtains closed)
100 lux   → 70% (default)  - Medium (cloudy day, curtains open)
125 lux   → 55% (default)  - Bright (partly sunny)
150 lux   → 40% (default)  - Very bright (at OFF threshold)
```

Linear interpolation fills gaps between control points.

#### Customization Examples

**Scenario 1: Room stays bright (south-facing windows)**
```yaml
Brightness at ≤50 Lux:  100%
Brightness at 75 Lux:   75%
Brightness at 100 Lux:  60%
Brightness at 125 Lux:  45%
Brightness at 150 Lux:  30%
```
Effect: Lights dim more aggressively as natural light increases

**Scenario 2: Darker room (north-facing, few windows)**
```yaml
Brightness at ≤50 Lux:  100%
Brightness at 75 Lux:   95%
Brightness at 100 Lux:  85%
Brightness at 125 Lux:  70%
Brightness at 150 Lux:  55%
```
Effect: Lights stay brighter even when some natural light present

**Scenario 3: Energy-saving mode**
```yaml
Brightness at ≤50 Lux:  80%
Brightness at 75 Lux:   65%
Brightness at 100 Lux:  50%
Brightness at 125 Lux:  35%
Brightness at 150 Lux:  25%
```
Effect: Reduced brightness across all lux levels (saves energy)

#### How to Adjust

1. Edit your automation
2. Scroll to **Dynamic Brightness Settings**
3. Adjust the 5 brightness sliders
4. Save and test
5. Fine-tune based on preference

### Lux Threshold Tuning

#### Finding Your Ideal Thresholds

1. **Morning Test (Curtains Closed)**
   - Check lux: Developer Tools → States → Your lux sensor
   - Typical: 20-60 lux
   - Note value: `_____ lux`

2. **Afternoon Test (Curtains Open, Cloudy)**
   - Check lux reading
   - Typical: 100-200 lux
   - Note value: `_____ lux`

3. **Afternoon Test (Curtains Open, Sunny)**
   - Check lux reading
   - Typical: 300-800 lux
   - Note value: `_____ lux`

4. **Calculate Thresholds**
   ```
   Turn-ON Threshold = Morning lux - 10 lux
   Example: 50 lux (morning) - 10 = 40 lux turn-on

   Turn-OFF Threshold = Turn-ON + 70 to 100 lux
   Example: 40 lux (turn-on) + 80 = 120 lux turn-off
   ```

#### Hysteresis Gap Guidelines

The gap between ON and OFF thresholds is critical:

| Gap Size | Behavior | Best For |
|----------|----------|----------|
| **30-50 lux** | Responsive, may flicker on cloudy days | Stable lighting (office buildings) |
| **70-100 lux** | **Recommended** - Balanced stability | Most living rooms |
| **100-150 lux** | Very stable, slower response | Rooms with variable cloud cover |
| **150+ lux** | Maximum stability, delayed response | Testing/troubleshooting |

**Rule of Thumb**: Maintain at least 50 lux gap. Increase if experiencing flicker.

#### Debounce Timing Guidelines

| Scenario | Turn-ON Debounce | Turn-OFF Debounce | Sunrise/Sunset Debounce |
|----------|------------------|-------------------|------------------------|
| **Stable climate** | 1-2 minutes | 3-5 minutes | 5-8 minutes |
| **Cloudy region (default)** | 2-3 minutes | 5-7 minutes | 10-12 minutes |
| **Variable weather** | 3-5 minutes | 7-10 minutes | 12-15 minutes |
| **Testing/troubleshooting** | 30 seconds | 1 minute | 2 minutes |

**Trade-offs**:
- **Shorter debounce**: More responsive, higher risk of flicker
- **Longer debounce**: More stable, slower adaptation

### Circadian Rhythm Customization

#### Update Frequency

Controls how often color temperature updates:

```yaml
30 seconds:  Very responsive (more automation triggers)
60 seconds:  Recommended balance
120 seconds: Less frequent updates (saves resources)
300 seconds: Minimal updates (5 minutes)
```

**When to increase frequency**:
- Sunrise/sunset periods (rapid sun movement)
- You notice abrupt color shifts
- Testing circadian behavior

**When to decrease frequency**:
- Resource-constrained systems
- Color temp changes feel too frequent
- Prefer slower, gentler transitions

#### Transition Duration

Controls smoothness of color changes:

```yaml
1 second:   Instant snap (not recommended)
2-3 seconds: Quick but smooth
4 seconds:   Recommended default (balanced)
5-7 seconds: Gentle, barely noticeable
10 seconds:  Very slow, theatrical
```

**Best practice**: Match transition duration to update frequency
- 60-second updates → 3-5 second transitions
- 120-second updates → 5-7 second transitions

#### Sun Elevation Mapping (Advanced)

If you want to customize the sun elevation to color temp mapping, edit the blueprint variables section:

```yaml
# Example: Prefer warmer tones overall
color_temp_kelvin: >
  {% if circadian_rhythm %}
    {% set elevation = sun_elevation | float %}

    {% if elevation < -6 %}
      1800  # Deep night - very warm amber
    {% elif elevation >= -6 and elevation < 0 %}
      {{ (1800 + (elevation + 6) * 150) | int }}  # Slower warm-up
    {% elif elevation >= 0 and elevation < 10 %}
      {{ (2700 + (elevation * 40)) | int }}  # Gentler morning
    ...
  {% endif %}
```

**Customization ideas**:
- Start warmer in morning (increase baseline)
- Stay warmer in evening (reduce sunset cooling)
- Limit maximum coolness (cap at 4500K instead of 5500K)
- Accelerate/decelerate transitions (adjust multipliers)

### Staged Turn-Off Customization

Perfect for preventing jarring darkness when you leave a room.

#### Timing Customization

Default timeline:
```
0 min:     Presence cleared - automation starts
30 min:    Stage 1 - Dim to 40% + Warm to 1800K
40 min:    Stage 2 - Dim to 20% + Keep 1800K
45 min:    Stage 3 - Turn off completely (3-sec fade)
```

#### Example Configurations

**Quick Turn-Off (Hallway, Bathroom)**
```yaml
Stage 1 Delay: 5 minutes   (300 seconds)
Stage 1 Brightness: 30%
Stage 2 Delay: 8 minutes   (480 seconds)
Stage 2 Brightness: 15%
Stage 3 Delay: 10 minutes  (600 seconds)
```

**Standard Turn-Off (Living Room Default)**
```yaml
Stage 1 Delay: 30 minutes  (1800 seconds)
Stage 1 Brightness: 40%
Stage 2 Delay: 40 minutes  (2400 seconds)
Stage 2 Brightness: 20%
Stage 3 Delay: 45 minutes  (2700 seconds)
```

**Extended Turn-Off (Office, Workshop)**
```yaml
Stage 1 Delay: 60 minutes  (3600 seconds)
Stage 1 Brightness: 50%
Stage 2 Delay: 75 minutes  (4500 seconds)
Stage 2 Brightness: 30%
Stage 3 Delay: 90 minutes  (5400 seconds)
```

**Single-Stage Turn-Off (Simplified)**
```yaml
Stage 1 Delay: 15 minutes  (900 seconds)
Stage 1 Brightness: 25%
Stage 2 Delay: 17 minutes  (1020 seconds)  # Only 2-min gap
Stage 2 Brightness: 25%    # Same as Stage 1
Stage 3 Delay: 20 minutes  (1200 seconds)
```

#### Brightness Warning Levels

| Brightness | Visibility | Best For |
|------------|------------|----------|
| **50-60%** | Clearly visible | Gentle reminder |
| **30-40%** | Noticeable dim | Standard warning |
| **15-25%** | Obvious warning | Strong signal to return |
| **5-10%** | Very dim | Last chance warning |

**Tip**: Larger brightness drops are more noticeable. Consider:
- Stage 1: 50% (gentle)
- Stage 2: 20% (clear signal)

#### Warning Color Temperature

Fixed at 1800K warm amber during stages - this creates a distinct visual cue that's different from normal circadian colors.

**Why 1800K?**
- Noticeably different from normal operation
- Warm tone is less jarring than sudden brightness change
- Easy to recognize as "warning mode"
- Still functional if you return to room

---

## FP2 Configuration

Detailed Aqara FP2 setup for optimal performance.

### FP2 Integration Setup

1. **Add FP2 to Home Assistant**
   - Method 1: Aqara Home Integration (recommended)
     - Settings → Devices & Services → Add Integration
     - Search "Aqara Home" → Sign in → Select FP2
   - Method 2: Zigbee2MQTT (advanced)
     - Pair FP2 via Z2M
     - Entities auto-created

2. **Verify FP2 Entities**
   Check Developer Tools → States for:
   ```
   binary_sensor.fp2_living_room_presence     ✓ Required
   sensor.fp2_living_room_light_sensor        ✓ Required
   binary_sensor.fp2_living_room_approach     ⚬ Optional
   ```

### FP2 Presence Zone Configuration

**Aqara Home App Setup**:

1. Open Aqara Home app
2. Select your FP2 device
3. Tap **Zone Settings**
4. **Recommended for Living Room**:
   - **Mode**: Whole Room Detection (simple, reliable)
   - **Zones**: 1 zone covering entire room
   - **Sensitivity**: High (for stationary detection)

**Why Whole Room?**
- Blueprint treats room as single presence zone
- No complex zone mapping needed
- Simpler setup, fewer false negatives
- Zone-specific features can be added later if needed

### FP2 Approach Detection Setup

Approach detection triggers lights instantly when walking toward room.

**Aqara Home App Configuration**:

1. Open Aqara Home app → Select FP2
2. Tap **Approach Detection**
3. Enable **Approach Detection**: ON
4. Configure detection zone:
   - **Zone**: Doorway/entrance area
   - **Distance**: 0.5m to 2m from door
   - **Angle**: 60-90 degrees (covers entrance)
5. Save settings

**Verify in Home Assistant**:
```
Developer Tools → States
Search: binary_sensor.fp2_living_room_approach
State should toggle when approaching room
```

**Add to Blueprint**:
1. Edit automation
2. Set **Approach Detection Sensor**: `binary_sensor.fp2_living_room_approach`
3. Save

**Behavior**:
- Approach detected → Lights turn on instantly
- Bypasses lux check temporarily
- Uses current circadian color and dynamic brightness
- Seamlessly integrates with main presence automation

### FP2 Lux Sensor Placement

**Optimal placement for FP2 lux sensor**:
- **Near window** but not in direct sunlight
- **Ceiling height** (FP2's standard mounting)
- **Represents overall room brightness** (not corner shadows)

**Calibration check**:
1. Compare FP2 lux readings to known conditions:
   - **Curtains closed (night)**: Should read 0-20 lux
   - **Curtains closed (day)**: Should read 20-80 lux
   - **Curtains open (cloudy)**: Should read 100-300 lux
   - **Curtains open (sunny)**: Should read 300-1000+ lux
2. If readings seem off, recalibrate:
   - Aqara Home app → FP2 settings → Light Sensor → Calibrate

### FP2 Sensitivity Tuning

**Presence Detection Sensitivity**:
- **Low**: Only detects obvious movement
- **Medium**: Balanced (recommended start)
- **High**: Detects subtle movement (sitting, typing)
- **Very High**: Maximum sensitivity (may have false positives)

**For living room use**:
- Start with **High** (detects TV watching, reading)
- Reduce to Medium if false triggers occur
- Increase to Very High if missing stationary presence

**Aqara Home App**:
1. Select FP2 → **Detection Settings**
2. Adjust **Presence Sensitivity**
3. Test by sitting still for 5 minutes
4. Verify presence sensor stays "on"

---

## Override System Setup

Manual control to keep lights on regardless of lux level (movie mode, party mode, manual preference).

### Option 1: Virtual Input Boolean (Simplest)

**Step 1: Create Helper**
1. Settings → Devices & Services → Helpers
2. Click **Create Helper**
3. Select **Toggle**
4. Configure:
   ```
   Name: Living Room Override
   Icon: mdi:movie-open (or mdi:lightbulb-on)
   ```
5. Click **Create**

**Step 2: Add to Automation**
1. Edit your living room automation
2. **Enable Manual Override System**: ON
3. **Override Trigger Entity**: `input_boolean.living_room_override`
4. Save

**Step 3: Add to Dashboard**
1. Edit dashboard → Add Card → Entity
2. Select `input_boolean.living_room_override`
3. Customize appearance:
   ```yaml
   Name: Movie Mode
   Icon: mdi:movie-open
   Tap Action: Toggle
   ```

**Usage**:
- Toggle ON → Lights stay on regardless of lux
- Toggle OFF → Resume normal lux-based automation

### Option 2: Aqara Wireless Button (Physical)

**Supported Aqara Buttons**:
- WXKG11LM (original round button)
- WXKG12LM (revised round button)
- WXKG13LM (square button - recommended)

**Step 1: Pair Button**
1. Add via Aqara Home integration or Zigbee2MQTT
2. Verify entity: `sensor.aqara_button_action`

**Step 2: Create Automation**

Create a SEPARATE automation to toggle override:

```yaml
alias: Living Room Override - Toggle via Button
trigger:
  - platform: state
    entity_id: sensor.aqara_button_action
    to: "single"  # Single press
action:
  - service: input_boolean.toggle
    target:
      entity_id: input_boolean.living_room_override
mode: single
```

**Step 3: Link to Main Automation**
1. Edit living room automation
2. **Enable Manual Override System**: ON
3. **Override Trigger Entity**: `input_boolean.living_room_override`
4. Save

**Usage**:
- Single press button → Toggles override state
- Button LED feedback (if equipped)
- Works even when away from dashboard

### Option 3: Philips Hue Dimmer Switch

**Step 1: Pair Switch**
1. Add via Philips Hue integration
2. Verify entities:
   ```
   sensor.hue_dimmer_living_room_button
   ```

**Step 2: Create Toggle Automation**

```yaml
alias: Living Room Override - Hue Dimmer Toggle
trigger:
  - platform: state
    entity_id: sensor.hue_dimmer_living_room_button
    to: "on_press"  # "ON" button
action:
  - service: input_boolean.toggle
    target:
      entity_id: input_boolean.living_room_override
mode: single
```

**Step 3: Link to Main Automation**
Same as Aqara button (link to input_boolean)

### Option 4: Double-Tap Physical Light Switch (Advanced)

Uses existing wall switch with double-tap detection.

**Requirements**:
- Smart switch with scene control (e.g., Inovelli, GE/Jasco)
- Exposes `event` or `sensor.xxx_action` entity

**Example with Inovelli Red Series**:

```yaml
alias: Living Room Override - Double Tap Toggle
trigger:
  - platform: event
    event_type: zwave_js_value_notification
    event_data:
      node_id: 23  # Your switch node ID
      value: KeyPressed2x  # Double-tap up
action:
  - service: input_boolean.toggle
    target:
      entity_id: input_boolean.living_room_override
mode: single
```

### Override Behavior

**When Override is ACTIVE**:
- ✅ Lights turn on/stay on regardless of lux level
- ✅ Presence detection still works
- ✅ Dynamic brightness still active (can be disabled if preferred)
- ✅ Circadian rhythm still updates color temp
- ❌ Lux turn-off threshold ignored
- ❌ Lights won't turn off due to bright natural light

**When Override is INACTIVE**:
- ✅ Normal lux-based operation resumes
- ✅ Hysteresis and debounce active
- ✅ All anti-flicker protections work

**Visual Indicator Ideas**:

Add a conditional card to dashboard showing override status:

```yaml
type: conditional
conditions:
  - entity: input_boolean.living_room_override
    state: "on"
card:
  type: markdown
  content: |
    🎬 **Movie Mode Active**
    Lights will stay on regardless of natural light.
    Tap below to deactivate.
  title: Override Active
```

---

## Scene Cycling Override (NEW v1.1)

Advanced scene cycling system allowing you to cycle through up to 3 Philips Hue scenes with a single button press. Perfect for Movie Mode, Reading Mode, and Party Mode presets.

### Overview

The scene cycling feature allows you to:
- Press button once: Activate Scene 1 (e.g., Movie Mode)
- Press button again: Activate Scene 2 (e.g., Reading Mode)
- Press button again: Activate Scene 3 (e.g., Party Mode)
- Press button again: Return to normal automation

**Key Features**:
- Scenes override lux-based control and circadian lighting
- Configurable presence timeout bypass (keep scene on even when room empty)
- Each scene can be enabled/disabled independently
- State persists across Home Assistant restarts
- Fully backward compatible (optional feature)

### Prerequisites

Before setting up scene cycling, you need:

1. **Philips Hue Scenes** (or Home Assistant scenes)
   - Created via Philips Hue app or HA Scenes interface
   - Automatically synced to Home Assistant
   - Must be entity domain: `scene`

2. **Input Number Helper** (for state tracking)
   - Create via Settings > Helpers
   - Stores current scene state (0-3)

3. **Button/Switch** (for cycling)
   - Physical button (Hue dimmer, Aqara button)
   - Virtual input_button or input_boolean
   - Any entity that can trigger on state change

### Step 1: Create Philips Hue Scenes

#### Option A: Via Philips Hue App (Recommended)

1. Open Philips Hue app on your phone
2. Navigate to **Scenes** tab
3. Tap **Create Scene** (+ icon)
4. Configure each scene:

**Scene 1: Movie Mode**
```
Name: Living Room Movie
Lights: Select all living room lights
Brightness: 30-40%
Color: Warm white (2000-2500K) or dim orange
Save scene
```

**Scene 2: Reading Mode**
```
Name: Living Room Reading
Lights: Select all living room lights
Brightness: 80-100%
Color: Neutral white (3500-4000K)
Save scene
```

**Scene 3: Party Mode**
```
Name: Living Room Party
Lights: Select all living room lights
Brightness: 100%
Color: Dynamic/colorful (use color loop or vibrant colors)
Save scene
```

5. Wait 1-2 minutes for scenes to sync to Home Assistant
6. Verify scenes in HA:
   - Go to **Developer Tools** > **States**
   - Search for `scene.living_room_movie`
   - Verify all 3 scenes appear

#### Option B: Via Home Assistant Scenes

1. Navigate to **Settings** > **Automations & Scenes** > **Scenes**
2. Click **Create Scene** (bottom right)
3. Configure scene:
   ```yaml
   Name: Living Room Movie Mode
   Entities:
     - light.living_room_ceiling:
         state: on
         brightness: 40
         color_temp: 454  # ~2200K warm
     - light.living_room_lamp:
         state: on
         brightness: 30
         color_temp: 454
   ```
4. Save scene
5. Repeat for all 3 scenes

**Scene Naming Tips**:
- Use descriptive names: "Living Room Movie" not just "Movie"
- Include room name for clarity with multiple rooms
- Keep names short (under 25 characters)

### Step 2: Create Input Number Helper (State Tracker)

This helper tracks which scene is currently active.

1. Navigate to **Settings** > **Devices & Services** > **Helpers**
2. Click **Create Helper** (bottom right)
3. Select **Number**
4. Configure:
   ```
   Name: Living Room Scene State
   Icon: mdi:lightbulb-multiple (or mdi:movie-open)
   Minimum value: 0
   Maximum value: 3
   Step: 1
   Display mode: Auto (or Box)
   Unit of measurement: (leave empty)
   ```
5. Click **Create**
6. Verify entity created: `input_number.living_room_scene_state`

**What the values mean**:
- `0` = No scene active (normal automation)
- `1` = Scene 1 active (Movie Mode)
- `2` = Scene 2 active (Reading Mode)
- `3` = Scene 3 active (Party Mode)

**Optional: Add to Dashboard**

Add the helper to your dashboard to see current scene state:

```yaml
type: entity
entity: input_number.living_room_scene_state
name: Active Scene
icon: mdi:lightbulb-multiple
```

This shows which scene is currently active (useful for debugging).

### Step 3: Configure Scene Cycle Button

Choose one of these methods for your cycle button:

#### Option A: Philips Hue Dimmer Switch (Physical)

**Best for**: Wall-mounted convenience, tactile feedback

1. Ensure Hue Dimmer is paired to Home Assistant
2. Verify entity exists:
   ```
   Developer Tools > States > Search: hue_dimmer
   Look for: sensor.hue_dimmer_living_room_button
   ```
3. Note the button entity for blueprint configuration
4. Button will trigger on any press (on, off, up, down)

**Recommended button mapping**:
- Use "ON" button for scene cycling
- Use "OFF" button for manual override toggle
- Use "UP/DOWN" buttons for brightness adjustment (separate automation)

#### Option B: Aqara Wireless Button (Physical)

**Best for**: Portable control, battery-powered convenience

1. Pair Aqara button via Aqara Home or Zigbee2MQTT
2. Verify entity:
   ```
   Developer Tools > States
   Search: aqara_button or wireless_button
   Look for: event.aqara_button_action or sensor.aqara_button_action
   ```
3. Note the entity for blueprint configuration

**Supported Aqara buttons**:
- WXKG11LM (original round button)
- WXKG12LM (revised round button)
- WXKG13LM (square button - recommended)

#### Option C: Virtual Input Button (Dashboard)

**Best for**: Testing, dashboard-only control

1. Settings > Devices & Services > Helpers
2. Create Helper > Button
3. Configure:
   ```
   Name: Living Room Scene Cycle
   Icon: mdi:cached
   ```
4. Add to dashboard:
   ```yaml
   type: button
   entity: input_button.living_room_scene_cycle
   name: Cycle Scene
   icon: mdi:movie-open
   tap_action:
     action: perform-action
     perform_action: input_button.press
   ```

#### Option D: Virtual Input Boolean (Toggle)

**Best for**: Two-way state tracking

1. Settings > Devices & Services > Helpers
2. Create Helper > Toggle
3. Configure:
   ```
   Name: Living Room Scene Cycle Trigger
   Icon: mdi:cached
   ```
4. Use in blueprint (toggles on each press)

### Step 4: Blueprint Configuration

Now configure the blueprint with scene cycling:

1. Edit your existing Living Room automation
2. Scroll to **SECTION 4B: SCENE CYCLING OVERRIDE SYSTEM**
3. Configure inputs:

#### Enable Scene Cycling
```yaml
Enable Scene Cycling System: ON
```

#### State Tracker
```yaml
Scene State Tracker Helper: input_number.living_room_scene_state
```

#### Cycle Button
```yaml
Scene Cycle Button:
  - sensor.hue_dimmer_living_room_button (Hue dimmer)
  - event.aqara_button_action (Aqara button)
  - input_button.living_room_scene_cycle (virtual button)
```

#### Scene 1 Configuration
```yaml
Enable Scene 1: ON
Scene 1 Entity: scene.living_room_movie
Scene 1 Name: Movie Mode
```

#### Scene 2 Configuration
```yaml
Enable Scene 2: ON
Scene 2 Entity: scene.living_room_reading
Scene 2 Name: Reading Mode
```

#### Scene 3 Configuration
```yaml
Enable Scene 3: ON
Scene 3 Entity: scene.living_room_party
Scene 3 Name: Party Mode
```

#### Scene Behavior
```yaml
Scene Bypass Presence Timeout:
  - OFF (default) - Scene respects 30/40/45 min turn-off when presence clears
  - ON - Scene stays on indefinitely until manually cycled off
```

**Recommended setting**:
- **OFF** for safety (lights still turn off if you forget)
- **ON** for movie nights (lights stay on even during quiet scenes)

4. Click **Save**

### Step 5: Testing Scene Cycling

#### Test 1: Basic Cycling

1. **Initial state**: Lights should be in normal automation mode
2. **Press button once**: Scene 1 activates (Movie Mode)
   - Verify lights change to Scene 1 settings
   - Check `input_number.living_room_scene_state` = 1
3. **Press button again**: Scene 2 activates (Reading Mode)
   - Verify lights change to Scene 2 settings
   - Check state tracker = 2
4. **Press button again**: Scene 3 activates (Party Mode)
   - Verify lights change to Scene 3 settings
   - Check state tracker = 3
5. **Press button again**: Return to normal automation
   - Verify lights transition to circadian color/brightness
   - Check state tracker = 0

**Expected behavior**:
- Smooth 2-second transitions between scenes
- State tracker updates immediately
- Lights reflect scene settings instantly

#### Test 2: Scene Override Behavior

**Scenario**: Scene active, test lux-based turn-off

1. Activate Scene 1 (Movie Mode)
2. Open curtains (increase lux to 300+, above OFF threshold)
3. Wait 5+ minutes (normal debounce period)
4. **Expected**: Lights stay on (scene ignores lux)
5. **Verify**: Lux-based turn-off is bypassed

**Scenario**: Scene active, test circadian updates

1. Activate Scene 2 (Reading Mode)
2. Wait 60+ seconds (circadian update interval)
3. **Expected**: Lights stay at Scene 2 settings (no color change)
4. **Verify**: Circadian rhythm updates are skipped

**Scenario**: Scene active, presence clears

1. Activate Scene 1 (Movie Mode)
2. Leave room (clear presence)
3. **If bypass OFF**:
   - 30 min: Lights dim to Stage 1 (40%, 1800K) - scene overridden
   - 40 min: Lights dim to Stage 2 (20%, 1800K)
   - 45 min: Lights turn off
4. **If bypass ON**:
   - Lights stay at Scene 1 settings indefinitely
   - No staged turn-off occurs

#### Test 3: Disabled Scene Skipping

**Scenario**: Scene 2 disabled, test cycling

1. Edit automation: Set **Enable Scene 2** = OFF
2. Save automation
3. Press button from state 0 (off)
4. **Expected**: Scene 1 activates (state = 1)
5. Press button again
6. **Expected**: Scene 3 activates (state = 3, skips disabled Scene 2)
7. Press button again
8. **Expected**: Return to off (state = 0)

**Verify**: Disabled scenes are automatically skipped in cycle

#### Test 4: State Persistence

1. Activate Scene 2 (Reading Mode)
2. Restart Home Assistant:
   - Settings > System > Restart
3. Wait for restart to complete
4. **Expected**: Scene 2 still active (state tracker = 2)
5. **Verify**: Lights maintain Scene 2 settings after restart

### Advanced Configuration

#### Customizing Scene Behavior

**Use Case 1: Movie Night (Bypass Enabled)**
```yaml
Scene 1 (Movie Mode):
  Scene: scene.living_room_movie (dim, warm)
  Bypass Presence Timeout: ON

Result: Lights stay dim throughout movie, even during quiet scenes
when presence sensor might not detect small movements.
```

**Use Case 2: Reading Time (Bypass Disabled)**
```yaml
Scene 2 (Reading Mode):
  Scene: scene.living_room_reading (bright, focused)
  Bypass Presence Timeout: OFF

Result: Lights stay bright while reading, but turn off after 45 min
if you fall asleep or leave the room.
```

**Use Case 3: Party Mode (Bypass Enabled)**
```yaml
Scene 3 (Party Mode):
  Scene: scene.living_room_party (colorful, dynamic)
  Bypass Presence Timeout: ON

Result: Party lights stay on throughout event, even if people
temporarily move to other rooms.
```

#### Conditional Scene Cycling (Advanced)

Create smart automations that auto-activate scenes:

**Auto Movie Mode at Night**:
```yaml
alias: Auto Movie Mode at Night
trigger:
  - platform: time
    at: "21:00:00"  # 9 PM
condition:
  - condition: state
    entity_id: binary_sensor.living_room_presence
    state: "on"
  - condition: state
    entity_id: media_player.living_room_tv
    state: "playing"
action:
  # Set scene tracker to 1 (Movie Mode)
  - service: input_number.set_value
    target:
      entity_id: input_number.living_room_scene_state
    data:
      value: 1
  # Activate Movie Mode scene
  - service: scene.turn_on
    target:
      entity_id: scene.living_room_movie
```

**Auto Reading Mode in Morning**:
```yaml
alias: Auto Reading Mode Weekends
trigger:
  - platform: time
    at: "08:00:00"
condition:
  - condition: state
    entity_id: binary_sensor.weekend
    state: "on"
  - condition: state
    entity_id: binary_sensor.living_room_presence
    state: "on"
action:
  - service: input_number.set_value
    target:
      entity_id: input_number.living_room_scene_state
    data:
      value: 2
  - service: scene.turn_on
    target:
      entity_id: scene.living_room_reading
```

#### Dashboard Scene Control

Create advanced dashboard controls:

**Scene Selector Card**:
```yaml
type: entities
title: Living Room Scenes
entities:
  - entity: input_number.living_room_scene_state
    name: Current Scene
    secondary_info: last-changed

  - type: button
    name: Movie Mode
    icon: mdi:movie-open
    tap_action:
      action: perform-action
      perform_action: input_number.set_value
      data:
        entity_id: input_number.living_room_scene_state
        value: 1

  - type: button
    name: Reading Mode
    icon: mdi:book-open-page-variant
    tap_action:
      action: perform-action
      perform_action: input_number.set_value
      data:
        entity_id: input_number.living_room_scene_state
        value: 2

  - type: button
    name: Party Mode
    icon: mdi:party-popper
    tap_action:
      action: perform-action
      perform_action: input_number.set_value
      data:
        entity_id: input_number.living_room_scene_state
        value: 3

  - type: button
    name: Normal Automation
    icon: mdi:home-automation
    tap_action:
      action: perform-action
      perform_action: input_number.set_value
      data:
        entity_id: input_number.living_room_scene_state
        value: 0
```

**Scene Status Conditional Card**:
```yaml
type: conditional
conditions:
  - entity: input_number.living_room_scene_state
    state_not: "0"
card:
  type: markdown
  content: |
    {% set scene_state = states('input_number.living_room_scene_state') | int %}
    {% if scene_state == 1 %}
      🎬 **Movie Mode Active**
      Lights in cinema mode. Press scene button to change.
    {% elif scene_state == 2 %}
      📖 **Reading Mode Active**
      Bright focused lighting. Press scene button to change.
    {% elif scene_state == 3 %}
      🎉 **Party Mode Active**
      Dynamic colorful lighting. Press scene button to change.
    {% endif %}
  title: Active Scene Override
```

### Troubleshooting Scene Cycling

#### Issue: Button press doesn't cycle scenes

**Diagnosis**:
1. Check button entity triggers:
   ```
   Settings > Automations > Your automation > Traces
   Look for "scene_cycle_pressed" trigger
   ```
2. Check scene cycling enabled:
   ```
   Edit automation > "Enable Scene Cycling System" = ON
   ```
3. Check state tracker entity:
   ```
   Developer Tools > States > input_number.living_room_scene_state
   Should exist and show value 0-3
   ```

**Solutions**:
- Ensure button entity is correct in blueprint config
- Verify button actually changes state (Developer Tools > States)
- Check automation mode = "restart" (allows re-triggering)
- Review automation traces for errors

#### Issue: Scenes don't activate (lights don't change)

**Diagnosis**:
1. Check scene entity exists:
   ```
   Developer Tools > States > scene.living_room_movie
   Should be listed
   ```
2. Test scene manually:
   ```
   Developer Tools > Actions
   Action: scene.turn_on
   Entity: scene.living_room_movie
   Call Service
   ```
3. Check scene configuration:
   ```
   Edit automation > Verify scene entities are filled in
   Not empty/default {}
   ```

**Solutions**:
- Recreate scenes in Philips Hue app or HA Scenes
- Wait 2-3 minutes for Hue scenes to sync
- Verify scene entities are correctly spelled
- Check scene entities match domain: `scene.xxx`

#### Issue: State tracker doesn't update

**Diagnosis**:
1. Check input_number permissions:
   ```
   Developer Tools > States > input_number.living_room_scene_state
   Should allow editing (not read-only)
   ```
2. Check for automation errors:
   ```
   Settings > System > Logs
   Search for: input_number.set_value errors
   ```

**Solutions**:
- Recreate input_number helper (delete and create new)
- Verify Min=0, Max=3, Step=1
- Check no other automations are conflicting
- Restart Home Assistant if helper just created

#### Issue: Scene stays active after cycling to "Off"

**Diagnosis**:
1. Check state tracker value:
   ```
   Developer Tools > States > input_number.living_room_scene_state
   Should be 0 when "off"
   ```
2. Check presence status:
   ```
   If presence = OFF and lux is high, lights may turn off
   normally (expected behavior)
   ```

**Solutions**:
- Manually set state tracker to 0 (Developer Tools > States > Set value)
- Verify presence detected before expecting lights to resume normal
- Check lux level is below ON threshold (< 80 lux)
- Press button again to manually cycle back to off state

#### Issue: Bypass timeout not working

**Diagnosis**:
1. Check bypass setting:
   ```
   Edit automation > "Scene Bypass Presence Timeout"
   Should be ON if you want lights to stay on indefinitely
   ```
2. Check presence status:
   ```
   Developer Tools > States > binary_sensor.xxx_presence
   Should show "off" when testing bypass
   ```

**Solutions**:
- Verify bypass enabled in blueprint
- Leave room to test (presence must actually clear)
- Check automation traces for "Scene bypass" log messages
- If bypass OFF, lights WILL turn off after 45 min (normal)

#### Issue: Circadian updates still happening during scene

**Diagnosis**:
1. Check scene state:
   ```
   input_number.living_room_scene_state should be > 0
   ```
2. Check automation traces:
   ```
   Look for "Skipping circadian update - scene X is active"
   ```

**Solutions**:
- Verify scene state tracker is actually set to 1, 2, or 3
- Check blueprint variables: `scene_is_active` should be true
- Review automation logic (ensure scene checks are in place)
- File bug report if circadian updates still occurring

### Scene Cycling Best Practices

1. **Scene Design**:
   - Make scenes distinctly different (easy to tell which is active)
   - Test scenes in actual room conditions
   - Consider transition times (2 sec default is good for most)

2. **Button Placement**:
   - Mount physical buttons near seating area
   - Test button reach from common sitting positions
   - Consider wireless range for battery buttons

3. **State Tracker Management**:
   - Add to dashboard for visibility
   - Include in Home Assistant groups if using multiple rooms
   - Back up helper configuration (Settings > System > Backups)

4. **Bypass Strategy**:
   - Use bypass OFF for safety (default)
   - Use bypass ON only for specific use cases (movies, parties)
   - Consider time-based automations to auto-disable bypass

5. **Testing**:
   - Test full cycle: 0→1→2→3→0
   - Test with presence clearing (verify bypass behavior)
   - Test scene persistence after HA restart
   - Test lux override (scenes should ignore lux)

6. **Backward Compatibility**:
   - Scene cycling is fully optional
   - Existing override system still works independently
   - Can use both systems together (override + scene cycling)
   - Disable scene cycling anytime (no harm to automation)

---

## Troubleshooting

### Common Issues and Solutions

#### Issue 1: Lights Flicker On/Off During Cloudy Days

**Symptoms**:
- Lights turn on/off repeatedly
- Happens when clouds pass over
- Mostly occurs during midday

**Diagnosis**:
1. Check lux readings during flicker:
   - Developer Tools → States → Lux sensor
   - Is lux oscillating between ON and OFF thresholds?

**Solutions** (try in order):

1. **Increase hysteresis gap**:
   ```
   Current: 80 lux ON / 150 lux OFF (70 lux gap)
   Try:     60 lux ON / 150 lux OFF (90 lux gap)
   Or:      80 lux ON / 180 lux OFF (100 lux gap)
   ```

2. **Increase debounce timers**:
   ```
   Turn-OFF Debounce: 5 min → 10 min (600 sec)
   Turn-ON Debounce:  2 min → 5 min (300 sec)
   ```

3. **Use sunrise/sunset extended debounce all day** (temporary):
   ```
   Sunrise/Sunset Extended Debounce: 10 min → 15 min
   (This applies during dawn/dusk only)
   ```

4. **Relocate lux sensor**:
   - Move away from direct window view
   - Position to measure average room brightness
   - Avoid spots with dramatic shadow changes

#### Issue 2: Lights Don't Turn On When Entering Dark Room

**Symptoms**:
- Walk into dark room
- Presence detected but lights stay off
- Lux seems appropriate

**Diagnosis**:
1. Check presence sensor:
   ```
   Developer Tools → States → binary_sensor.xxx_presence
   Should show "on" when in room
   ```

2. Check lux reading:
   ```
   Current lux: ____ lux
   Turn-ON threshold: ____ lux
   Is current lux < threshold?
   ```

3. Check override state:
   ```
   If override is ON and you expect lux-based control,
   override might be blocking normal operation
   ```

**Solutions**:

1. **Increase Turn-ON threshold**:
   ```
   Current: 80 lux
   Try:     120 lux (if room actually dark at 80-100 lux)
   ```

2. **Decrease Turn-ON debounce**:
   ```
   Current: 2 minutes (120 sec)
   Try:     30 seconds (testing) or 1 minute (normal use)
   ```

3. **Verify presence sensor sensitivity**:
   - FP2: Increase sensitivity in Aqara Home app
   - PIR: Check sensor angle and placement
   - Test by waving arm vigorously

4. **Check automation mode**:
   - Ensure automation is enabled
   - Check for conflicting automations controlling same lights

#### Issue 3: Lights Turn Off Too Quickly When Opening Curtains

**Symptoms**:
- Open curtains on sunny day
- Lights turn off within seconds/minutes
- Debounce doesn't seem to work

**Diagnosis**:
1. Check current lux after opening curtains:
   ```
   Lux reading: ____ lux (probably > 300 lux)
   Turn-OFF threshold: 150 lux
   ```

2. Verify debounce setting:
   ```
   Turn-OFF Debounce: Should be 5-10 minutes
   ```

**Solutions**:

1. **Increase Turn-OFF threshold**:
   ```
   Current: 150 lux
   Try:     250 lux (tolerates more natural light before turning off)
   Or:      300 lux (keeps lights on even with curtains open)
   ```

2. **Increase Turn-OFF debounce**:
   ```
   Current: 5 minutes
   Try:     10 minutes (600 sec)
   Or:      15 minutes (900 sec) for maximum stability
   ```

3. **Use override for sunny days**:
   - Activate override when curtains open
   - Manually control lighting on bright days

#### Issue 4: Staged Turn-Off Not Working

**Symptoms**:
- Leave room (presence clears)
- Lights turn off immediately or after long delay
- No dimming stages observed

**Diagnosis**:
1. Check staged turn-off setting:
   ```
   Settings → Edit automation
   "Enable Staged Turn-Off": Should be ON
   ```

2. Check stage timing:
   ```
   Stage 1 Delay: 30 minutes (1800 sec)
   Stage 2 Delay: 40 minutes (2400 sec)
   Stage 3 Delay: 45 minutes (2700 sec)
   ```

3. Check if presence re-detected:
   ```
   Automation restarts if presence detected before stage 3
   Check presence sensor for false triggers
   ```

**Solutions**:

1. **Verify staged turn-off enabled**:
   - Edit automation
   - Ensure toggle is ON
   - Save

2. **Reduce stage delays for testing**:
   ```
   Stage 1: 30 min → 2 min (120 sec) for quick testing
   Stage 2: 40 min → 4 min (240 sec)
   Stage 3: 45 min → 6 min (360 sec)
   Test, then restore original values
   ```

3. **Check for presence sensor false positives**:
   - Leave room completely
   - Monitor presence sensor state
   - Should stay "off" consistently
   - Reduce FP2 sensitivity if needed

4. **Review automation mode**:
   - Mode: restart (correct)
   - If presence detected during stages, automation restarts
   - This is normal behavior

#### Issue 5: Circadian Colors Don't Change Throughout Day

**Symptoms**:
- Lights always same color temperature
- No variation dawn to dusk
- Circadian feature seems inactive

**Diagnosis**:
1. Check circadian setting:
   ```
   Settings → Edit automation
   "Enable Circadian Rhythm": Should be ON
   ```

2. Check sun elevation:
   ```
   Developer Tools → States → sun.sun
   Attribute: elevation
   Should change throughout day (-18° to +60°)
   ```

3. Check lights support color_temp:
   ```
   Developer Tools → States → Your light entity
   Attributes should include: color_temp, min_mireds, max_mireds
   ```

**Solutions**:

1. **Verify circadian enabled**:
   - Edit automation
   - Ensure toggle is ON
   - Save

2. **Verify light compatibility**:
   - Check light specifications
   - Must support tunable white / color temperature
   - Test manually: Set color_temp via Developer Tools

3. **Increase update frequency for testing**:
   ```
   Current: 60 seconds
   Try:     30 seconds (more frequent updates)
   Observe for 5-10 minutes
   ```

4. **Decrease transition duration for visibility**:
   ```
   Current: 4 seconds
   Try:     1 second (more obvious changes)
   Watch for color shifts
   Restore to 4 seconds after testing
   ```

5. **Check mireds range**:
   ```
   Most Hue bulbs: 153-500 mireds (2000K-6500K)
   Blueprint range: 182-556 mireds (1800K-5500K)
   Should be compatible
   ```

#### Issue 6: Approach Detection Not Working (FP2)

**Symptoms**:
- Walk toward room
- Lights don't turn on instantly
- Approach sensor doesn't trigger

**Diagnosis**:
1. Check approach sensor entity:
   ```
   Developer Tools → States → binary_sensor.xxx_approach
   Should toggle "on" briefly when approaching
   ```

2. Verify approach zone in Aqara app:
   - Open Aqara Home → FP2 → Approach Detection
   - Should show configured zone at entrance

**Solutions**:

1. **Reconfigure approach detection**:
   - Aqara Home app → FP2 → Approach Detection
   - Redraw approach zone at entrance
   - Increase detection distance (0.5m → 2m)
   - Increase angle coverage (60° → 90°)

2. **Test approach sensor independently**:
   - Monitor in Developer Tools
   - Walk toward room multiple times
   - Verify sensor triggers consistently

3. **Verify approach sensor linked in automation**:
   - Edit automation
   - "Approach Detection Sensor": Should be set
   - Try clearing and re-selecting entity
   - Save

4. **Check FP2 firmware**:
   - Outdated firmware may have bugs
   - Update via Aqara Home app
   - Reboot FP2 after update

#### Issue 7: Override Not Working

**Symptoms**:
- Toggle override ON
- Lights still turn off when lux high
- Override seems ignored

**Diagnosis**:
1. Check override system enabled:
   ```
   Settings → Edit automation
   "Enable Manual Override System": Should be ON
   ```

2. Check override trigger entity state:
   ```
   Developer Tools → States → input_boolean.xxx_override
   Should show "on" when override active
   ```

3. Check automation receives trigger:
   ```
   Settings → Automations → Your automation → Traces
   Look for override_toggled trigger events
   ```

**Solutions**:

1. **Verify override system enabled**:
   - Edit automation
   - "Enable Manual Override System": Toggle ON
   - "Override Trigger Entity": Select your entity
   - Save

2. **Test override entity manually**:
   - Developer Tools → Services
   - Service: `input_boolean.toggle`
   - Target: Your override entity
   - Call Service
   - Watch state change

3. **Check entity type compatibility**:
   - Supported: input_boolean, switch, binary_sensor
   - Verify your entity is one of these types
   - Try creating new input_boolean helper

4. **Review automation logic**:
   - Traces show override_active variable value
   - Should be `true` when override ON
   - If always `false`, report as blueprint bug

### Advanced Troubleshooting

#### Enable Debug Logging

Add custom logging to track blueprint behavior:

1. Edit `configuration.yaml`:
   ```yaml
   logger:
     default: info
     logs:
       homeassistant.components.automation: debug
   ```

2. Restart Home Assistant

3. Monitor logs:
   - Settings → System → Logs
   - Filter for your automation name
   - Look for trigger events, condition failures, action executions

#### Use Automation Traces

View detailed execution history:

1. Settings → Automations & Scenes
2. Find your automation → Click ⋮ → **Traces**
3. Select recent trace to inspect:
   - Trigger that started automation
   - Conditions evaluated (pass/fail)
   - Actions executed
   - Variable values throughout execution

**What to look for**:
- Trigger: Verify correct trigger fired
- Variables: Check lux, presence, override values
- Conditions: See which conditions passed/failed
- Timing: Identify delays or stuck steps

#### Test Individual Components

Isolate issues by testing components separately:

**Test 1: Lights Only**
```yaml
# Temporary test automation
service: light.turn_on
target:
  entity_id: light.living_room_ceiling
data:
  brightness_pct: 70
  color_temp: 300  # ~3333K
  transition: 2
```

**Test 2: Lux Sensor Only**
- Monitor in Developer Tools
- Cover sensor → should read 0-10 lux
- Uncover sensor → should increase
- Verify realistic readings

**Test 3: Presence Sensor Only**
- Monitor in Developer Tools
- Enter room → should show "on"
- Leave room → should show "off" after timeout
- Verify no false triggers

**Test 4: Override System Only**
- Toggle input_boolean manually
- Verify state changes immediately
- Check automation traces for override_toggled trigger

---

## Example Configurations

### Configuration 1: Standard Living Room (Recommended Default)

**Environment**:
- Medium-sized living room (20-30 m²)
- 2-3 windows (moderate natural light)
- Mixture of sunny and cloudy days
- Used for TV watching, reading, family time

**Hardware**:
- Philips Hue White Ambiance bulbs (x4)
- Aqara FP2 mmWave sensor (ceiling-mounted)
- FP2 built-in lux sensor

**Configuration**:
```yaml
Lux Turn-ON Threshold: 80 lux
Lux Turn-OFF Threshold: 150 lux
Turn-ON Debounce: 2 minutes
Turn-OFF Debounce: 5 minutes
Sunrise/Sunset Debounce: 10 minutes

Dynamic Brightness: ENABLED
  ≤50 lux: 100%
  75 lux: 85%
  100 lux: 70%
  125 lux: 55%
  150 lux: 40%

Circadian Rhythm: ENABLED
  Update Interval: 60 seconds
  Transition Duration: 4 seconds

Staged Turn-Off: ENABLED
  Stage 1: 30 minutes → 40% brightness, 1800K
  Stage 2: 40 minutes → 20% brightness, 1800K
  Stage 3: 45 minutes → Off

Manual Override: ENABLED
  Trigger: input_boolean.living_room_override (dashboard toggle)

Approach Detection: ENABLED
  Sensor: binary_sensor.fp2_living_room_approach
```

**Why this works**:
- Balanced hysteresis gap (70 lux) prevents flicker
- Moderate debounce timers (stable but responsive)
- Dynamic brightness complements natural light
- Circadian rhythm supports healthy sleep cycles
- Staged turn-off prevents jarring darkness
- Override for movie nights

---

### Configuration 2: Bright South-Facing Room

**Environment**:
- Large living room with floor-to-ceiling windows
- South-facing (receives direct sun most of day)
- Often exceeds 500+ lux midday
- Needs lights for mornings and evenings only

**Hardware**:
- IKEA Tradfri tunable white bulbs (x6)
- Screek mmWave sensor
- Separate Xiaomi lux sensor (window-mounted)

**Configuration**:
```yaml
Lux Turn-ON Threshold: 60 lux  # Lower - room gets very bright
Lux Turn-OFF Threshold: 200 lux  # Higher - tolerates more light
Turn-ON Debounce: 3 minutes  # Slightly longer
Turn-OFF Debounce: 7 minutes  # Longer for stability
Sunrise/Sunset Debounce: 15 minutes  # Extended dawn/dusk

Dynamic Brightness: ENABLED
  ≤50 lux: 100%
  75 lux: 75%   # Dimmer - room naturally bright
  100 lux: 60%
  125 lux: 45%
  150 lux: 30%

Circadian Rhythm: ENABLED
  Update Interval: 60 seconds
  Transition Duration: 5 seconds  # Slower for gentle changes

Staged Turn-Off: ENABLED
  Stage 1: 20 minutes → 40%  # Faster turn-off
  Stage 2: 25 minutes → 20%
  Stage 3: 30 minutes → Off

Manual Override: ENABLED
  Trigger: Aqara wireless button (single press toggle)

Approach Detection: DISABLED (not using FP2)
```

**Why this works**:
- Wider hysteresis gap (140 lux) handles variable sun
- Longer debounce accommodates rapid cloud changes
- Aggressive brightness dimming (room is naturally bright)
- Faster staged turn-off (room used less frequently)
- Physical button override (convenient for movie mode)

---

### Configuration 3: Dark North-Facing Room

**Environment**:
- Smaller living room (15 m²)
- North-facing (limited direct sunlight)
- Rarely exceeds 200 lux even on sunny days
- Needs lights most of the day

**Hardware**:
- Philips Hue White & Color Ambiance bulbs (x2)
- Aqara FP2 (ceiling-mounted)
- FP2 built-in lux sensor

**Configuration**:
```yaml
Lux Turn-ON Threshold: 100 lux  # Higher - room stays darker
Lux Turn-OFF Threshold: 150 lux  # Narrower gap (stable light)
Turn-ON Debounce: 1 minute  # Faster response
Turn-OFF Debounce: 5 minutes  # Standard stability
Sunrise/Sunset Debounce: 8 minutes  # Shorter (less sun variation)

Dynamic Brightness: ENABLED
  ≤50 lux: 100%
  75 lux: 95%   # Keep brighter - room is dark
  100 lux: 85%
  125 lux: 70%
  150 lux: 55%  # Higher minimum brightness

Circadian Rhythm: ENABLED
  Update Interval: 90 seconds  # Less frequent (minimal sun change)
  Transition Duration: 4 seconds

Staged Turn-Off: ENABLED
  Stage 1: 45 minutes → 50%  # Longer delays, higher brightness
  Stage 2: 55 minutes → 30%
  Stage 3: 60 minutes → Off

Manual Override: ENABLED
  Trigger: input_boolean.living_room_override

Approach Detection: ENABLED
  Sensor: binary_sensor.fp2_living_room_approach
```

**Why this works**:
- Narrower hysteresis gap (50 lux) - room lux is stable
- Faster turn-on response (room clearly dark most of time)
- Higher brightness curve (compensates for lack of natural light)
- Longer staged turn-off (room is primary living space)
- Approach detection ensures instant activation

---

### Configuration 4: Energy-Saving Mode

**Environment**:
- Any living room
- Priority: Minimize energy consumption
- Accept slightly dimmer lighting
- Maximize use of natural light

**Hardware**:
- Any tunable white bulbs
- Any presence + lux sensor combination

**Configuration**:
```yaml
Lux Turn-ON Threshold: 60 lux  # Lower - prefer natural light
Lux Turn-OFF Threshold: 120 lux  # Lower - turn off sooner
Turn-ON Debounce: 3 minutes  # Longer - reduce activations
Turn-OFF Debounce: 3 minutes  # Shorter - turn off quicker
Sunrise/Sunset Debounce: 10 minutes

Dynamic Brightness: ENABLED
  ≤50 lux: 80%   # Reduced max brightness
  75 lux: 65%
  100 lux: 50%
  125 lux: 35%
  150 lux: 25%   # Very dim at threshold

Circadian Rhythm: ENABLED
  Update Interval: 120 seconds  # Less frequent updates
  Transition Duration: 3 seconds

Staged Turn-Off: ENABLED
  Stage 1: 10 minutes → 30%  # Fast turn-off
  Stage 2: 13 minutes → 15%
  Stage 3: 15 minutes → Off

Manual Override: ENABLED
  Trigger: input_boolean.living_room_override
  (Use sparingly - defeats energy savings)

Approach Detection: DISABLED (saves resources)
```

**Why this works**:
- Lower lux thresholds favor natural light
- Reduced brightness levels across all lux ranges
- Faster turn-off sequence minimizes runtime
- Less frequent circadian updates (lower overhead)
- Narrower hysteresis acceptable (energy priority)

**Energy Savings Estimate**:
- 40-60% reduction in lighting energy vs. always-on
- 20-30% reduction vs. standard presence automation
- Depends on room usage and natural light availability

---

## Future Enhancements

These features are planned for future releases:

### Night Lights Mode
**Concept**: Separate lighting behavior for nighttime hours (e.g., 11 PM - 6 AM)
- Very dim brightness (5-15%)
- Extra warm color temperature (1800K)
- Faster turn-off sequence (5-10 minutes)
- Ideal for bathroom trips, late-night kitchen visits

**Status**: Coming in v1.2

### Scene Cycling (IMPLEMENTED in v1.1)
**Concept**: Cycle through multiple Philips Hue scenes with a single button
- Movie scene, Reading scene, Party scene
- Scenes override lux-based control and circadian lighting
- Configurable presence timeout bypass

**Status**: ✅ IMPLEMENTED in v1.1 - See [Scene Cycling Override](#scene-cycling-override-new-v11)

### Adaptive Lighting Detection
**Concept**: Auto-detect when Home Assistant Adaptive Lighting component is active
- Seamless integration with Adaptive Lighting
- Coordinate color temperature changes
- Prevent conflicts between systems

**Status**: Not implemented in v1.0 - coming in v1.2

### Auto-Off Safety Timer
**Concept**: Absolute maximum on-time regardless of presence
- Safety feature: Turn off lights after X hours even if presence detected
- Prevents lights staying on indefinitely if sensor fails
- User-configurable timeout (e.g., 8-12 hours)

**Status**: Not implemented in v1.0 - coming in v1.1

### FP2 Target Count Integration
**Concept**: Adjust brightness based on number of people in room
- 1 person: Standard brightness
- 2+ people: Increase brightness by 10-20%
- 0 people: Turn off (faster than staged sequence)

**Status**: Not implemented in v1.0 - requires multi-zone support (v2.0)

### Multi-Zone Support
**Concept**: Different lighting behavior based on FP2 zone occupancy
- Couch zone active: Dim for TV watching
- Desk zone active: Bright for working
- Multiple zones: Balanced lighting

**Status**: Not implemented in v1.0 - planned for v2.0 (major rewrite)

**Why Not in v1.0?**
v1.0 focuses on rock-solid core functionality: hysteresis, dynamic brightness, circadian rhythm, staged turn-off. These features work flawlessly without complexity. Future versions will layer on advanced features for power users.

---

## Conclusion

This blueprint provides professional-grade intelligent lighting with:
- ✅ Anti-flicker hysteresis system (no more light oscillation!)
- ✅ Dynamic brightness that adapts to natural light
- ✅ Circadian rhythm for healthy sleep cycles
- ✅ Staged turn-off warnings (considerate automation)
- ✅ Manual override for movies/parties/preference
- ✅ Sunrise/sunset intelligence (extended debounce)
- ✅ FP2 approach detection (instant activation)

**Next Steps**:
1. Start with recommended defaults
2. Test for 3-7 days
3. Tune based on your room's unique characteristics
4. Fine-tune lux thresholds and debounce timers
5. Enjoy effortless, flicker-free intelligent lighting!

**Questions?**
- Review troubleshooting section
- Check blueprint comments (extensively documented)
- Open GitHub issue for bugs
- Share your configuration for others!

---

**Blueprint Version**: 1.0.0
**Last Updated**: 2025-11-17
**Compatibility**: Home Assistant 2024.1+
