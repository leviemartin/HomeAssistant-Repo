# Anti-Flicker Hysteresis Technical Guide

Deep dive into the hysteresis-based anti-flicker system that prevents annoying light oscillation.

## Table of Contents

1. [The Problem: Light Flickering](#the-problem-light-flickering)
2. [Understanding Hysteresis](#understanding-hysteresis)
3. [How This Blueprint Solves It](#how-this-blueprint-solves-it)
4. [Debounce Timer System](#debounce-timer-system)
5. [Sunrise/Sunset Special Handling](#sunrisesunset-special-handling)
6. [Mathematical Analysis](#mathematical-analysis)
7. [Tuning Guide](#tuning-guide)
8. [Real-World Scenarios](#real-world-scenarios)

---

## The Problem: Light Flickering

### What is Light Flickering?

Light flickering occurs when automated lights rapidly cycle ON → OFF → ON → OFF due to:
- **Passing clouds** (lux rapidly changes)
- **Sunrise/sunset transitions** (slow but continuous lux change)
- **Movement near lux sensor** (temporary shadows)
- **Single-threshold automation** (same value for on/off decisions)

### Why It Happens

Traditional single-threshold automation:
```yaml
# BAD EXAMPLE - Single threshold
condition:
  - condition: numeric_state
    entity_id: sensor.lux
    below: 100

# Result:
# Lux 98 → Lights ON
# Lux 102 → Lights OFF
# Lux 99 → Lights ON  ← FLICKER!
# Lux 101 → Lights OFF ← FLICKER!
```

**Problem**: Lux hovers around threshold → lights flicker endlessly

### Real-World Example

**Cloudy afternoon scenario**:
```
2:00 PM - Cloud passes → Lux drops 120 → 95 → Lights turn ON
2:02 PM - Cloud clears → Lux rises 95 → 125 → Lights turn OFF
2:05 PM - Another cloud → Lux drops 125 → 90 → Lights turn ON
2:07 PM - Cloud clears → Lux rises 90 → 130 → Lights turn OFF
... repeats 20+ times in afternoon
```

**User experience**: Annoying, disruptive, feels "broken"

---

## Understanding Hysteresis

### What is Hysteresis?

**Definition**: Using different thresholds for turning ON vs turning OFF to create a "dead band" that prevents oscillation.

**Origin**: Borrowed from control systems engineering (thermostats, electronics, industrial processes)

### Visual Explanation

#### Single Threshold (Flickering)
```
Lux Value:  0 -------- 100 -------- 200 -------- 300
                        ↑
                   Same threshold
                   (100 lux)

Behavior near 100 lux:
99 lux → ON
101 lux → OFF
98 lux → ON   ← Oscillation!
102 lux → OFF ← Oscillation!
```

#### Dual Threshold (Hysteresis - No Flickering)
```
Lux Value:  0 -------- 80 -------- 150 -------- 300
                       ↑             ↑
                    Turn-ON       Turn-OFF
                   threshold      threshold

                   [Dead Band: 80-150 lux]
                   (No automatic changes)

Behavior in dead band:
85 lux → No change (stay in current state)
95 lux → No change
110 lux → No change
145 lux → No change
```

### State Machine Diagram

```
                ┌──────────────┐
                │  LIGHTS OFF  │
                └──────┬───────┘
                       │
       Lux < 80        │        Lux > 150
       (for 2 min)     │        (ignored)
                       │
                       ↓
                ┌──────────────┐
                │  LIGHTS ON   │
                └──────┬───────┘
                       │
       Lux < 80        │        Lux > 150
       (ignored)       │        (for 5 min)
                       │
                       ↓
                ┌──────────────┐
                │  LIGHTS OFF  │
                └──────────────┘
```

**Key insight**: Different thresholds depending on current state

---

## How This Blueprint Solves It

### Three-Layer Protection System

#### Layer 1: Dual Thresholds (Hysteresis)

```yaml
Turn-ON Threshold:  80 lux   (lower threshold)
Turn-OFF Threshold: 150 lux  (upper threshold)
Hysteresis Gap:     70 lux   (difference between thresholds)
```

**Effect**: Creates 80-150 lux "dead band" where no automatic changes occur

#### Layer 2: Debounce Timers

```yaml
Turn-ON Debounce:   2 minutes (120 seconds)
Turn-OFF Debounce:  5 minutes (300 seconds)
```

**Effect**: Lux must stay above/below threshold for full duration before action

#### Layer 3: Sunrise/Sunset Extended Debounce

```yaml
Normal Turn-OFF Debounce:     5 minutes
Dawn/Dusk Turn-OFF Debounce:  10 minutes
Detection Window:             1 hour before/after sunrise/sunset
```

**Effect**: Extra stability during slow gradual transitions

### Complete Logic Flow

```yaml
# Pseudocode for turn-ON decision
IF lights are OFF:
  IF lux < 80:
    START timer (2 minutes)
    IF lux stays < 80 for entire 2 minutes:
      IF presence detected:
        Turn lights ON
    ELSE:
      CANCEL timer (lux rose above 80 before 2 min elapsed)
```

```yaml
# Pseudocode for turn-OFF decision
IF lights are ON:
  IF lux > 150:
    IF currently dawn or dusk (1 hr window):
      START timer (10 minutes extended)
    ELSE:
      START timer (5 minutes normal)

    IF lux stays > 150 for entire timer duration:
      IF override is NOT active:
        Turn lights OFF
    ELSE:
      CANCEL timer (lux dropped below 150 before timer elapsed)
```

### Blueprint Implementation

**Variables Section**:
```yaml
variables:
  lux_on_threshold: !input lux_turn_on_threshold
  lux_off_threshold: !input lux_turn_off_threshold
  lux_on_debounce_sec: !input lux_turn_on_debounce
  lux_off_debounce_sec: !input lux_turn_off_debounce
  sunrise_sunset_debounce_sec: !input sunrise_sunset_extended_debounce

  current_lux: "{{ states(lux_sensor_entity) | float(0) }}"

  # Smart debounce selection
  effective_off_debounce: >
    {% if is_dawn_or_dusk %}
      {{ sunrise_sunset_debounce_sec }}
    {% else %}
      {{ lux_off_debounce_sec }}
    {% endif %}
```

**Turn-ON Logic** (simplified excerpt):
```yaml
- condition: template
  value_template: "{{ current_lux < lux_on_threshold }}"

- wait_template: "{{ states(lux_sensor_entity) | float < lux_on_threshold }}"
  timeout:
    seconds: "{{ lux_on_debounce_sec }}"
  continue_on_timeout: false  # If lux rises during wait, stop

- service: light.turn_on  # Only reached if lux stayed low
```

**Turn-OFF Logic** (simplified excerpt):
```yaml
- condition: template
  value_template: "{{ not override_active and (current_lux >= lux_off_threshold) }}"

- wait_template: "{{ states(lux_sensor_entity) | float >= lux_off_threshold }}"
  timeout:
    seconds: "{{ effective_off_debounce }}"  # Smart debounce
  continue_on_timeout: false  # If lux drops during wait, stop

- service: light.turn_off  # Only reached if lux stayed high
```

---

## Debounce Timer System

### What is Debounce?

**Definition**: Waiting for a signal to stabilize before acting on it.

**Origin**: Electronics term for eliminating "bounce" in mechanical switches

### Why Multiple Debounce Timers?

Different situations require different waiting periods:

| Scenario | Debounce | Reason |
|----------|----------|--------|
| **Turn-ON** | 2 minutes | Fast response when room darkens |
| **Turn-OFF** | 5 minutes | Longer wait (more annoying to turn off too soon) |
| **Dawn/Dusk** | 10 minutes | Slow gradual change requires extra patience |

### Turn-ON Debounce (2 minutes default)

**Purpose**: Prevent lights turning on from brief shadows

**Example**:
```
12:30 - Large truck drives by window → Shadow cast
12:30:05 - Lux drops 95 → 70 (below 80 threshold)
12:30:05 - Debounce timer starts (2 minutes)
12:30:45 - Truck passes → Shadow gone
12:30:45 - Lux rises 70 → 95 (above 80 threshold)
12:30:45 - Debounce timer CANCELLED
Result: Lights did NOT turn on (shadow was temporary)
```

**Tuning**:
- **Decrease** (30-60 sec): Faster response, more false positives
- **Increase** (3-5 min): Slower response, fewer false positives

### Turn-OFF Debounce (5 minutes default)

**Purpose**: Prevent lights turning off from brief bright spots

**Example**:
```
3:00 PM - Cloud clears → Lux spikes 130 → 180 (above 150 threshold)
3:00:00 - Debounce timer starts (5 minutes)
3:02:30 - Another cloud → Lux drops 180 → 140 (below 150 threshold)
3:02:30 - Debounce timer CANCELLED
Result: Lights stayed ON (bright period was temporary)
```

**Why longer than turn-ON?**
- More disruptive to turn lights off when you're using them
- Better to err on side of "leave lights on" vs "turn off too soon"
- Users tolerate lights staying on > lights turning off prematurely

**Tuning**:
- **Decrease** (2-3 min): Faster response to natural light, more flicker risk
- **Increase** (7-10 min): Maximum stability, slower response

### Sunrise/Sunset Extended Debounce (10 minutes default)

**Purpose**: Extra stability during dawn/dusk slow transitions

**Detection**:
```yaml
sunrise_timestamp: 7:30 AM (example)
Detection window: 6:30 AM - 8:30 AM (1 hour before/after)

sunset_timestamp: 6:45 PM (example)
Detection window: 5:45 PM - 7:45 PM (1 hour before/after)
```

**Why extended?**
- Sunrise/sunset is gradual (not abrupt like clouds)
- Lux changes slowly over 30-60 minutes
- Normal 5-minute debounce insufficient for slow drift
- 10-minute debounce ensures lux truly stabilized

**Example**:
```
6:00 AM - Sunrise begins (lux slowly rising)
6:10 AM - Lux crosses 150 threshold (6:10-7:10 detection window active)
6:10:00 - Extended debounce starts (10 minutes)
6:15 AM - Lux continues rising (still above 150)
6:20:00 - 10 minutes elapsed → Lux confirmed high → Lights turn OFF
```

**Tuning**:
- **Decrease** (5-7 min): Faster sunrise response, may oscillate
- **Increase** (15-20 min): Maximum dawn/dusk stability

---

## Sunrise/Sunset Special Handling

### Why Dawn/Dusk is Different

**Typical cloud**:
```
Duration: 2-10 minutes
Lux change: Abrupt (drops/rises quickly)
Pattern: Irregular
```

**Sunrise/Sunset**:
```
Duration: 30-90 minutes
Lux change: Gradual (smooth continuous)
Pattern: Predictable daily
```

**Problem with normal debounce**:
```
6:00 AM - Sunrise begins (lux: 50 → slowly rising)
6:08 AM - Lux crosses 150 (turn-off threshold)
6:08 AM - 5-min debounce starts
6:10 AM - Lux still rising (155)
6:13 AM - Lights turn off (5 min elapsed, lux confirmed high)
6:25 AM - Lux drops back to 145 (sun angle changes)
6:25 AM - Lights turn back ON (lux < 150 again)
← Oscillation during sunrise!
```

### Blueprint Solution

**Detection Logic**:
```yaml
# Calculate sunrise/sunset timestamps
sunrise_timestamp: "{{ as_timestamp(state_attr('sun.sun', 'next_rising')) | int }}"
sunset_timestamp: "{{ as_timestamp(state_attr('sun.sun', 'next_setting')) | int }}"
current_timestamp: "{{ now().timestamp() | int }}"

# Check if within 1 hour before/after sunrise
is_near_sunrise: >
  {{ (current_timestamp >= (sunrise_timestamp - 3600) and
      current_timestamp <= (sunrise_timestamp + 3600)) }}

# Check if within 1 hour before/after sunset
is_near_sunset: >
  {{ (current_timestamp >= (sunset_timestamp - 3600) and
      current_timestamp <= (sunset_timestamp + 3600)) }}

# Combined dawn/dusk detection
is_dawn_or_dusk: >
  {{ is_near_sunrise or is_near_sunset }}
```

**Smart Debounce Selection**:
```yaml
effective_off_debounce: >
  {% if is_dawn_or_dusk %}
    {{ sunrise_sunset_debounce_sec }}  # 10 minutes
  {% else %}
    {{ lux_off_debounce_sec }}  # 5 minutes
  {% endif %}
```

**Usage in Turn-OFF Logic**:
```yaml
- wait_template: "{{ states(lux_sensor_entity) | float >= lux_off_threshold }}"
  timeout:
    seconds: "{{ effective_off_debounce }}"  # Automatically selects correct timer
```

### Visual Timeline

```
Sunrise Example (Summer, 6:30 AM sunrise):

5:30 AM ──────────────┐
                       │ 1-hour detection window STARTS
6:00 AM               │
                       │ (Extended 10-min debounce active)
6:30 AM ─── SUNRISE ──┤
                       │ (Extended 10-min debounce active)
7:00 AM               │
                       │ 1-hour detection window ENDS
7:30 AM ──────────────┘

Outside window: Normal 5-min debounce
Inside window: Extended 10-min debounce
```

---

## Mathematical Analysis

### Hysteresis Gap Calculation

**Formula**:
```
Hysteresis Gap = Turn-OFF Threshold - Turn-ON Threshold
```

**Example**:
```
Turn-ON:  80 lux
Turn-OFF: 150 lux
Gap:      150 - 80 = 70 lux
```

### Stability Region

**Definition**: Lux range where lights remain in current state regardless of small fluctuations

```
State: LIGHTS OFF
Stable region: 80 lux to infinity (will stay off unless lux < 80)

State: LIGHTS ON
Stable region: 0 lux to 150 lux (will stay on unless lux > 150)

Dead band (either state stable): 80 lux to 150 lux
```

### Flicker Prevention Calculation

**Required gap to prevent single-threshold flicker**:
```
Minimum Gap = 2 × (Maximum Lux Fluctuation)

Example:
Cloudy day lux fluctuations: ±30 lux
Minimum gap needed: 2 × 30 = 60 lux
Recommended: 70-100 lux (safety margin)
```

### Debounce Effectiveness

**Probability of false trigger**:
```
P(false) = P(lux stays beyond threshold for T seconds)

Where:
- T = debounce duration
- P(lux stays) decreases exponentially with T

Example with passing clouds (avg duration 3 minutes):
T = 30 sec → P(false) ≈ 40% (high)
T = 2 min  → P(false) ≈ 15% (moderate)
T = 5 min  → P(false) ≈ 3% (low)
T = 10 min → P(false) ≈ 0.5% (very low)
```

### Optimal Threshold Selection

**Given**:
- Room lux distribution throughout day
- Desired lighting activation point

**Calculate**:
```
Step 1: Measure lux when "too dark" (want lights on)
Example: 70 lux

Step 2: Measure lux when "bright enough" (want lights off)
Example: 180 lux

Step 3: Calculate thresholds with safety margin
Turn-ON = "too dark" + 10 lux = 80 lux
Turn-OFF = "bright enough" - 30 lux = 150 lux

Step 4: Verify gap
Gap = 150 - 80 = 70 lux ✓ (adequate)
```

---

## Tuning Guide

### Step-by-Step Tuning Process

#### Phase 1: Baseline Testing (3-7 days)

**Goal**: Observe behavior with default settings

1. **Install with defaults**:
   ```
   Turn-ON: 80 lux
   Turn-OFF: 150 lux
   Turn-ON Debounce: 2 min
   Turn-OFF Debounce: 5 min
   Sunrise/Sunset: 10 min
   ```

2. **Monitor and log**:
   - Times when lights flicker
   - Times when lights turn on/off unexpectedly
   - Lux readings during problem events

3. **Identify issues**:
   - Frequent flickering → Increase gap or debounce
   - Lights stay on too long → Decrease turn-OFF threshold
   - Lights turn on too early → Decrease turn-ON threshold

#### Phase 2: Threshold Adjustment

**Lux Measurement Exercise**:

1. **Morning (curtains closed)**:
   - Developer Tools → States → sensor.lux
   - Record: `_____ lux`
   - Want lights ON at this level?

2. **Afternoon (curtains open, cloudy)**:
   - Record: `_____ lux`
   - Want lights ON or OFF at this level?

3. **Afternoon (curtains open, sunny)**:
   - Record: `_____ lux`
   - Want lights OFF at this level?

**Calculate New Thresholds**:
```
Target Turn-ON = (Morning lux + Afternoon cloudy lux) / 2
Example: (60 + 100) / 2 = 80 lux ✓

Target Turn-OFF = Afternoon sunny lux × 0.8
Example: 200 × 0.8 = 160 lux

Gap = 160 - 80 = 80 lux ✓ (good)
```

#### Phase 3: Debounce Optimization

**Test Matrix**:

| Scenario | Current Behavior | Desired Behavior | Action |
|----------|-----------------|-----------------|---------|
| Quick cloud pass (2 min) | Lights turn on/off | Ignore completely | Increase turn-ON to 3 min |
| Sustained overcast (10 min) | Lights turn on too late | Faster activation | Decrease turn-ON to 1 min |
| Sunrise | Lights oscillate | Stable | Increase sunrise debounce to 15 min |
| Curtain opening | Lights turn off too fast | Slower response | Increase turn-OFF to 7 min |

**General Rules**:
```
IF flickering on cloudy days:
  → Increase turn-OFF debounce (5 → 7 → 10 min)
  OR increase hysteresis gap (70 → 90 → 100 lux)

IF lights don't turn on when needed:
  → Decrease turn-ON debounce (2 → 1 min)
  OR increase turn-ON threshold (80 → 100 lux)

IF lights don't turn off when bright:
  → Decrease turn-OFF threshold (150 → 120 lux)
  OR decrease turn-OFF debounce (5 → 3 min) [caution!]

IF sunrise/sunset oscillation:
  → Increase extended debounce (10 → 15 → 20 min)
```

#### Phase 4: Fine-Tuning (Advanced)

**Per-Condition Optimization**:

**Weather-Dependent** (requires manual adjustment):
```
Stable sunny day:
  - Can use shorter debounce (3 min)
  - Narrower gap acceptable (50 lux)

Variable cloudy day:
  - Use longer debounce (7-10 min)
  - Wider gap needed (100 lux)

Winter (low sun angle):
  - Extend sunrise/sunset window (±90 min instead of 60 min)
  - Increase extended debounce (15 min)
```

**Room-Specific**:
```
South-facing (very bright):
  - Higher turn-OFF threshold (200+ lux)
  - Wider gap (120+ lux)

North-facing (darker):
  - Lower turn-ON threshold (60 lux)
  - Narrower gap acceptable (50 lux)
```

---

## Real-World Scenarios

### Scenario 1: Typical Cloudy Afternoon

**Setup**:
- Turn-ON: 80 lux
- Turn-OFF: 150 lux
- Turn-ON Debounce: 2 min
- Turn-OFF Debounce: 5 min
- Current state: Lights ON, 120 lux

**Event Timeline**:
```
2:00 PM - Cloud passes over (lux: 120 → 95)
  Action: None (lux still above turn-ON threshold, lights stay ON)

2:05 PM - Cloud thickens (lux: 95 → 75)
  Action: None (lux < 80, but lights already ON - no action needed)

2:10 PM - Cloud clears (lux: 75 → 165)
  Action: Lux > 150 → Start 5-min turn-OFF debounce timer

2:12 PM - Another cloud (lux: 165 → 140)
  Action: Lux dropped below 150 → CANCEL turn-OFF timer
  Result: Lights stay ON (cloud prevented premature turn-off)

2:20 PM - Sun fully out (lux: 140 → 190)
  Action: Lux > 150 → Start 5-min turn-OFF debounce timer (again)

2:25 PM - Lux stable at 190 for 5 minutes
  Action: Turn lights OFF (debounce completed, lux confirmed high)
```

**Outcome**: No flickering despite variable clouds. Lights only turned off when lux stable > 150 for full 5 minutes.

---

### Scenario 2: Sunrise Transition

**Setup**:
- Turn-ON: 80 lux
- Turn-OFF: 150 lux
- Normal debounce: 5 min
- Extended debounce: 10 min
- Sunrise: 6:30 AM

**Event Timeline**:
```
5:30 AM - Pre-dawn (lux: 15)
  State: Lights ON (dark room)

6:00 AM - Dawn begins (lux: 15 → 50, slowly rising)
  State: Lights ON (still below 80 lux)
  Note: Within 1-hour sunrise window (5:30-7:30 AM)

6:15 AM - Civil twilight (lux: 50 → 100)
  State: Lights ON (below 150 lux)

6:25 AM - Lux crosses threshold (lux: 100 → 152)
  Action: Lux > 150 + dawn/dusk detected
  Action: Start EXTENDED 10-min debounce timer

6:30 AM - Sunrise (lux: 152 → 180)
  Timer: 5 minutes elapsed (5 more to go)

6:32 AM - Brief cloud (lux: 180 → 145)
  Action: Lux dropped below 150 → CANCEL timer
  Result: Lights stay ON (temporary dip prevented turn-off)

6:35 AM - Cloud clears (lux: 145 → 190)
  Action: Lux > 150 → Start new 10-min extended timer

6:45 AM - Lux stable at 190-210 for 10 minutes
  Action: Turn lights OFF (confirmed bright, sunrise complete)
```

**Outcome**: Extended debounce prevented oscillation during slow sunrise. Lights turned off only when lux confirmed stable and high.

---

### Scenario 3: Passing Shadow

**Setup**:
- Turn-ON: 80 lux
- Turn-OFF: 150 lux
- Turn-ON Debounce: 2 min
- Current state: Lights OFF, 95 lux

**Event Timeline**:
```
12:00 PM - Normal light (lux: 95)
  State: Lights OFF (room bright enough)

12:01 PM - Large truck parks outside window (shadow)
  Event: Lux drops 95 → 70 in 10 seconds

12:01:10 - Lux < 80 detected
  Action: Start 2-min turn-ON debounce timer

12:01:50 - Truck drives away (shadow gone)
  Event: Lux rises 70 → 95 in 5 seconds

12:01:55 - Lux > 80 detected
  Action: CANCEL turn-ON timer (only 45 seconds elapsed, need 2 min)
  Result: Lights did NOT turn on (shadow was temporary)

12:02 PM - Normal light restored (lux: 95)
  State: Lights still OFF (correctly avoided false trigger)
```

**Outcome**: Turn-ON debounce prevented lights from turning on for brief 50-second shadow. System correctly identified temporary condition.

---

### Scenario 4: Sustained Overcast

**Setup**:
- Turn-ON: 80 lux
- Turn-OFF: 150 lux
- Turn-ON Debounce: 2 min
- Turn-OFF Debounce: 5 min
- Current state: Lights OFF, 110 lux

**Event Timeline**:
```
10:00 AM - Partly cloudy (lux: 110)
  State: Lights OFF (sufficient natural light)

10:15 AM - Overcast moves in (lux: 110 → 75 over 3 minutes)

10:18 AM - Lux < 80 detected (lux: 75)
  Action: Start 2-min turn-ON debounce timer

10:20 AM - Lux continues low (lux: 70)
  Timer: 2 minutes elapsed, lux stayed < 80 entire time

10:20:00 - Presence detected in room
  Action: Turn lights ON (debounce confirmed, presence active)

10:20 AM - 1:00 PM - Sustained overcast (lux: 65-75)
  State: Lights ON (appropriate for dark conditions)

1:00 PM - Clouds begin breaking up (lux: 75 → 155 over 10 minutes)

1:10 PM - Lux > 150 detected (lux: 155)
  Action: Start 5-min turn-OFF debounce timer

1:12 PM - Lux dips briefly (lux: 155 → 145 for 1 minute)
  Action: CANCEL turn-OFF timer (lux dropped below threshold)

1:13 PM - Lux rises again (lux: 145 → 170)
  Action: Start new 5-min turn-OFF timer

1:18 PM - Lux stable at 170-180 for 5 minutes
  Action: Turn lights OFF (confirmed bright, clouds cleared)
```

**Outcome**: System correctly activated lights during sustained dark period, then deactivated when brightness stabilized. Brief dip in lux didn't cause premature turn-off.

---

### Scenario 5: Edge Case - Lux Hovering in Dead Band

**Setup**:
- Turn-ON: 80 lux
- Turn-OFF: 150 lux
- Current state: Lights ON, presence active
- Lux hovering: 100-140 lux (within dead band)

**Event Timeline**:
```
3:00 PM - Variable clouds (lux oscillating 100-140)
  State: Lights ON (entered this state earlier when lux was < 80)

3:05 PM - Lux: 105
  Action: None (lux in dead band, no thresholds crossed)

3:10 PM - Lux: 135
  Action: None (still in dead band, below 150 turn-OFF threshold)

3:15 PM - Lux: 110
  Action: None (dead band - stable state maintained)

3:20 PM - Lux: 125
  Action: None (lights remain ON until lux > 150 for 5 minutes)

3:25 PM - Lux: 140
  Action: None (still below 150 threshold)

... Lights stay ON entire time despite lux variations ...
... No flickering, no oscillation ...
```

**Outcome**: Dead band (80-150 lux) successfully prevents any automatic changes. Lights stay in current state regardless of lux variations within band. This is the hysteresis system working perfectly.

---

## Conclusion

### Key Takeaways

1. **Hysteresis is essential** for flicker-free lux-based automation
2. **Dual thresholds** (80 ON / 150 OFF) create stable dead band
3. **Debounce timers** prevent false triggers from brief changes
4. **Extended dawn/dusk handling** stabilizes slow transitions
5. **Tuning is room-specific** - measure your actual lux values
6. **Wider gap = more stable** but less responsive
7. **Longer debounce = more stable** but slower response

### Quick Reference Card

```
┌─────────────────────────────────────────────────────┐
│           ANTI-FLICKER QUICK REFERENCE              │
├─────────────────────────────────────────────────────┤
│ DEFAULT SETTINGS (Recommended Start)                │
│   Turn-ON Threshold:    80 lux                      │
│   Turn-OFF Threshold:   150 lux                     │
│   Gap:                  70 lux                      │
│   Turn-ON Debounce:     2 minutes                   │
│   Turn-OFF Debounce:    5 minutes                   │
│   Dawn/Dusk Debounce:   10 minutes                  │
├─────────────────────────────────────────────────────┤
│ TROUBLESHOOTING                                     │
│   Flickering?          → Increase gap OR debounce   │
│   Late turn-on?        → Increase ON threshold      │
│   Late turn-off?       → Decrease OFF threshold     │
│   Sunrise oscillation? → Increase extended debounce │
├─────────────────────────────────────────────────────┤
│ TUNING RANGES                                       │
│   Gap:              30-150 lux (sweet spot: 70-100) │
│   Turn-ON Debounce: 30 sec - 10 min (default: 2)   │
│   Turn-OFF Debounce: 1-15 min (default: 5)         │
│   Extended Debounce: 5-20 min (default: 10)        │
└─────────────────────────────────────────────────────┘
```

### Further Resources

- **Blueprint file**: See extensive inline comments for implementation details
- **Setup guide**: `INTELLIGENT_LIVING_ROOM_SETUP_GUIDE.md` for practical configuration
- **FP2 reference**: `FP2_FEATURES_REFERENCE.md` for sensor-specific setup

---

**Document Version**: 1.0.0
**Last Updated**: 2025-11-17
**Applies to Blueprint**: `intelligent_living_room_mmwave_lux_aware.yaml`
