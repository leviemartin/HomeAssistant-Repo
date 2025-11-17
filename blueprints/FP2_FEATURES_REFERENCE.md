# Aqara FP2 mmWave Sensor - Features Reference (Blueprint v1.0)

Complete guide to Aqara FP2 capabilities, configuration, and integration with the Intelligent Living Room Lighting blueprint v1.0.

**Blueprint Version**: 1.0.0
**Last Updated**: 2025-11-17
**FP2 Firmware Tested**: v1.0.7+

---

## FP2 Features - Blueprint v1.0 Status

| Feature | Status | Implementation | Notes |
|---------|--------|----------------|-------|
| **Presence Detection** | ENABLED | Single presence sensor | Whole-room detection (simple & reliable) |
| **Approach Detection** | ENABLED (Optional) | Separate approach sensor | Instant activation when approaching room |
| **Lux Sensor** | ENABLED (Required) | Built-in illuminance sensor | Core component for hysteresis logic |
| **Target Count** | DISABLED | Not implemented in v1.0 | Reserved for future enhancement |
| **Fall Detection** | DISABLED | Not implemented in v1.0 | Reserved for future enhancement |
| **Zone Mapping** | DISABLED | Single-zone only in v1.0 | Whole-room approach, multi-zone future |

**v1.0 Design Philosophy**: Simplicity and reliability first. Advanced zone-specific features reserved for future versions.

---

## Table of Contents

