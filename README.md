# Closed-Loop Buck Converter

Design and LTspice simulation of a closed-loop nonsynchronous buck converter using voltage-mode PWM control.
 
This project was created to put into practice and further develop the power electronics knowledge I have gained throughout my professional career. Starting from first principles, I will design and verify the open-loop power stage, implement voltage feedback and compensation, and evaluate the converter's closed-loop behavior under steady-state and transient conditions.

---

## 1. Project Objectives

- Design a DC-DC buck converter from first principles
- Calculate the required duty cycle, inductor, and output capacitor
- Simulate the converter power stage in LTspice
- Understand MOSFET, diode, inductor, and capacitor behavior
- Implement voltage feedback
- Implement a closed-loop voltage-mode controller
- Evaluate startup behavior
- Evaluate load and input-voltage transients
- Analyze component losses and converter efficiency
- Compare calculated values with simulation results

---

## 2. Design Requirements

| Parameter | Requirement |
|---|---:|
| Input voltage range | 10–14 V |
| Nominal input voltage | 12 V |
| Output voltage | 5.0 V |
| Output regulation | ±2% |
| Output current | 0.1–0.5 A |
| Maximum output power | 2.5 W |
| Switching frequency | 100 kHz |
| Output ripple target | <50 mVpp |
| Inductor ripple target | ~30% of full-load current |
| Topology | Nonsynchronous buck |
| Control method | Voltage-mode PWM |
| Feedback reference | 1.0 V |

The regulated output-voltage target is therefore:

4.90 V ≤ VOUT ≤ 5.10 V

---

## 3. Transient Requirements

The converter will also be evaluated under changing operating conditions.

### Load transient

0.1 A → 0.5 A → 0.1 A

Initial target:

- Peak output deviation: <10%
- Recovery into ±2% regulation band: <2 ms

### Input-voltage transient

10 V → 14 V → 10 V

The feedback controller should adjust the PWM duty cycle to return VOUT to 5 V.

### Startup

Initial condition:

VOUT(0) = 0 V

Targets:

- Settling time: <5 ms
- Maximum startup overshoot: <5.5 V

---

## 4. Converter Architecture

The final converter architecture will consist of:

```text
                      POWER STAGE

VIN ── MOSFET ── SW ── Inductor ───── VOUT
                 │                      │
               Diode                 Capacitor
                 │                      │
                GND                   Load
                                        │
                                       GND


                     CONTROL LOOP

VOUT
  │
  ▼
Feedback Divider
  │
  ▼
Error Amplifier ◄──── VREF
  │
  ▼
Compensator
  │
  ▼
PWM Comparator ◄──── Ramp
  │
  ▼
Gate Driver
  │
  ▼
MOSFET
```

---

## 5. Initial Design Calculations

### Ideal Duty Cycle

For an ideal buck converter:

D = VOUT / VIN

At the nominal operating point:

D = 5 V / 12 V

D = 0.417

Therefore, the ideal nominal duty cycle is approximately:

**D = 41.7%**

The practical duty cycle will be slightly different because of diode and switching losses.

---

### Load Resistance

At maximum output current:

RLOAD = VOUT / IOUT

RLOAD = 5 V / 0.5 A

**RLOAD = 10 Ω**

At the minimum design load of 0.1 A:

RLOAD = 5 V / 0.1 A

**RLOAD = 50 Ω**

---

### Inductor Ripple Target

Target inductor-current ripple:

ΔIL = 30% × IOUT,max

ΔIL = 0.30 × 0.5 A

**ΔIL = 0.15 App**

The inductor value will be calculated from this requirement.

---

### Output Capacitor

The output capacitor will be selected to achieve:

**ΔVOUT < 50 mVpp**

The effect of capacitance and ESR will be evaluated in simulation.

---

## 6. Feedback Network

The controller uses a 1.0 V reference.

The feedback divider must therefore generate:

VFB = 1.0 V

when:

VOUT = 5.0 V

Initial divider:

```text
5 V OUT
   │
  40 kΩ
   │
   ├──── VFB = 1 V
   │
  10 kΩ
   │
  GND
```

The divider equation is:

VFB = VOUT × RBOTTOM / (RTOP + RBOTTOM)

Therefore:

VFB = 5 × 10k / (40k + 10k)

**VFB = 1.0 V**

---

## 7. Development Plan

The project is being developed incrementally.

### Phase 1: Requirements
- [x] Define input-voltage range
- [x] Define output voltage
- [x] Define maximum load
- [x] Define switching frequency
- [x] Define voltage/current ripple targets

