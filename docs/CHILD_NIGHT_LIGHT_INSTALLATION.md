# Child Night Light Blueprint - Installation Guide

Quick start guide for installing and configuring the Child-Friendly Night Light automation blueprint.

## Prerequisites

Before installing this blueprint, ensure you have:

### Required Hardware

1. **Smart Lights** (1-4 lights):
   - Must support color temperature adjustment
   - Compatible brands: Philips Hue, IKEA TRADFRI, Sengled, Lifx
   - Not compatible: RGB-only lights or fixed color temperature bulbs

2. **Home Assistant** (version 2024.11 or later):
   - Properly configured and running
   - Lights integrated and working

### Optional Hardware

3. **Motion Sensor** (recommended):
   - Binary sensor with device_class: motion
   - For automatic brightness boost during nighttime checks

## Installation Methods

### Method 1: Direct Import (Easiest)

1. Open Home Assistant web interface
2. Navigate to **Settings** → **Automations & Scenes**
3. Click the **Blueprints** tab
4. Click **Import Blueprint** button (bottom right)
5. Paste the blueprint URL:
   ```
   https://github.com/yourusername/ha-blueprints/blob/main/blueprints/child_night_light.yaml
   ```
6. Click **Preview Blueprint**
7. Review the blueprint details
8. Click **Import Blueprint**

### Method 2: Manual File Copy

1. Download `child_night_light.yaml` from the repository
2. Connect to your Home Assistant instance (via SSH, Samba, or file editor)
3. Navigate to your configuration directory (usually `/config/`)
4. Create the blueprints directory structure if it doesn't exist:
   ```bash
   mkdir -p /config/blueprints/automation
   ```
5. Copy `child_night_light.yaml` to:
   ```
   /config/blueprints/automation/child_night_light.yaml
   ```
6. Restart Home Assistant OR reload automations:
   - **Settings** → **System** → **Restart** (choose "Quick reload" → "Automations")

### Method 3: File Editor Add-on

If you have the File Editor add-on installed:

1. Open **File Editor** from Home Assistant sidebar
2. Navigate to `/config/blueprints/automation/`
3. Create new file: `child_night_light.yaml`
4. Copy and paste the blueprint YAML content
5. Save file
6. Reload automations (Settings → System → Quick Reload → Automations)

## Verification

Confirm successful installation:

1. Navigate to **Settings** → **Automations & Scenes**
2. Click **Blueprints** tab
3. Look for "Child-Friendly Night Light Automation (Ages 1-5)"
4. You should see the blueprint card with description

## Creating Your First Automation

### Basic Setup (5 Minutes)

1. **Start New Automation**:
   - Settings → Automations & Scenes → Blueprints tab
   - Find "Child-Friendly Night Light Automation"
   - Click **Create Automation**

2. **Name Your Automation**:
   - Example: "Nursery Night Light"
   - Example: "Bedroom Night Light - Toddler"

