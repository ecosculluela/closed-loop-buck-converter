# Bang-Bang Controlled Buck Converter

Design, LTspice simulation, and PCB implementation of a closed-loop nonsynchronous buck converter using **bang-bang (hysteretic) voltage control**.

This project was created to put into practice and further develop the power electronics knowledge I have gained throughout my professional career. Starting from first principles, I will design the buck converter power stage, implement a hysteretic feedback controller, verify it in LTspice, and then build and test a PCB to compare real hardware against simulation.

The main objective is to demonstrate how a simple nonlinear controller can regulate the output voltage **without an error amplifier, compensation network, PWM ramp, or fixed-frequency PWM modulator**.

---

## 1. Project Objectives

- Design a DC-DC buck converter from first principles
- Calculate the required inductor and capacitors
- Simulate the power stage in LTspice
- Understand MOSFET, diode, inductor, and capacitor behavior
- Implement a bang-bang / hysteretic controller
- Define upper and lower switching thresholds
- Observe the resulting variable switching frequency
- Evaluate startup, load-transient, and input-voltage-transient behavior
- Analyze component losses and converter efficiency
- Design, assemble, and test a PCB prototype
- Compare calculated, simulated, and measured results

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

| Component | Selected value / part |
|---|---|
| Input voltage | 10–14 V |
| Nominal input | 12 V |
| Output voltage | ≈ 5 V |
| Output current | 0.1–0.5 A |
| Inductor | 220 µH, I<sub>sat</sub> ≥ 1 A |
| Output capacitor | 100 µF, low ESR |
| Input capacitor | 10 µF ceramic, 25 V |
| Diode | MBRS340 Schottky (40 V, 3 A) |
| High-side switch (simulation) | Si4410DY N-channel, floating gate drive |
| High-side switch (PCB) | P-channel MOSFET, ≥ 30 V, V<sub>GS(max)</sub> ≥ ±20 V |
| Full-load resistance | 10 Ω |

These values are a starting point inherited from the initial open-loop buck design. Because the bang-bang controller does not enforce a fixed switching frequency, the final values of **L, C, and hysteresis** will be evaluated together through simulation.

> The PCB uses a P-channel MOSFET because it can be driven from ground with a single NPN transistor. A high-side N-channel MOSFET would need a bootstrap gate driver.

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

### 6.1 Feedback Divider

$$
V_{FB} = V_{OUT} \times \frac{R_{BOT}}{R_{TOP} + R_{BOT}}
$$

With $R_{TOP} = 40.2\ \text{k}\Omega$ and $R_{BOT} = 10\ \text{k}\Omega$, $V_{FB} \approx V_{OUT}/5$.

A 100 mV window at $V_{OUT}$ is therefore a 20 mV window at FB (0.99–1.01 V).

### 6.2 Reference and Hysteresis Network

FB connects to the comparator inverting input (IN−). The non-inverting input (IN+) is driven by a reference divider, with positive feedback from the comparator output.

| Element | Value |
|---|---|
| Reference | 2.5 V shunt reference (LM4040-2.5) |
| Reference divider | R<sub>A</sub> = 2.49 kΩ (top), R<sub>B</sub> = 1.65 kΩ (bottom) → ≈ 0.996 V |
| Thevenin resistance | R<sub>TH</sub> ≈ 1 kΩ |
| Hysteresis resistor | R<sub>H</sub> = 249 kΩ, comparator output → IN+ |

With a 0–5 V push-pull comparator output:

$$
\Delta V_{IN+} = 5\ \text{V} \cdot \frac{R_{TH}}{R_{TH} + R_H} \approx 20\ \text{mV}
$$

This places the thresholds at about **4.95 V and 5.05 V** at $V_{OUT}$.

> With an open-collector comparator, the pull-up resistor adds to R<sub>H</sub> and the network must be recalculated.

### 6.3 Gate Driver (PCB)

- Comparator output high → NPN (MMBT3904) pulls the PMOS gate low → PMOS ON
- ~1 kΩ gate pull-up to VIN turns the PMOS OFF
- Optional PNP (MMBT3906) emitter follower for faster turn-off
- If the selected PMOS is rated only ±12 V V<sub>GS</sub>, add a 12 V Zener from gate to source

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

