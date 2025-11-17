# Circadian Rhythm Lighting - Quick Reference

## Color Temperature Schedule

```
2000K ┤     ╭─────────────────────────────╮     
      │    ╱                               ╲    
3000K ┤   ╱                                 ╲   
      │  ╱                                   ╲  
4500K ┤ ╱                                     ╲ 
      │╱                                       ╲
5500K ┼                                         
      
Time: 5am   7am   10am   3pm    6pm   9pm   5am
      └─┬─┘ └─┬─┘ └─┬──┘ └─┬──┘ └─┬─┘ └─┬──┘
      Dawn  Morn Midday  Aftn  Even  Night
```

## Phase Details

| Time | Phase | Kelvin | Mireds | Brightness* | Effect |
|------|-------|--------|--------|-------------|---------|
| 5am-7am | Dawn | 2000K | 500 | 40% | Gentle wake-up |
| 7am-10am | Morning | 3000K | 333 | 70% | Comfortable start |
| 10am-3pm | Midday | 5500K | 182 | 100% | Maximum alertness |
| 3pm-6pm | Afternoon | 4500K | 222 | 80% | Sustained energy |
| 6pm-9pm | Evening | 3000K | 333 | 50% | Wind-down begins |
| 9pm-5am | Night | 2200K | 455 | 20% | Sleep-friendly |

*When brightness adjustment is enabled

## Input Summary

### Required
- **Target Lights**: Lights that support color temperature
- **Motion Sensors**: One or more motion sensors

### Optional But Recommended
- **Lux Sensor**: Prevents daytime activation
- **Lux Threshold**: Default 100 lux

### Behavior Settings
- **Light Timeout**: Default 300s (5 min)
- **Adjust Brightness**: Default OFF

### Time Customization
- All 6 phase start times are customizable
- Defaults match typical sleep/wake schedules

## How It Works

```
Motion Detected
      ↓
Lux Check (if sensor configured)
      ↓
Below threshold? OR No lux sensor?
      ↓ YES
Determine current circadian phase
      ↓
Turn on lights with:
  - Appropriate color temperature
  - Appropriate brightness (if enabled)
      ↓
Wait for motion to clear + timeout
      ↓
Turn off lights (2s fade)
```

## Compatibility

### Works With
- Philips Hue (White Ambiance, White & Color)
- IKEA TRADFRI (Tunable White)
- Sengled (Color Changing models)
- LIFX (White to Warm, Color models)
- Any light supporting `color_temp` or `kelvin`

### Does NOT Work With
- RGB-only lights (no white spectrum)
- Fixed color temperature lights
- Simple on/off switches/bulbs

## Common Use Cases

### Bedroom
```yaml
Lights: Ceiling, bedside lamps
Motion: Bedroom motion sensor
Lux Sensor: Yes (don't activate during day naps)
Brightness: Enabled (dim at night)
Timeout: 5 minutes
```

### Bathroom
```yaml
Lights: Vanity, ceiling
Motion: Bathroom motion sensor
Lux Sensor: Optional
Brightness: Enabled
Timeout: 3 minutes (short visits)
```

### Home Office
```yaml
Lights: Desk lamp, overhead
Motion: Office motion sensor
Lux Sensor: No (want light even during day)
Brightness: Enabled (100% midday for productivity)
Timeout: 10 minutes
```

### Hallway
```yaml
Lights: Hallway lights
Motion: Multiple sensors (both ends)
Lux Sensor: Yes
Brightness: Enabled
Timeout: 2 minutes (just passing through)
```

## Tips & Tricks

### Multiple Rooms, Different Schedules
- Create separate automations per room
- Customize phase times for each room's use pattern
- Office: Longer midday phase
- Bedroom: Extra warm settings earlier in evening

### Night Shift Workers
- Shift all times by 12 hours
- Your "midday" might be midnight
- Fully customizable to any schedule

### Seasonal Adjustments
- Winter: Extend warm phases (earlier evening)
- Summer: Extend bright phases (later evening)
- Adjust 2-4 times per year as daylight changes

### Testing
1. Set current time's phase to unusual color (e.g., 2200K during day)
2. Trigger motion
3. Verify correct color appears
4. Return phase time to normal

### Optimization
- Start with defaults, observe for a week
- Note when colors feel wrong for activity
- Adjust phase boundaries to better match your routine
- Fine-tune brightness levels to preference

## Troubleshooting One-Liners

| Problem | Solution |
|---------|----------|
| Lights don't change color | Check light supports color_temp |
| Turn on during day | Add/configure lux sensor |
| Turn off too quickly | Increase timeout duration |
| Turn off while in room | Check motion sensor coverage |
| Wrong color for time | Verify phase start times |
| Too bright at night | Enable brightness, lower night % |
| Too dim during day | Enable brightness, increase day % |

## File Locations

```
/config/blueprints/automation/circadian_rhythm_lighting.yaml
└─ Main blueprint file

Settings → Automations & Scenes → Blueprints
└─ Where to create automation instances
```

## Quick Start Command

```yaml
1. Import blueprint
2. Create automation
3. Select lights (color temp capable)
4. Select motion sensor(s)
5. Save & test
```

That's it! The defaults work great for most use cases.

---

**Full Documentation**: See CIRCADIAN_LIGHTING_README.md
**Installation Help**: See INSTALLATION_GUIDE.md