1. [FP2 Overview](#fp2-overview)
2. [Hardware Specifications](#hardware-specifications)
3. [Home Assistant Integration](#home-assistant-integration)
4. [Presence Detection Configuration](#presence-detection-configuration)
5. [Approach Detection Setup](#approach-detection-setup)
6. [Lux Sensor Configuration](#lux-sensor-configuration)
7. [Zone Management](#zone-management)
8. [Troubleshooting FP2 Issues](#troubleshooting-fp2-issues)
9. [FP2 vs Other Presence Sensors](#fp2-vs-other-presence-sensors)

---

## FP2 Overview

### What is the Aqara FP2?

The **Aqara FP2** (Presence Sensor FP2) is a millimeter-wave (mmWave) radar-based presence sensor that:

- **Detects stationary presence** (sitting, sleeping, working - not just movement)
- **Provides multi-zone tracking** (up to 30 configurable zones)
- **Includes built-in lux sensor** (measures ambient light)
- **Supports approach detection** (triggers when walking toward sensor)
- **Offers fall detection** (elderly care, safety monitoring)
- **Works through fabric/thin materials** (detects even behind curtains)

**Perfect for**: Living rooms, bedrooms, offices where people remain stationary for extended periods.

### Key Advantages Over PIR Motion Sensors

| Feature | FP2 mmWave | Traditional PIR |
|---------|------------|-----------------|
| **Stationary Detection** | ✅ Yes (core feature) | ❌ No (requires movement) |
| **False Negatives** | Very rare | Common (sitting still = "no presence") |
| **Detection Accuracy** | High (radar-based) | Moderate (infrared heat) |
| **Through Materials** | Yes (fabric, thin walls) | No (line-of-sight only) |
| **Zone Mapping** | Up to 30 zones | Single zone only |
| **Approach Detection** | ✅ Built-in | ❌ Not available |
| **Lux Sensor** | ✅ Built-in | Sometimes (separate sensor) |
| **Price** | Higher ($60-80) | Lower ($15-30) |

### Model Numbers

- **FP2**: Main model (CN/EU/US versions - same hardware)
- **Full Name**: Aqara Presence Sensor FP2
- **Manufacturer Code**: PS-S02D (Zigbee version - DISCONTINUED)
- **Current Version**: Wi-Fi based (connects via Aqara Home app)

---

## Hardware Specifications

### Technical Specs

| Specification | Details |
|--------------|---------|
| **Detection Technology** | 60 GHz millimeter-wave radar |
| **Detection Range** | 0-6 meters (configurable) |
| **Detection Angle** | 120° horizontal |
| **Detection Height** | 2.8 meters recommended (ceiling mount) |
| **Multi-Zone Support** | Up to 30 zones |
| **Lux Range** | 0-10,000 lux |
| **Power Supply** | USB-C (5V/1A) - wired only |
| **Connectivity** | Wi-Fi 2.4 GHz (Aqara Home app required) |
| **Integration** | Home Assistant via Aqara Home integration |
| **Dimensions** | 39 × 39 × 27.2 mm |
| **Weight** | ~35g (without cable) |

### Mounting Recommendations

**Optimal Placement**:
- **Ceiling-mounted** (preferred): Best coverage, fewer blind spots
- **Wall-mounted**: Possible but reduces detection range
- **Height**: 2.5-3 meters for living room use
- **Angle**: Tilted 15-30° downward (adjust via Aqara app)

**Coverage Area** (ceiling-mounted at 2.8m):
```
┌─────────────────────┐
│                     │
│    6m range         │
│    120° angle       │
│                     │
│       [FP2]         │ ← Ceiling
│      /     \        │
│    /         \      │
│  /             \    │
│                     │
└─────────────────────┘
      Living Room
```

**Avoid Placement Near**:
- Large metal objects (interference)
- HVAC vents (air movement false positives)
- Direct sunlight on sensor (overheating)
- Glass/mirrors (radar reflections)

---

## Home Assistant Integration

### Method 1: Aqara Home Integration (Recommended)

**Requirements**:
- Aqara Home app (iOS/Android)
- FP2 added to Aqara Home account
- Home Assistant Aqara Home integration

**Setup Steps**:

1. **Add FP2 to Aqara Home App**:
   - Open Aqara Home app
   - Tap **+** → **Add Device**
   - Select **Presence Sensor FP2**
   - Follow pairing instructions (connect to Wi-Fi)
   - Place in your living room location

2. **Install Aqara Home Integration in HA**:
   - Home Assistant → **Settings** → **Devices & Services**
   - Click **+ Add Integration**
   - Search for **Aqara Home**
   - Sign in with Aqara account
   - Select region (China/Europe/USA/Russia)
   - Integration will discover your FP2

3. **Verify Entities Created**:
   ```
   Developer Tools → States → Search "fp2"

   Expected entities:
   ✓ binary_sensor.fp2_living_room_presence
   ✓ sensor.fp2_living_room_light_sensor
   ✓ binary_sensor.fp2_living_room_approach (if configured)
   ⚬ sensor.fp2_living_room_monitoring_mode
   ⚬ switch.fp2_living_room_identify
   ```

### Method 2: Zigbee2MQTT (Legacy - No Longer Supported)

**Note**: FP2 previously had a Zigbee version (PS-S02D) but it was discontinued. Current FP2 is Wi-Fi only and does NOT support Zigbee2MQTT. If you have an old Zigbee version, it can still work but has limited features.

**Not Recommended**: Stick with Aqara Home integration for full functionality.

### Entity Breakdown

#### `binary_sensor.fp2_living_room_presence`
**Description**: Main presence sensor (on/off)
```yaml
State: on/off
Attributes:
  - device_class: occupancy
  - friendly_name: "Living Room Presence"
```

**Usage**:
- Primary trigger for blueprint
- `on` = Person detected in room
- `off` = No presence detected
- Updates in real-time (subsecond latency)

#### `sensor.fp2_living_room_light_sensor`
**Description**: Built-in lux/illuminance sensor
```yaml
State: 0-10000 (lux)
Attributes:
  - device_class: illuminance
  - unit_of_measurement: lux
  - friendly_name: "Living Room Lux"
```

**Usage**:
- Blueprint lux sensor input
- Real-time ambient light measurement
- Calibration available in Aqara app

#### `binary_sensor.fp2_living_room_approach` (Optional)
**Description**: Approach detection sensor
```yaml
State: on/off
Attributes:
  - device_class: motion (or occupancy)
  - friendly_name: "Living Room Approach"
```

**Usage**:
- Blueprint approach detection input
- Triggers when walking toward configured approach zone
- Must be configured in Aqara Home app first

---

## Presence Detection Configuration

### Basic Presence Setup (Recommended for Blueprint)

**Aqara Home App Configuration**:

1. Open Aqara Home → Select FP2 device
2. Tap **Detection Settings**
3. Configure:
   ```
   Detection Mode: Whole Room (simple, reliable)
   Sensitivity: High (detects sitting/stationary)
   Detection Range: 4-6 meters (adjust for room size)
   Interference Rejection: Auto (recommended)
   ```

**Why "Whole Room" Mode?**
- Blueprint treats entire room as single zone
- Simpler setup (no zone mapping needed)
- Fewer false negatives
- Zone-specific features not required for basic use

### Advanced: Multi-Zone Presence

If you want zone-specific automations (not covered by this blueprint):

1. Aqara Home → FP2 → **Zone Settings**
2. Tap **+ Add Zone**
3. Draw zones on floor plan:
   - **Zone 1**: Couch area
   - **Zone 2**: Desk area
   - **Zone 3**: Entrance
4. Each zone gets separate entity:
   ```
   binary_sensor.fp2_living_room_zone_1_presence
   binary_sensor.fp2_living_room_zone_2_presence
   binary_sensor.fp2_living_room_zone_3_presence
   ```

**Use Case**: Trigger different scenes based on which part of room you're in.

### Sensitivity Tuning

**Aqara Home App** → FP2 → **Detection Settings** → **Sensitivity**

| Level | Behavior | Best For |
|-------|----------|----------|
| **Low** | Detects obvious movement only | High-traffic areas (false positive issues) |
| **Medium** | Balanced (walking, sitting with movement) | General use |
| **High** | Detects subtle movement (typing, reading) | Living rooms, offices (stationary work) |
| **Very High** | Maximum sensitivity (breathing, micro-movements) | Bedrooms, detailed tracking |

**Recommendation for Living Room**: **High**
- Detects TV watching (mostly stationary)
- Catches reading, phone use
- Minimal false positives with proper placement

**Testing Sensitivity**:
1. Set to **High**
2. Sit completely still on couch for 5 minutes
3. Check if presence sensor stays `on`
4. If turns `off` → Increase to Very High
5. If false triggers from outside room → Decrease to Medium

---

## Approach Detection Setup

### What is Approach Detection?

**Definition**: Triggers when a person walks TOWARD a configured zone (before entering).

**Use Case**: Turn lights on as you approach the room (not after entering).

**How It Works**:
- FP2 tracks movement direction
- If movement vector points toward approach zone → Trigger
- Separate from general presence detection

### Configuration in Aqara Home App

1. **Open Aqara Home** → Select FP2
2. Tap **Approach Detection**
3. **Enable Approach Detection**: Toggle ON
4. **Configure Approach Zone**:
   - Tap **Set Zone**
   - Draw a zone at room entrance:
     ```
     ┌────────────────────┐
     │   Living Room      │
     │                    │
     │                    │
     │   [Approach Zone]  │ ← Near doorway
     │        ↑           │
     └────────┼───────────┘
              │
            Hallway
     ```
   - **Zone Size**: 0.5m - 2m from doorway
   - **Angle**: 60-90° (covers entrance path)

5. **Sensitivity**: Medium (reduce false triggers from hallway)
6. **Save Settings**

### Verify in Home Assistant

1. **Developer Tools** → **States**
2. Search: `binary_sensor.fp2_living_room_approach`
3. **Test**:
   - Stand in hallway
   - Walk toward living room entrance
   - Entity should briefly show `on`
   - Then switch to `off` (momentary trigger)

### Add to Blueprint

1. Edit your automation
2. Scroll to **Approach Detection Sensor**
3. Select: `binary_sensor.fp2_living_room_approach`
4. Save

**Behavior**:
- Approach detected → Lights turn on immediately
- Bypasses lux check (instant activation)
- Seamlessly transitions to normal presence automation

### Troubleshooting Approach Detection

**Issue**: Approach sensor never triggers

**Solutions**:
1. **Check zone placement**:
   - Must be at entrance (not inside room)
   - Should cover walking path to room
2. **Increase sensitivity** in Aqara app
3. **Increase zone size** (1m → 2m)
4. **Verify entity exists** in HA (Developer Tools)

**Issue**: False triggers from hallway

**Solutions**:
1. **Decrease sensitivity** in Aqara app
2. **Reduce zone size** (2m → 1m)
3. **Narrow zone angle** (90° → 60°)
4. **Reposition FP2** (angle away from hallway)

---

## Lux Sensor Configuration

### FP2 Built-In Lux Sensor

**Specifications**:
- **Range**: 0-10,000 lux
- **Accuracy**: ±15% (typical)
- **Update Frequency**: 1-5 seconds
- **Placement**: Built into FP2 housing

**Usage in Blueprint**:
```yaml
Lux/Illuminance Sensor: sensor.fp2_living_room_light_sensor
```

### Calibration

**Aqara Home App** → FP2 → **Light Sensor** → **Calibrate**

**When to Calibrate**:
- After initial installation
- If lux readings seem inaccurate
- After relocating FP2

**Calibration Process**:
1. Place FP2 in final installation location
2. Open Aqara Home → FP2 → Light Sensor
3. Tap **Calibrate**
4. Follow on-screen instructions:
   - Expose sensor to known light level
   - App measures baseline
   - Confirms calibration success

### Lux Sensor Placement Considerations

**Optimal Placement**:
- **Near window** (represents natural light)
- **NOT in direct sunlight** (overreads)
- **Ceiling height** (if FP2 ceiling-mounted)
- **Unobstructed view** of room

**Example Placement**:
```
        Window
          │
          │  ☀️ Sunlight
          ↓
    ┌──────────────┐
    │  [FP2]       │ ← Ceiling, near window (but not directly under)
    │              │
    │              │    Measures overall room brightness
    │              │    (not direct sun, not shadowed corner)
    │              │
    │    Couch     │
    └──────────────┘
```

### Comparing FP2 Lux vs Dedicated Sensor

**FP2 Lux Sensor**:
- ✅ Convenient (built-in)
- ✅ Single device (less complexity)
- ⚠️ Placement limited to FP2 location
- ⚠️ Accuracy acceptable but not lab-grade

**Dedicated Lux Sensor** (e.g., Xiaomi, Hue Motion):
- ✅ Flexible placement (anywhere)
- ✅ Higher accuracy (±5-10%)
- ❌ Additional device (more cost)
- ❌ Extra maintenance (battery replacements)

**Recommendation**: Start with FP2 built-in sensor. Switch to dedicated sensor only if:
- FP2 placement not ideal for lux measurement
- Need higher accuracy
- FP2 readings inconsistent

---

## Zone Management

### Zone Configuration Overview

FP2 supports up to **30 configurable zones** for detailed presence tracking.

**Note**: This blueprint uses **simple whole-room presence** (not zone-specific). Zone features described here are for reference/future expansion.

### Creating Zones (Aqara Home App)

1. **Open Aqara Home** → Select FP2
2. Tap **Zone Settings**
3. Tap **+ Add Zone**
4. **Draw Zone on Floor Plan**:
   - Drag corners to define area
   - Name zone (e.g., "Couch", "Desk", "TV Area")
   - Set detection sensitivity per zone
5. **Save**

### Zone Entities in Home Assistant

Each zone creates a separate presence entity:

```yaml
binary_sensor.fp2_living_room_zone_1_presence  # Couch area
binary_sensor.fp2_living_room_zone_2_presence  # Desk area
binary_sensor.fp2_living_room_zone_3_presence  # TV area
```

**Advanced Use** (beyond this blueprint):
- Zone 1 active → Dim lights (watching TV)
- Zone 2 active → Bright lights (working at desk)
- Zone 3 active → Warm lights (reading nook)

### Zone Best Practices

**For Living Room**:
- **Keep it simple**: 1-3 zones max
- **Logical areas**: Couch, desk, entrance
- **Avoid tiny zones**: FP2 accuracy ~0.5m radius
- **Test coverage**: Walk around, verify detection

**Common Mistakes**:
- Too many zones (30 zones = overkill)
- Overlapping zones (confusing automation)
- Zones outside FP2 range (6m limit)

---

## Troubleshooting FP2 Issues

### Issue 1: FP2 Not Detected in Home Assistant

**Symptoms**:
- FP2 shows in Aqara Home app
- Doesn't appear in HA Aqara integration
- No entities created

**Solutions**:

1. **Verify Integration Installed**:
   - Settings → Devices & Services → Search "Aqara"
   - Should show "Aqara Home" integration
   - If not, add integration (see setup section)

2. **Check Region Match**:
   - Aqara Home app → Account → Region
   - HA Integration → Configure → Region
   - Must match (China/Europe/USA/Russia)

3. **Reload Integration**:
   - Devices & Services → Aqara Home → ⋮ → Reload

4. **Remove and Re-add FP2**:
   - Aqara Home app → FP2 → Settings → Remove Device
   - Re-add FP2 to Aqara Home
   - Wait 5 minutes → Check HA integration

5. **Check HA Logs**:
   - Settings → System → Logs
   - Search for "aqara" errors
   - Common: Authentication failed, region mismatch

---

### Issue 2: Presence Detection Not Working

**Symptoms**:
- FP2 installed, entities exist
- `binary_sensor.xxx_presence` always shows `off`
- No presence detected even when in room

**Solutions**:

1. **Check Detection Range**:
   - Aqara Home → FP2 → Detection Settings → Range
   - Increase to 5-6 meters
   - Save and test

2. **Increase Sensitivity**:
   - Detection Settings → Sensitivity → High or Very High
   - Save and test

3. **Verify Mounting**:
   - FP2 should be ceiling-mounted (or high wall)
   - Angle tilted downward 15-30°
   - Not blocked by furniture/objects

4. **Check Interference**:
   - Move away from metal objects
   - Avoid HVAC vents
   - Check for other mmWave devices (interference)

5. **Test in Aqara App**:
   - Open Aqara Home → FP2 → Real-Time Detection
   - Walk around room → Should show movement
   - If no detection in app → Hardware/placement issue

6. **Restart FP2**:
   - Unplug USB-C power
   - Wait 10 seconds
   - Plug back in
   - Wait for reconnection (2-3 minutes)

---

### Issue 3: Lux Sensor Readings Seem Wrong

**Symptoms**:
- Lux sensor shows unusual values
- Reads 0 lux in bright room
- Reads 5000 lux in dim room
- Erratic readings

**Solutions**:

1. **Recalibrate Lux Sensor**:
   - Aqara Home → FP2 → Light Sensor → Calibrate
   - Follow calibration procedure

2. **Check Sensor Obstruction**:
   - Ensure lux sensor opening not blocked
   - Remove any tape/stickers from FP2 housing
   - Verify clear line of sight

3. **Compare to Known Source**:
   - Use phone lux meter app (approx reference)
   - Compare readings in same location
   - FP2 should be within 20-30% of phone

4. **Verify Entity in HA**:
   ```
   Developer Tools → States → sensor.fp2_xxx_light_sensor
   State should be numeric (e.g., 142 lux)
   If "unavailable" or "unknown" → Integration issue
   ```

5. **Check HA Recorder**:
   - History Graph → Lux sensor
   - Look for sudden spikes/drops (sensor malfunction)
   - Smooth curve = healthy sensor

---

### Issue 4: Approach Detection Not Triggering

**See**: [Approach Detection Setup](#approach-detection-setup) troubleshooting section above.

---

### Issue 5: FP2 Goes Offline Frequently

**Symptoms**:
- FP2 shows "unavailable" in HA
- Presence/lux entities show "unknown"
- Frequent disconnections

**Solutions**:

1. **Check Wi-Fi Signal**:
   - FP2 location should have strong Wi-Fi (-60 dBm or better)
   - Use Wi-Fi analyzer app to check signal
   - Move router closer or add Wi-Fi extender

2. **Check Power Supply**:
   - Use quality USB-C cable (not cheap knockoff)
   - Verify 5V/1A power adapter (official recommended)
   - Try different USB power source

3. **Restart Router**:
   - Restart home Wi-Fi router
   - Wait for FP2 to reconnect (2-5 minutes)

4. **Update FP2 Firmware**:
   - Aqara Home → FP2 → Settings → Firmware Update
   - Install latest firmware
   - Restart FP2 after update

5. **Check Network Settings**:
   - FP2 only supports 2.4 GHz Wi-Fi (not 5 GHz)
   - Verify router has 2.4 GHz enabled
   - Disable AP isolation (if enabled on router)

---

## FP2 vs Other Presence Sensors

### Comparison Matrix

| Feature | FP2 mmWave | PIR Motion | Aqara FP1 | Other mmWave (Screek, HLK) |
|---------|------------|------------|-----------|----------------------------|
| **Stationary Detection** | ✅ Excellent | ❌ No | ✅ Good | ✅ Varies |
| **Multi-Zone** | ✅ 30 zones | ❌ Single | ❌ Single | ⚠️ Some models |
| **Approach Detection** | ✅ Yes | ❌ No | ❌ No | ❌ Usually no |
| **Lux Sensor** | ✅ Built-in | ⚠️ Some models | ❌ No | ❌ Usually no |
| **Range** | 6 meters | 3-8 meters | 7 meters | 5-12 meters |
| **Accuracy** | High | Moderate | Moderate | Varies |
| **Price** | $60-80 | $15-30 | $40-50 | $30-60 |
| **Power** | USB wired | Battery | USB wired | USB wired |
| **Integration** | Wi-Fi (Aqara) | Zigbee/Z-Wave | Zigbee | Zigbee/ESPHome |
| **Local Control** | ⚠️ Cloud-dependent | ✅ Fully local | ✅ Fully local | ✅ Usually local |

### When to Choose FP2

**Best For**:
- Living rooms (long stationary periods)
- Offices (sitting at desk)
- Bedrooms (sleeping detection)
- Multi-zone tracking needs
- All-in-one solution (presence + lux + approach)

**Not Ideal For**:
- Budget-constrained setups (use PIR)
- Fully local-only systems (FP2 needs cloud)
- Outdoor use (not weather-rated)
- Very large rooms (>6m range limit)

### When to Choose Alternatives

**Use PIR Motion if**:
- Budget priority ($15 vs $70)
- Movement-based automation sufficient
- Battery-powered needed
- Fully local control required

**Use Aqara FP1 if**:
- Need stationary detection
- Don't need multi-zone or approach
- Prefer fully local Zigbee
- Lower cost than FP2

**Use Other mmWave (Screek/HLK) if**:
- Need ESPHome/local control
- Longer range required (>6m)
- DIY/tinkering preferred
- Want customizable firmware

---

## Advanced FP2 Features

### Fall Detection (Not Used by Blueprint)

**What It Does**: Detects sudden falls (elderly care, safety)

**Setup**:
- Aqara Home → FP2 → Fall Detection → Enable
- Configure fall zone (area to monitor)
- Set up notifications (Aqara app or HA automation)

**Entities Created**:
```yaml
binary_sensor.fp2_living_room_fall_detected
```

**Use Case**: Separate automation for fall alerts (beyond scope of lighting blueprint).

### Edge Zones (Not Used by Blueprint)

**What It Does**: Detects when presence enters/exits specific zones

**Use Case**:
- Entrance zone → Turn on lights
- Exit zone → Start turn-off countdown

**Note**: Blueprint uses simple whole-room presence (edge zones optional for advanced users).

---

## Conclusion

### FP2 Setup Checklist for Blueprint

- [ ] FP2 added to Aqara Home app
- [ ] Aqara Home integration installed in HA
- [ ] Presence entity verified: `binary_sensor.xxx_presence`
- [ ] Lux entity verified: `sensor.xxx_light_sensor`
- [ ] Detection sensitivity set to **High**
- [ ] Detection range set to **4-6 meters**
- [ ] Detection mode set to **Whole Room**
- [ ] Lux sensor calibrated
- [ ] (Optional) Approach detection configured
- [ ] (Optional) Approach entity verified: `binary_sensor.xxx_approach`

### Key Takeaways

1. **FP2 excels at stationary presence detection** (core advantage over PIR)
2. **Built-in lux sensor** convenient for single-device solution
3. **Approach detection** enables instant light activation
4. **Whole room mode** sufficient for blueprint (multi-zone optional)
5. **Ceiling mounting** provides best coverage
6. **High sensitivity** recommended for living room use
7. **Wi-Fi connectivity** requires cloud (trade-off for features)

### Further Resources

- **Aqara Official Docs**: https://www.aqara.com/en/product/presence-sensor-fp2
- **Blueprint Setup**: See `INTELLIGENT_LIVING_ROOM_SETUP_GUIDE.md`
- **Troubleshooting**: See setup guide troubleshooting section
- **Community**: Home Assistant forums, r/homeassistant

---

**Document Version**: 1.0.0
**Last Updated**: 2025-11-17
**FP2 Firmware Tested**: v1.0.7+ (check Aqara app for latest)