> In practice, the ripple is expected to be larger than the 100 mV window. After the switch turns off, the inductor keeps charging the capacitor, and comparator and driver delays add further overshoot.

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

| Test | Expected observation |
|---|---|
| Increase VIN | Switching behavior changes |
| Increase load | Switching frequency/duty behavior changes |
| Increase COUT | Slower VOUT movement |
| Increase hysteresis | Larger ripple, generally lower switching frequency |
| Decrease hysteresis | Smaller ripple, potentially much higher switching frequency |

---

## 9. Hardware Prototype (PCB)

A 2-layer PCB will be designed in KiCad to validate the LTspice results on real hardware.

### 9.1 Hardware Architecture

```text
VIN (10–14 V) ──┬── LDO (5 V) ── powers comparator + reference
                │
                ├── P-MOSFET (high side) ── SW ── 220 µH ──┬── VOUT ── Load
                │        ▲                  │              │
                │     Gate driver        Schottky        100 µF
                │     (NPN level shift)     │              │
                │        ▲                 GND            GND
                │        │
                │   Comparator ◄── FB (VOUT ÷ 5)
                │        ▲
                │   Reference + hysteresis network
```

| Block | Implementation |
|---|---|
| High-side switch | P-channel MOSFET (≥ 30 V, no bootstrap needed) |
| Gate driver | NPN level shifter with gate pull-up to VIN |
| Controller | Fast push-pull comparator with resistive hysteresis |
| Reference | 2.5 V shunt reference, divided to ~1.0 V |
| Feedback | 40.2 kΩ / 10 kΩ divider (VOUT ÷ 5) |
| Controller supply | 5 V LDO from VIN |
| Rectifier | 40 V, 3 A Schottky |
| Soft-start | RC ramp on the reference |

### 9.2 Startup Protection

With only voltage feedback, the MOSFET stays fully on at startup until $V_{OUT}$ reaches the upper threshold. With 220 µH and 100 µF, the LC charging current can reach several amps, which risks inductor saturation and large overshoot.

Mitigations:

- **Soft-start:** RC on the reference for a ~5 ms ramp, so $V_{OUT}$ follows a slowly rising reference
- **Current sense:** 0.1 Ω sense resistor with a test point, to measure inductor current and allow a future current limit
- **Bench supply:** first power-up with a current-limited supply (~0.3 A)

### 9.3 Test Features

- Test points: VIN, VOUT, SW, GATE, FB, comparator output, GND
- Ground springs or loops for scope probes
- Current-sense resistor / wire loop for inductor current measurement
- Selectable hysteresis resistor (50 / 100 / 200 mV windows)
- Screw terminals for VIN and external load
- Onboard 10 Ω, 3 W load with jumper
- LED indicator on VOUT

### 9.4 Layout Guidelines

| Area | Guideline |
|---|---|
| Hot loop | C<sub>IN</sub>, MOSFET, and Schottky as close as possible, short wide traces |
| SW node | Small copper area, kept away from FB |
| Ground | Solid bottom ground plane; power and signal ground joined near C<sub>OUT</sub> |
| FB trace | Short, Kelvin-sensed from the C<sub>OUT</sub> pad, divider next to the comparator |
| Comparator | 100 nF decoupling directly at the supply pin |
| Board | 2-layer, 1 oz copper, ~50 × 50 mm, 0805 or larger passives |

### 9.5 Bring-Up Procedure

1. **Controller only:** MOSFET not populated; check the 5 V rail, reference, and comparator thresholds by sweeping a voltage into FB
2. **First power-up:** full board at 10 V, 0.3 A current limit, 50 Ω load
3. **Check waveforms:** GATE, SW, and VOUT on the oscilloscope
4. **Increase stress:** step to 12 V and 14 V, then to the 10 Ω load
5. **Run tests:** load step, line step, hysteresis sweep, and efficiency

---