### Phase 2: Power Stage Design
- [x] Calculate nominal duty cycle
- [x] Calculate load resistance
- [x] Define inductor ripple requirement
- [ ] Calculate inductor value
- [ ] Calculate output capacitor
- [ ] Select diode
- [ ] Select MOSFET

### Phase 3: Open-Loop Simulation
- [ ] Build open-loop buck in LTspice
- [ ] Verify switching-node voltage
- [ ] Verify inductor current
- [ ] Verify output voltage
- [ ] Compare calculated and simulated ripple
- [ ] Sweep input voltage
- [ ] Sweep load

### Phase 4: Feedback
- [ ] Implement feedback divider
- [ ] Add voltage reference
- [ ] Implement error amplifier
- [ ] Verify error response

### Phase 5: PWM Controller
- [ ] Generate PWM ramp
- [ ] Add error compensation
- [ ] Add PWM comparator
- [ ] Verify control voltage versus duty cycle

### Phase 6: Closed-Loop Operation
- [ ] Close the feedback loop
- [ ] Verify 5 V regulation
- [ ] Analyze startup response
- [ ] Perform load transient test
- [ ] Perform line transient test

### Phase 7: Stability
- [ ] Tune compensation
- [ ] Analyze overshoot
- [ ] Analyze settling time
- [ ] Evaluate loop stability
- [ ] Investigate loop gain and phase margin

### Phase 8: Real Components
- [ ] Replace ideal switch with MOSFET model
- [ ] Add realistic diode model
- [ ] Include inductor DCR
- [ ] Include capacitor ESR
- [ ] Evaluate switching losses

### Phase 9: Efficiency
- [ ] Calculate input power
- [ ] Calculate output power
- [ ] Calculate efficiency
- [ ] Sweep efficiency versus load
- [ ] Identify major loss contributors

---

## 8. Verification Plan

| Test | VIN | Load | Verification |
|---|---:|---:|---|
| Nominal operation | 12 V | 0.5 A | 5 V regulation |
| Light load | 12 V | 0.1 A | Regulation |
| Minimum VIN | 10 V | 0.5 A | Line regulation |
| Maximum VIN | 14 V | 0.5 A | Line regulation |
| Startup | 12 V | 0.5 A | Settling and overshoot |
| Load step | 12 V | 0.1 → 0.5 A | Transient response |
| Load release | 12 V | 0.5 → 0.1 A | Transient response |
| Line transient | 10 → 14 V | 0.5 A | Control response |
| Full load | 12 V | 0.5 A | Output ripple |
| Full load | 12 V | 0.5 A | Efficiency |

---

## 9. Simulation Results

### Open-Loop Operation

*Results will be added after completion of the open-loop power-stage simulation.*

Planned plots:

- Switching-node voltage
- Inductor current
- Output-voltage startup
- Output ripple

### Closed-Loop Operation

*Results will be added after implementation of the feedback controller.*

Planned plots:

- Output-voltage startup
- Feedback voltage
- Error-amplifier output
- PWM duty cycle
- Load-step response
- Line-transient response

---

## 10. Efficiency Target

After nonideal component models are introduced, the initial full-load efficiency target is:

**η > 85%**

at:

- VIN = 12 V
- VOUT = 5 V
- IOUT = 0.5 A

Efficiency will be calculated using:

η = POUT / PIN × 100%

An efficiency versus load-current curve will be generated.

---

## 11. Repository Structure

```text
closed-loop-buck-converter/
│
├── README.md
│
├── LTSPICE/
│   ├── 01_open_loop.asc
│   ├── 02_feedback.asc
│   ├── 03_closed_loop.asc
│   └── 04_load_step.asc
│
├── calculations/
│   └── design_calculations.md
│
├── results/
│   ├── switching_node.png
│   ├── inductor_current.png
│   ├── startup.png
│   ├── load_transient.png
│   └── efficiency.png
│
└── docs/
    └── final_report.pdf
```

---

## 12. Tools

- LTspice
- GitHub
- Analytical power-electronics calculations

---

## 13. Project Status

**Current stage: Power-stage design**

Next steps:

1. Calculate the required inductance from the 150 mApp ripple specification
2. Calculate the output capacitance from the 50 mVpp ripple specification
3. Select appropriate MOSFET and diode ratings
4. Build the open-loop LTspice model
5. Compare calculated and simulated results

---

## 14. Lessons Learned

This section will document design decisions, unexpected simulation behavior, debugging steps, and differences between theoretical calculations and simulation results as the project progresses.
