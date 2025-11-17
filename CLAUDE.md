# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is a Home Assistant blueprint repository containing a **Circadian Rhythm Lighting** automation blueprint. The blueprint adjusts light color temperature throughout the day based on scientific research to support natural circadian rhythms.

## Repository Structure

```
/blueprints/               # Production blueprint files
  circadian_rhythm_lighting.yaml
/docs/                     # Comprehensive documentation
  CIRCADIAN_LIGHTING_README.md
  INSTALLATION_GUIDE.md
  QUICK_REFERENCE.md
  PROJECT_SUMMARY.md
/examples/                 # Example automation configurations
  EXAMPLE_AUTOMATION.yaml
README.md                  # Main repository documentation
```

## Blueprint Architecture

### Core Design Principles

1. **Philips Hue Optimized**: Uses `color_temp` (mireds) exclusively, not `kelvin`. Home Assistant doesn't allow both in the same service call (causes "two or more values in the same group of exclusion" error).

2. **Six Circadian Phases**: Dawn (2000K) → Morning (3000K) → Midday (5500K) → Afternoon (4500K) → Evening (3000K) → Night (2200K)

3. **Time-Based Logic**: Uses seconds-since-midnight for phase determination with special handling for midnight wraparound (night phase spans from 9pm to 5am).

4. **Pleasant Transitions**:
   - Turn-on: 1.5 second smooth fade-in
   - Turn-off: Three-step "considerate" sequence (dim warning → wait → fade off)
   - Based on Lutron Maestro UX patterns

5. **Restart Mode**: Blueprint uses `mode: restart` so motion detection resets the timer, preventing lights from turning off while room is occupied.

### Key Variables Pattern

The blueprint converts:
- **Kelvin → Mireds**: `mireds = 1,000,000 / kelvin`
- **Time strings → Seconds**: For numeric comparisons
- **Conditional brightness**: Only included in service call when `adjust_brightness` is enabled

### Critical Compatibility Notes

- **NEVER** send both `kelvin` and `color_temp` in the same `light.turn_on` service call
- Use templated `data:` blocks with conditional key inclusion for optional parameters
- Philips Hue requires mireds (not kelvin)

## Testing Workflow

When modifying the blueprint:

1. **Local Testing**: Users test by re-importing the blueprint URL in Home Assistant
2. **URL Format**: `https://raw.githubusercontent.com/leviemartin/HomeAssistant-Repo/main/blueprints/circadian_rhythm_lighting.yaml`
3. **Common Issues**:
   - Color descriptor conflicts (both kelvin + color_temp)
   - Empty brightness values (use conditional templating)
   - Transition values must be numbers (not strings)

## Git Workflow

This repository is managed via **GitHub Desktop** (not command line git). When changes are committed:

```bash
# Commit in command line
cd "/Users/martinlevie/Documents/GitHub/HomeAssistant-Repo"
git add <files>
git commit -m "message"

# Then user pushes via GitHub Desktop application
# Do NOT attempt git push from command line (authentication issues)
```

## Documentation Sync

When modifying the blueprint, update:
1. `blueprints/circadian_rhythm_lighting.yaml` - The source of truth
2. `README.md` - Feature list and quick start
3. `docs/CIRCADIAN_LIGHTING_README.md` - Detailed documentation (if architectural changes)
4. Commit with descriptive message explaining UX impact

## Scientific Color Temperature Values

Based on peer-reviewed research (DO NOT change arbitrarily):
- **2000K**: Warm sunrise/sunset (minimal blue light)
- **2200K**: Night/sleep-friendly (extra warm amber)
- **3000K**: Warm white (morning/evening)
- **4500K**: Neutral white (afternoon)
- **5500K**: Cool daylight (maximum alertness, midday)

These values optimize for melanopsin photopigment response and melatonin regulation.

## User Experience Requirements

- **Smooth transitions**: Never instant (jarring), use 1.5-3 second fades
- **Dim warning**: Two-step turn-off prevents surprising "lights suddenly off" experience
- **User-friendly selectors**: All inputs use dropdowns, no manual entity_id typing
- **Sensible defaults**: Blueprint works out-of-box without configuration tweaking

## Blueprint Import URL

Users import via this raw GitHub URL:
```
https://raw.githubusercontent.com/leviemartin/HomeAssistant-Repo/main/blueprints/circadian_rhythm_lighting.yaml
```

**Not** the regular GitHub page URL (that won't work for Home Assistant import).
