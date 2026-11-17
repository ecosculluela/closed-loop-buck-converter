# Bang-Bang Controlled Buck Converter

Design and LTspice simulation of a closed-loop nonsynchronous buck converter using **bang-bang (hysteretic) voltage control**.

This project was created to put into practice and further develop the power electronics knowledge I have gained throughout my professional career. Starting from first principles, I will design and verify the buck converter power stage, implement a hysteretic feedback controller, and evaluate the converter under steady-state, startup, load-transient, and input-voltage-transient conditions.

The main objective is to demonstrate how a simple nonlinear controller can regulate the output voltage **without an error amplifier, compensation network, PWM ramp, or fixed-frequency PWM modulator**.

---

## 1. Project Objectives

- Design a DC-DC buck converter from first principles
- Calculate the required inductor and capacitors
- Simulate the power stage in LTspice
- Understand MOSFET, diode, inductor, and capacitor behavior
- Implement output-voltage feedback
- Implement a bang-bang / hysteretic controller
- Define upper and lower switching thresholds
- Observe the resulting variable switching frequency
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
| Nominal output voltage | 5.0 V |
| Output current | 0.1–0.5 A |
| Maximum output power | 2.5 W |
| Control method | Bang-bang / hysteretic |
| Lower switching threshold | 4.95 V |
| Upper switching threshold | 5.05 V |
| Hysteresis window | 100 mV |
| Nominal output ripple target | ≈ 100 mV |
| Topology | Nonsynchronous buck |
| Target efficiency | > 85% |

The controller regulates the output inside the hysteresis window:

$$
4.95\ \text{V} \le V_{OUT} \le 5.05\ \text{V}
$$

Unlike a conventional fixed-frequency PWM controller, **switching frequency is not specified as a control requirement**. It is an outcome of the power stage, load, input voltage, and hysteresis window.

---

## 3. Control Principle

The controller uses two output-voltage thresholds:

$$
V_{LOW} = 4.95\ \text{V}
$$

$$
V_{HIGH} = 5.05\ \text{V}
$$

The basic switching law is:

```text
if VOUT < 4.95 V:
    MOSFET = ON

if VOUT > 5.05 V:
    MOSFET = OFF

otherwise:
    keep previous MOSFET state
```

The difference between the two thresholds creates hysteresis:

$$
V_{HYS} = V_{HIGH} - V_{LOW} = 5.05 - 4.95 = 100\ \text{mV}
$$

The output therefore oscillates around approximately 5 V.

---

## 4. Converter Architecture

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
                 Hysteretic Comparator
                    │             │
                 4.95 V         5.05 V
                    │             │
                    └──────┬──────┘
                           │
                           ▼
                     Gate Driver
                           │
                           ▼
                        MOSFET
```

This is considerably simpler than the voltage-mode PWM architecture:

```text
Voltage-mode PWM:

VOUT → Divider → Error Amp → Compensation → PWM Comparator → Driver


Bang-bang:

VOUT → Hysteretic Comparator → Driver
```

The simplicity of the control loop is one of the main reasons for selecting this control method for the project.

---

## 5. Power Stage

The initial power stage uses the following components:

| Component | Selected value / part |
|---|---|
| Input voltage | 10–14 V |
| Nominal input | 12 V |
| Output voltage | ≈ 5 V |
| Output current | 0.1–0.5 A |
| Inductor | 220 µH |
| Output capacitor | 100 µF |
| Input capacitor | 10 µF |
| Diode | MBRS340 |
| MOSFET | Si4410DY |
| Full-load resistance | 10 Ω |

These values are a starting point inherited from the initial open-loop buck design.

Because the bang-bang controller does not enforce a fixed switching frequency, the final values of **L, C, and hysteresis** will be evaluated together through simulation.

### 5.1 Load Resistance

$$
R_{LOAD} = \frac{V_{OUT}}{I_{OUT}}
$$

| Load condition | Current | Resistance | Output power |
|---|---:|---:|---:|
| Full load | 0.5 A | 10 Ω | 2.5 W |
| Half load | 0.25 A | 20 Ω | 1.25 W |
| Light load | 0.1 A | 50 Ω | 0.5 W |

---

## 6. Bang-Bang Controller

The controller determines the MOSFET state directly from the output voltage.

### MOSFET ON

When $V_{OUT} < V_{LOW}$, the MOSFET turns ON. Inductor current increases and energy is transferred from the input supply to the load and output capacitor.

### MOSFET OFF

When $V_{OUT} > V_{HIGH}$, the MOSFET turns OFF. The inductor current freewheels through the Schottky diode and begins decreasing.

### Hysteresis region

When $V_{LOW} < V_{OUT} < V_{HIGH}$, the comparator retains its previous state. This prevents rapid switching around a single reference voltage.

---

## 7. Expected Waveforms

The project will analyze four main signals.

### Output voltage

```text
5.05 V       /¯¯\      /¯¯\
            /    \    /    \