## 10. Development Plan

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
- [ ] Replace ideal parts with the exact PCB MOSFET, comparator, and driver models
- [ ] Use Schottky diode model
- [ ] Add inductor DCR
- [ ] Add capacitor ESR
- [ ] Add comparator propagation delay
- [ ] Add gate-driver delay
- [ ] Add soft-start and verify startup current
- [ ] Evaluate switching-node ringing

### Phase 10: Efficiency (Simulation)
- [ ] Calculate input power
- [ ] Calculate output power
- [ ] Calculate efficiency
- [ ] Sweep efficiency versus load
- [ ] Identify dominant losses

### Phase 11: PCB Design
- [ ] Draw the schematic in KiCad
- [ ] Lay out the PCB
- [ ] Review layout against Section 9.4
- [ ] Generate Gerbers and BOM
- [ ] Order the PCB and components

### Phase 12: Hardware Bring-Up and Testing
- [ ] Assemble the board
- [ ] Verify the controller with no power stage
- [ ] First power-up with a current-limited supply
- [ ] Measure switching waveforms
- [ ] Run load, line, and hysteresis tests
- [ ] Measure efficiency
- [ ] Compare hardware results with simulation

---

## 11. Verification Plan

Each test will be performed in simulation and on the PCB.

| Test | VIN | Load | Verification |
|---|---:|---:|---|
| Nominal | 12 V | 0.5 A | Output regulation |
| Light load | 12 V | 0.1 A | Regulation / switching behavior |
| Minimum VIN | 10 V | 0.5 A | Regulation |
| Maximum VIN | 14 V | 0.5 A | Regulation |
| Startup | 12 V | 0.5 A | Overshoot and inrush current |
| Load step | 12 V | 0.1 → 0.5 A | Undershoot / recovery |
| Load release | 12 V | 0.5 → 0.1 A | Overshoot / recovery |
| Line step | 10 → 14 V | 0.5 A | Regulation |
| Hysteresis sweep | 12 V | 0.5 A | Ripple vs. frequency |
| Full load | 12 V | 0.5 A | Efficiency |

---

## 12. Key Measurements

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

## 13. Efficiency

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
- Gate-drive and controller losses

---

## 14. Repository Structure

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
│   ├── 05_real_components.asc
│   └── 06_pcb_circuit.asc
│
├── calculations/
│   ├── power_stage_design.md
│   └── hysteresis_network.md
│
├── hardware/
│   ├── kicad/
│   ├── gerbers/
│   ├── bom.csv
│   └── photos/
│
├── results/
│   ├── simulation/
│   │   ├── switching_node.png
│   │   ├── inductor_current.png
│   │   ├── output_voltage.png
│   │   ├── load_transient.png
│   │   ├── hysteresis_comparison.png
│   │   └── efficiency.png
│   └── hardware/
│       ├── scope_captures/
│       └── efficiency.png
│
└── docs/
    └── final_report.pdf
```

---

## 15. Tools

- LTspice
- KiCad
- GitHub
- Oscilloscope, current-limited bench supply, electronic load
- Analytical power-electronics calculations

---

## 16. Project Status

**Current stage: Open-loop power-stage simulation (Phase 3)**

Next steps:

1. Build the 12 V → 5 V buck power stage in LTspice
2. Verify the power stage using a fixed gate signal
3. Replace the fixed PWM source with a hysteretic controller
4. Use initial thresholds of **4.95 V and 5.05 V**
5. Plot $V_{OUT}$, $V_{SW}$, $I_L$, and the MOSFET gate signal
6. Measure the resulting switching frequency
7. Test VIN = **10 V, 12 V, and 14 V** and loads of **10 Ω, 20 Ω, and 50 Ω**
8. Perform load and line transients
9. Simulate the exact PCB circuit before ordering the board
10. Design, build, and test the PCB

---

## 17. Lessons Learned

This section will document design decisions, unexpected simulation and hardware behavior, debugging steps, and differences between calculations, LTspice results, and bench measurements as the project progresses.

Particular attention will be given to the trade-offs between:

- Hysteresis width
- Output-voltage ripple
- Switching frequency
- Transient response
- Component parasitics and PCB layout
- Comparator and gate-driver delay

The project will also compare the simplicity of bang-bang control with the more structured but more complex approach of conventional fixed-frequency voltage-mode PWM control.