3. **Configure Required Settings**:

   **Night Lights**:
   - Click the selector
   - Choose entity, device, or area
   - Select 1-4 lights (e.g., "Bedroom Lamp")

   **Nighttime Schedule**:
   - Enable Nighttime Schedule: ✓ (checked)
   - Nighttime Start Time: `19:00:00` (7:00 PM or your child's bedtime)
   - Nighttime End Time: `07:00:00` (7:00 AM or your child's wake time)

4. **Use Default Settings** (recommended for first setup):
   - Base Brightness: 3%
   - Color Temperature: 2000K
   - All other defaults are research-backed and safe

5. **Save Automation**:
   - Click **Save** (bottom right)

6. **Test Immediately** (optional):
   - Click the ⋮ menu on your new automation
   - Select **Run**
   - Lights should turn on at 3% brightness with warm amber color
   - Verify the result looks appropriate

### Advanced Setup with Motion Sensor

Follow steps 1-3 above, then additionally configure:

4. **Motion Sensor Settings**:
   - Motion Sensor: Select your motion detector
   - Motion Brightness Boost: `10` (or 8-15%)
   - Motion Boost Duration: `60` (or 30-120 seconds)

5. **Save and Test**:
   - Save automation
   - Wait for active time period OR manually trigger
   - Trigger motion sensor
   - Verify brightness increases then returns to base level

### Setup with Nap Times

Follow basic setup, then additionally:

4. **Nap Time Slot 1** (if applicable):
   - Enable Nap Time Slot 1: ✓ (checked)
   - Nap 1 Start Time: `13:00:00` (1:00 PM or your child's nap time)
   - Nap 1 End Time: `15:00:00` (3:00 PM or when nap ends)

5. **Nap Time Slot 2** (if applicable):
   - Enable Nap Time Slot 2: ✓ (checked)
   - Nap 2 Start Time: `09:00:00` (9:00 AM for morning nap)
   - Nap 2 End Time: `11:00:00` (11:00 AM or when nap ends)

6. **Save automation**

## Configuration Templates

### Template 1: Young Infant (1-12 months)

```yaml
Target Lights: Nursery ceiling light
Base Brightness: 2%
Color Temperature: 2000K
Nighttime Enabled: Yes
Nighttime Start: 18:30:00 (6:30 PM)
Nighttime End: 06:30:00 (6:30 AM)
Nap 1 Enabled: Yes
Nap 1 Start: 09:00:00
Nap 1 End: 11:00:00
Nap 2 Enabled: Yes
Nap 2 Start: 13:00:00
Nap 2 End: 15:00:00
Motion Sensor: Nursery motion detector
Motion Boost: 10%
Motion Duration: 90 seconds
```

**Why these settings**: Very low brightness for highly sensitive infant eyes, two nap periods, motion boost for parent checks.

### Template 2: Toddler (1-3 years)

```yaml
Target Lights: Bedroom lamp, Hallway light
Base Brightness: 3%
Color Temperature: 2000K
Nighttime Enabled: Yes
Nighttime Start: 19:00:00 (7:00 PM)
Nighttime End: 07:00:00 (7:00 AM)
Nap 1 Enabled: Yes
Nap 1 Start: 13:00:00 (1:00 PM)
Nap 1 End: 15:00:00 (3:00 PM)
Nap 2 Enabled: No
Motion Sensor: Hallway motion detector
Motion Boost: 12%
Motion Duration: 60 seconds
```

**Why these settings**: Standard safe brightness, one afternoon nap, hallway motion for bathroom trips.

### Template 3: Preschooler (3-5 years)

```yaml
Target Lights: Bedroom night light
Base Brightness: 4%
Color Temperature: 2200K
Nighttime Enabled: Yes
Nighttime Start: 20:00:00 (8:00 PM)
Nighttime End: 07:00:00 (7:00 AM)
Nap 1 Enabled: No
Nap 2 Enabled: No
Motion Sensor: Bedroom motion detector
Motion Boost: 15%
Motion Duration: 120 seconds
```

**Why these settings**: Slightly higher brightness acceptable at this age, no naps, motion for bathroom independence.

### Template 4: Sleep-Sensitive Child (Any Age)

```yaml
Target Lights: Small corner lamp
Base Brightness: 1%
Color Temperature: 1700K
Nighttime Enabled: Yes
Nighttime Start: 19:00:00
Nighttime End: 07:00:00
Nap 1/2 Enabled: As needed
Motion Sensor: None
Transition Duration: 8 seconds
```

**Why these settings**: Absolute minimum light presence, warmest color, very gentle transitions, no motion triggers to avoid disturbance.

## Verifying Proper Operation

### Visual Verification Checklist

When automation is active, verify:

- [ ] Light is very dim (should barely illuminate room)
- [ ] Color is warm amber/orange (not white or blue)
- [ ] Light is comfortable for sleeping child
- [ ] Transitions are smooth and gentle
- [ ] Motion trigger works (if configured)
- [ ] Motion return-to-base works correctly

### Light Level Test

To ensure safe brightness:

1. Turn on automation manually during daytime
2. Enter child's room
3. Close eyes for 30 seconds
4. Open eyes - light should be:
   - Barely visible initially
   - Warm amber/orange color
   - Not bright enough to read by
   - Equivalent to dim firelight

If light is too bright, reduce Base Brightness setting.

### Color Temperature Test

To ensure proper amber color:

1. Manually set light to your configured temperature (default 2000K)
2. Compare to these references:
   - Should look similar to candlelight or sunset
   - Should be noticeably more orange/amber than regular "warm white"
   - Should NOT look neutral white or cool white
   - Compare to this scale: 🔴 Red → 🟠 Amber ← You want this | 🟡 Warm White | ⚪ White

### Motion Sensor Test (if configured)

1. Wait for active time period (or manually trigger automation)
2. Verify lights are at base brightness
3. Trigger motion sensor
4. Confirm lights increase to motion boost brightness
5. Stop moving and wait for boost duration
6. Confirm lights return to base brightness

**Expected behavior**:
- Base: Very dim amber (3%)
- Motion detected: Noticeably brighter amber (10-15%)
- After duration: Returns to very dim amber (3%)

## Troubleshooting Installation

### Blueprint doesn't appear after installation

**Solutions**:
1. Reload automations: Settings → System → Quick Reload → Automations
2. Full restart: Settings → System → Restart
3. Clear browser cache and refresh
4. Check file is in correct location: `/config/blueprints/automation/child_night_light.yaml`
5. Verify YAML syntax is correct (no copy/paste errors)

### "Invalid blueprint" error

**Causes**:
- YAML syntax error
- Incorrect indentation
- Missing required fields

**Solutions**:
1. Re-download original blueprint file
2. Use YAML validator tool
3. Check Home Assistant logs for specific error
4. Ensure you copied the entire file content

### Automation doesn't trigger at scheduled time

**Check**:
1. Automation is enabled (toggle should be blue/on)
2. Lights have power (smart bulbs need physical power)
3. Time zone is correct in Home Assistant
4. Schedule times are correct (AM vs PM)
5. "Enable Nighttime Schedule" is checked
6. Check automation traces for errors

### Lights turn on but wrong color/brightness

**Check**:
1. Light actually supports color temperature (test manually)
2. Light's color temperature range (may not support 2000K)
3. Service call data in automation trace
4. Try different color temperature value
5. Update light firmware if available

## Next Steps

After successful installation:

1. **Monitor for 3-5 days**:
   - Observe child's response
   - Check sleep quality
   - Adjust brightness if needed

2. **Fine-tune settings**:
   - Reduce brightness if possible
   - Adjust motion sensitivity
   - Modify time ranges as needed

3. **Consider expansion**:
   - Add hallway lights for bathroom path
   - Create separate automations for different rooms
   - Integrate with other sleep routines

4. **Read full documentation**:
   - See CHILD_NIGHT_LIGHT_README.md for detailed guidance
   - Review age-specific recommendations
   - Learn about scientific research behind settings

## Getting Help

If you encounter issues:

1. Check the Troubleshooting section in the main README
2. Review Home Assistant automation traces
3. Verify light compatibility
4. Test lights manually before automating
5. Check Home Assistant community forums
6. Review logs for error messages

## Updating the Blueprint

When a new version is released:

### Direct Import Method:
1. Go to Settings → Automations & Scenes → Blueprints
2. Find the blueprint
3. Click ⋮ menu → Re-import
4. Confirm update

### Manual Method:
1. Download new version
2. Replace old file
3. Reload automations
4. Existing automations will use new version

**Note**: Updating blueprint doesn't change existing automations' settings. Your configurations are preserved.

## Uninstalling

To remove the blueprint:

1. Delete all automations created from this blueprint
2. Go to Settings → Automations & Scenes → Blueprints
3. Find the blueprint
4. Click ⋮ menu → Delete
5. Confirm deletion

OR manually delete the file:
```bash
rm /config/blueprints/automation/child_night_light.yaml
```

Then reload automations.

---

**Quick Links**:
- [Full Documentation](CHILD_NIGHT_LIGHT_README.md)
- [Scientific Research References](CHILD_NIGHT_LIGHT_README.md#research-references)
- [Troubleshooting Guide](CHILD_NIGHT_LIGHT_README.md#troubleshooting)
- [Example Configurations](CHILD_NIGHT_LIGHT_EXAMPLES.md)

**Need help?** Check the troubleshooting section or review Home Assistant logs for specific error messages.