5.00 V ----/------\--/------\----
          /        \/        \
4.95 V
```

### MOSFET gate

```text
ON     ┌──────┐        ┌──────┐
       │      │        │      │
OFF ───┘      └────────┘      └──
```

### Switching node

```text
VIN     ┌──────┐        ┌──────┐
        │      │        │      │
0 V  ───┘      └────────┘      └──
```

During diode conduction, the switching node falls slightly below ground by approximately the diode forward voltage.

### Inductor current

```text
        /\        /\        /\
       /  \      /  \      /  \
      /    \    /    \    /    \
─────/      \──/      \──/      \──
```

The key behavior to observe is:

**VOUT falls → MOSFET ON → IL rises → VOUT rises → MOSFET OFF → IL falls → VOUT falls → repeat**

---

## 8. Variable-Frequency Operation

A major characteristic of bang-bang control is that the switching frequency is **not fixed**:

$$
f_{SW} = f(V_{IN}, V_{OUT}, I_{OUT}, L, C, V_{HYS})
$$

Changing any of the following can change the switching frequency:

- Input voltage
- Load current
- Inductance
- Output capacitance
- Hysteresis window
- Component ESR
- Propagation delay

One objective of this project is to characterize how the switching frequency changes with operating conditions.

| Test | Expected observation |
|---|---|
| Increase VIN | Switching behavior changes |
| Increase load | Switching frequency/duty behavior changes |
| Increase COUT | Slower VOUT movement |
| Increase hysteresis | Larger ripple, generally lower switching frequency |
| Decrease hysteresis | Smaller ripple, potentially much higher switching frequency |

---

## 9. Development Plan

### Phase 1: Requirements
- [x] Define input-voltage range
- [x] Define nominal output voltage
- [x] Define load-current range
- [x] Define hysteresis window
- [x] Define efficiency target

### Phase 2: Power Stage
- [x] Calculate load resistance
- [x] Select initial inductor
- [x] Select output capacitor
- [x] Select input capacitor
- [x] Select diode
- [x] Select MOSFET

### Phase 3: Open-Loop Simulation
- [ ] Build buck power stage in LTspice
- [ ] Apply fixed-duty-cycle gate signal
- [ ] Verify switching-node voltage
- [ ] Verify inductor current
- [ ] Verify output voltage
- [ ] Verify basic power-stage operation

### Phase 4: Bang-Bang Controller
- [ ] Remove fixed PWM source
- [ ] Implement hysteretic comparator
- [ ] Set lower threshold to 4.95 V
- [ ] Set upper threshold to 5.05 V
- [ ] Connect comparator output to MOSFET driver
- [ ] Verify autonomous switching
- [ ] Verify output regulation

### Phase 5: Operating-Point Analysis
- [ ] Measure average output voltage
- [ ] Measure output ripple
- [ ] Measure switching frequency
- [ ] Measure inductor-current ripple
- [ ] Verify MOSFET and diode stresses

### Phase 6: Load Transients
- [ ] Apply 0.1 A → 0.5 A load step
- [ ] Measure voltage undershoot
- [ ] Measure recovery time
- [ ] Apply 0.5 A → 0.1 A load release
- [ ] Measure voltage overshoot

### Phase 7: Line Transients
- [ ] Apply 10 V → 14 V input step
- [ ] Measure output disturbance
- [ ] Verify recovery
- [ ] Apply 14 V → 10 V step
- [ ] Measure output disturbance

### Phase 8: Hysteresis Study
- [ ] Test 50 mV window
- [ ] Test 100 mV window
- [ ] Test 200 mV window
- [ ] Compare output ripple
- [ ] Compare switching frequency
- [ ] Compare transient response

### Phase 9: Real Components
- [ ] Use MOSFET model
- [ ] Use Schottky diode model
- [ ] Add inductor DCR
- [ ] Add capacitor ESR
- [ ] Add comparator propagation delay
- [ ] Add gate-driver delay
- [ ] Evaluate switching-node ringing

### Phase 10: Efficiency
- [ ] Calculate input power
- [ ] Calculate output power
- [ ] Calculate efficiency
- [ ] Sweep efficiency versus load
- [ ] Identify dominant losses

---

## 10. Verification Plan

| Test | VIN | Load | Verification |
|---|---:|---:|---|
| Nominal | 12 V | 0.5 A | Output regulation |
| Light load | 12 V | 0.1 A | Regulation / switching behavior |
| Minimum VIN | 10 V | 0.5 A | Regulation |
| Maximum VIN | 14 V | 0.5 A | Regulation |
| Startup | 12 V | 0.5 A | Overshoot |
| Load step | 12 V | 0.1 → 0.5 A | Undershoot / recovery |
| Load release | 12 V | 0.5 → 0.1 A | Overshoot / recovery |
| Line step | 10 → 14 V | 0.5 A | Regulation |
| Hysteresis sweep | 12 V | 0.5 A | Ripple vs. frequency |
| Full load | 12 V | 0.5 A | Efficiency |

---

## 11. Key Measurements

For every operating condition, the following parameters will be recorded:

| Measurement | Symbol |
|---|---|
| Average output voltage | $V_{OUT,AVG}$ |
| Output ripple | $\Delta V_{OUT}$ |
| Average switching frequency | $f_{SW,AVG}$ |
| Inductor peak current | $I_{L,PK}$ |
| Inductor minimum current | $I_{L,MIN}$ |
| Input power | $P_{IN}$ |
| Output power | $P_{OUT}$ |
| Efficiency | $\eta$ |

Because switching frequency varies with time, it will be measured over multiple switching cycles rather than from a single period.

---

## 12. Efficiency

Efficiency will be calculated as:

$$
\eta = \frac{P_{OUT}}{P_{IN}} \times 100\%
$$

Initial target: **η > 85%** at:

- $V_{IN} = 12$ V
- $V_{OUT} \approx 5$ V
- $I_{OUT} = 0.5$ A

Expected major losses include:

- Schottky diode conduction
- Inductor DCR
- MOSFET conduction
- MOSFET switching
- Gate-drive losses

---

## 13. Repository Structure

```text
bang-bang-buck-converter/
│
├── README.md
│
├── LTSPICE/
│   ├── 01_open_loop.asc
│   ├── 02_bang_bang_ideal.asc
│   ├── 03_load_transient.asc
│   ├── 04_line_transient.asc
│   └── 05_real_components.asc
│
├── calculations/
│   └── power_stage_design.md
│
├── results/
│   ├── switching_node.png
│   ├── inductor_current.png
│   ├── output_voltage.png
│   ├── load_transient.png
│   ├── hysteresis_comparison.png
│   └── efficiency.png
│
└── docs/
    └── final_report.pdf
```

---

## 14. Tools

- LTspice
- GitHub
- Analytical power-electronics calculations

---

## 15. Project Status

**Current stage: Open-loop power-stage simulation (Phase 3)**

Next steps:

1. Build the 12 V → 5 V buck power stage in LTspice
2. Verify the power stage using a fixed gate signal
3. Replace the fixed PWM source with a hysteretic controller
4. Use initial thresholds of **4.95 V and 5.05 V**
5. Plot $V_{OUT}$, $V_{SW}$, $I_L$, and the MOSFET gate signal
6. Measure the resulting switching frequency
7. Test VIN = **10 V, 12 V, and 14 V**
8. Test loads of **10 Ω, 20 Ω, and 50 Ω**
9. Perform load and line transients
10. Introduce realistic component losses and delays

---

## 16. Lessons Learned

This section will document design decisions, unexpected simulation behavior, debugging steps, and differences between theoretical calculations and LTspice results as the project progresses.

Particular attention will be given to the trade-offs between:

- Hysteresis width
- Output-voltage ripple
- Switching frequency
- Transient response
- Component parasitics
- Comparator and gate-driver delay

The project will also compare the simplicity of bang-bang control with the more structured but more complex approach of conventional fixed-frequency voltage-mode PWM control.
