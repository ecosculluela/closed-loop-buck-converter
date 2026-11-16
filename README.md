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
| Output ripple target | < 50 mV<sub>pp</sub> |
| Inductor ripple target | ~30% of full-load current |
| Topology | Nonsynchronous buck |
| Control method | Voltage-mode PWM |
| Feedback reference | 1.0 V |

The regulated output-voltage target is therefore:

$$
4.90\ \text{V} \le V_{OUT} \le 5.10\ \text{V}
$$

---

## 3. Transient Requirements

The converter will also be evaluated under changing operating conditions.

### Load transient

0.1 A → 0.5 A → 0.1 A

Initial target:

- Peak output deviation: < 10%
- Recovery into ±2% regulation band: < 2 ms

### Input-voltage transient

10 V → 14 V → 10 V

The feedback controller should adjust the PWM duty cycle to return V<sub>OUT</sub> to 5 V.

### Startup

Initial condition: V<sub>OUT</sub>(0) = 0 V

Targets:

- Settling time: < 5 ms
- Maximum startup overshoot: < 5.5 V

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

## 5. Power Stage Design

### 5.1 Design Assumptions

| Parameter | Symbol | Value |
|---|---|---:|
| Input voltage range | $V_{IN}$ | 10–14 V |
| Output voltage | $V_{OUT}$ | 5.0 V |
| Output current range | $I_{OUT}$ | 0.1–0.5 A |
| Switching frequency | $f_{SW}$ | 100 kHz |
| Switching period | $T_{SW}$ | 10 µs |
| Schottky diode forward drop (assumed) | $V_D$ | 0.5 V |
| Inductor ripple target | $\Delta I_L$ | 0.15 A<sub>pp</sub> (30% of 0.5 A) |
| Output ripple target | $\Delta V_{OUT}$ | < 50 mV<sub>pp</sub> |

The MOSFET conduction drop is neglected in the duty-cycle calculation. At 0.5 A with an on-resistance of about 15 mΩ, it is roughly 7 mV.

### 5.2 Duty Cycle

Ideal buck converter:

$$
D_{ideal} = \frac{V_{OUT}}{V_{IN}}
$$

Including the diode forward drop. During the off-time, the switching node is clamped to $-V_D$, and volt-second balance on the inductor gives:

$$
D = \frac{V_{OUT} + V_D}{V_{IN} + V_D}
$$

| $V_{IN}$ | $D_{ideal}$ | $D$ with diode drop |
|---:|---:|---:|
| 10 V | 50.0% | 52.4% |
| 12 V | 41.7% | 44.0% |
| 14 V | 35.7% | 37.9% |

The controller must cover a duty-cycle range of about **38–52%**, plus margin for transients.

### 5.3 Load Resistance

$$
R_{LOAD} = \frac{V_{OUT}}{I_{OUT}}
$$

| Load condition | $I_{OUT}$ | $R_{LOAD}$ | $P_{OUT}$ |
|---|---:|---:|---:|
| Full load | 0.5 A | 10 Ω | 2.5 W |
| Half load | 0.25 A | 20 Ω | 1.25 W |
| Light load | 0.1 A | 50 Ω | 0.5 W |

### 5.4 Inductor Ripple Requirement

$$
\Delta I_L = 0.30 \times I_{OUT,max} = 0.30 \times 0.5\ \text{A} = 0.15\ \text{A}_{pp}
$$

Ripple is largest at the **maximum input voltage**, so the inductor is sized at $V_{IN} = 14$ V.

### 5.5 Inductor Selection

During the off-time, the voltage across the inductor is $-(V_{OUT} + V_D)$. Therefore:

$$
\Delta I_L = \frac{(V_{OUT} + V_D)(1 - D)}{L \, f_{SW}}
$$

Solving for $L$:

$$
L = \frac{(V_{OUT} + V_D)(1 - D)}{\Delta I_L \, f_{SW}}
$$

At $V_{IN} = 14$ V and $D = 0.379$:

$$
L = \frac{5.5 \times 0.621}{0.15 \times 100\,000} = 228\ \mu\text{H}
$$

With the ideal equation, which ignores $V_D$, the result is $L = 214\ \mu\text{H}$.

**Selected value: $L = 220\ \mu\text{H}$** (nearest standard value).

#### Resulting ripple with 220 µH

| $V_{IN}$ | $D$ | $\Delta I_L$ | Ripple ratio | $I_{L,min}$ (0.5 A) | $I_{L,peak}$ (0.5 A) |
|---:|---:|---:|---:|---:|---:|
| 10 V | 52.4% | 119 mA<sub>pp</sub> | 24% | 0.441 A | 0.560 A |
| 12 V | 44.0% | 140 mA<sub>pp</sub> | 28% | 0.430 A | 0.570 A |
| 14 V | 37.9% | 155 mA<sub>pp</sub> | 31% | 0.422 A | 0.578 A |

At 14 V, the ripple is slightly above the 30% target. This is acceptable for a "~30%" requirement.

#### Continuous-conduction check

The converter stays in continuous conduction mode (CCM) when:

$$
I_{OUT,min} > \frac{\Delta I_L}{2}
$$

$$
0.1\ \text{A} > \frac{0.155\ \text{A}}{2} = 0.078\ \text{A} \quad \checkmark
$$

The critical inductance at the worst case ($V_{IN} = 14$ V, $I_{OUT} = 0.1$ A) is:

$$
L_{crit} = \frac{(V_{OUT}+V_D)(1-D)}{2\, I_{OUT,min}\, f_{SW}} = 171\ \mu\text{H}
$$

Because $220\ \mu\text{H} > 171\ \mu\text{H}$, the converter operates in **CCM across the full input and load range**.

#### Inductor current ratings

$$
I_{L,peak} = I_{OUT,max} + \frac{\Delta I_L}{2} = 0.5 + 0.078 = 0.578\ \text{A}
$$

$$
I_{L,RMS} = \sqrt{I_{OUT}^2 + \frac{\Delta I_L^2}{12}} \approx 0.502\ \text{A}
$$

| Inductor parameter | Requirement |
|---|---:|
| Inductance | 220 µH |
| Saturation current | ≥ 1 A (margin for startup and transients) |
| RMS current rating | ≥ 0.75 A |
| DCR | ≤ 0.3 Ω |

DCR loss at full load:

$$
P_{DCR} = I_{L,RMS}^2 \times DCR = 0.252 \times 0.3 = 76\ \text{mW}
$$

### 5.6 Output Capacitor Selection

#### Ripple requirement

The ripple caused by capacitance alone is:

$$
\Delta V_C = \frac{\Delta I_L}{8\, f_{SW}\, C_{OUT}}
$$

$$
C_{OUT,min} = \frac{0.155}{8 \times 100\,000 \times 0.05} = 3.9\ \mu\text{F}
$$

The maximum allowable ESR is:

$$
ESR_{max} = \frac{\Delta V_{OUT}}{\Delta I_L} = \frac{50\ \text{mV}}{0.155\ \text{A}} = 0.32\ \Omega
$$

#### Transient requirement

The ripple requirement alone gives a very small capacitor. Load-transient performance drives the real choice.

**Load release (0.5 A → 0.1 A).** The energy stored in the inductor is transferred to the capacitor:

$$
\Delta V_{overshoot} \approx \sqrt{V_{OUT}^2 + \frac{L\,(I_1^2 - I_2^2)}{C_{OUT}}} - V_{OUT}
$$

With $C_{OUT} = 100\ \mu\text{F}$:

$$
\Delta V_{overshoot} \approx \sqrt{25 + \frac{220\mu \times 0.24}{100\mu}} - 5 = 53\ \text{mV} \quad (1.1\%)
$$

**Loop-bandwidth estimate.** For a loop crossover near $f_c \approx f_{SW}/10 = 10$ kHz and a 0.4 A load step, keeping the deviation within 2% (100 mV) requires:

$$
C_{OUT} \gtrsim \frac{\Delta I_{OUT}}{2\pi f_c \, \Delta V} = \frac{0.4}{2\pi \times 10^4 \times 0.1} = 64\ \mu\text{F}
$$

**Selected value: $C_{OUT} = 100\ \mu\text{F}$, 16 V, low-ESR (≈ 30 mΩ)**

#### Expected output ripple

$$
\Delta V_C = \frac{0.155}{8 \times 10^5 \times 100\mu} = 1.9\ \text{mV}
$$

$$
\Delta V_{ESR} = 0.155 \times 0.03 = 4.7\ \text{mV}
$$

$$
\Delta V_{OUT} \lesssim 6.6\ \text{mV}_{pp} \ll 50\ \text{mV}_{pp} \quad \checkmark
$$

#### Capacitor RMS current

$$
I_{C,RMS} = \frac{\Delta I_L}{\sqrt{12}} = 45\ \text{mA}
$$

| Output capacitor parameter | Requirement |
|---|---:|
| Capacitance | 100 µF |
| Voltage rating | 16 V (≥ 2× $V_{OUT}$) |
| ESR | ≤ 50 mΩ |
| Ripple-current rating | ≥ 100 mA |

> Note: if MLCCs are used, the effective capacitance drops under DC bias, so a higher voltage rating or more capacitance is needed.

### 5.7 Input Capacitor Selection

The input capacitor supplies the pulsed MOSFET current. Its RMS current is highest near $D = 0.5$:

$$
I_{CIN,RMS} = I_{OUT}\sqrt{D(1-D)} = 0.5\sqrt{0.524 \times 0.476} = 0.25\ \text{A}
$$

Input voltage ripple:

$$
\Delta V_{IN} \approx \frac{I_{OUT}\, D(1-D)}{f_{SW}\, C_{IN}}
$$

With $C_{IN} = 10\ \mu\text{F}$:

$$
\Delta V_{IN} \approx \frac{0.5 \times 0.249}{10^5 \times 10\mu} = 125\ \text{mV}_{pp} \ (\approx 1\%)
$$

**Selected value: $C_{IN} = 10\ \mu\text{F}$, 25 V ceramic**, placed close to the MOSFET and diode.

### 5.8 Diode Selection

A Schottky diode is used because of its low forward voltage and negligible reverse-recovery charge.

#### Voltage stress

$$
V_{R,max} = V_{IN,max} = 14\ \text{V}
$$

With margin for switching-node ringing, the diode should be rated for **≥ 30 V**.

#### Current stress

The diode conducts during the off-time:

$$
I_{D,avg} = I_{OUT}(1 - D)
$$

| $V_{IN}$ | $I_{D,avg}$ | $I_{D,peak}$ | Conduction loss $V_D \cdot I_{D,avg}$ |
|---:|---:|---:|---:|
| 10 V | 0.24 A | 0.56 A | 119 mW |
| 12 V | 0.28 A | 0.57 A | 140 mW |
| 14 V | 0.31 A | 0.58 A | 155 mW |

**Selected diode: MBRS340** (40 V, 3 A Schottky, available in the LTspice default library)

| Parameter | Requirement | MBRS340 |
|---|---:|---:|
| Reverse voltage | ≥ 30 V | 40 V |
| Average forward current | ≥ 1 A | 3 A |

> The diode is the dominant loss in this converter, at about 5.6% of output power at 12 V. A synchronous rectifier would remove most of this loss and is a possible future improvement.

### 5.9 MOSFET Selection

#### Voltage stress

$$
V_{DS,max} = V_{IN,max} + V_D \approx 14.5\ \text{V}
$$

With margin for ringing, the MOSFET should be rated for **≥ 30 V**.

#### Current stress

$$
I_{Q,RMS} \approx I_{OUT}\sqrt{D}
$$

Worst case is $V_{IN} = 10$ V:

$$
I_{Q,RMS} = 0.5\sqrt{0.524} = 0.36\ \text{A}
$$

**Selected MOSFET: Si4410DY** (30 V N-channel, available in the LTspice default library)

#### Loss estimate at 12 V, 0.5 A

Conduction loss, assuming $R_{DS(on)} \approx 15\ \text{m}\Omega$ at $V_{GS} = 10$ V:

$$
P_{cond} = I_{Q,RMS}^2 \, R_{DS(on)} = 0.11 \times 0.015 \approx 2\ \text{mW}
$$

Switching loss, assuming $t_r + t_f \approx 40$ ns:

$$
P_{sw} \approx \frac{1}{2} V_{IN}\, I_{OUT}\,(t_r + t_f)\, f_{SW} = 0.5 \times 12 \times 0.5 \times 40\text{n} \times 100\text{k} = 12\ \text{mW}
$$

Gate-drive power is supplied by the driver:

$$
P_{gate} = Q_g \, V_{GS} \, f_{SW}
$$

> The assumed $R_{DS(on)}$, rise/fall times, and $Q_g$ must be checked against the datasheet and verified in simulation in Phase 8.

#### Gate-drive consideration

This is a **high-side N-channel MOSFET**. Its source is connected to the switching node, so the gate must be driven about 10 V **above** the switching node, which means up to about 24 V above ground.

- **Simulation:** drive the gate with a pulse source connected between the gate and the switching node (floating drive).
- **Real hardware:** a bootstrap gate driver is required. A P-channel MOSFET is an alternative that avoids the bootstrap circuit, at the cost of higher $R_{DS(on)}$.

### 5.10 Estimated Loss Budget (12 V, 0.5 A)

| Loss contributor | Estimate |
|---|---:|
| Diode conduction | 140 mW |
| Inductor DCR | 76 mW |
| MOSFET switching | 12 mW |
| MOSFET conduction | 2 mW |
| Output capacitor ESR | < 1 mW |
| **Total** | **≈ 230 mW** |

$$
\eta \approx \frac{P_{OUT}}{P_{OUT} + P_{loss}} = \frac{2.5}{2.73} \approx 91.6\%
$$

This estimate excludes controller and gate-drive power. It meets the **η > 85%** target with margin.

### 5.11 Power Stage Summary

| Component | Selected value / part | Key rating |
|---|---|---|
| Inductor | 220 µH | I<sub>sat</sub> ≥ 1 A, DCR ≤ 0.3 Ω |
| Output capacitor | 100 µF | 16 V, ESR ≈ 30 mΩ |
| Input capacitor | 10 µF ceramic | 25 V |
| Diode | MBRS340 Schottky | 40 V, 3 A |
| MOSFET | Si4410DY N-channel | 30 V |
| Full-load resistance | 10 Ω | 2.5 W |
| Nominal duty cycle | 44% | at 12 V |

#### Calculated values to verify in Phase 3

| Quantity (12 V, 0.5 A) | Calculated |
|---|---:|
| Duty cycle | 44.0% |
| Inductor ripple | 140 mA<sub>pp</sub> |
| $I_{L,min}$ / $I_{L,peak}$ | 0.430 A / 0.570 A |
| Output ripple | < 7 mV<sub>pp</sub> |
| Switching-node low level | ≈ −0.5 V |

---

## 6. Feedback Network

The controller uses a 1.0 V reference. The feedback divider must therefore generate $V_{FB} = 1.0$ V when $V_{OUT} = 5.0$ V.

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

$$
V_{FB} = V_{OUT} \times \frac{R_{BOTTOM}}{R_{TOP} + R_{BOTTOM}}
$$

Therefore:

$$
V_{FB} = 5 \times \frac{10\text{k}}{40\text{k} + 10\text{k}} = 1.0\ \text{V}
$$

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
- [x] Calculate inductor value
- [x] Verify continuous conduction mode
- [x] Calculate output capacitor
- [x] Calculate input capacitor
- [x] Select diode
- [x] Select MOSFET
- [x] Estimate loss budget

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

After nonideal component models are introduced, the initial full-load efficiency target is **η > 85%** at:

- V<sub>IN</sub> = 12 V
- V<sub>OUT</sub> = 5 V
- I<sub>OUT</sub> = 0.5 A

The Phase 2 loss budget estimates **≈ 91.6%** (see Section 5.10).

Efficiency will be calculated using:

$$
\eta = \frac{P_{OUT}}{P_{IN}} \times 100\%
$$

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

**Current stage: Open-loop simulation (Phase 3)**

Next steps:

1. Build the open-loop LTspice model with an ideal switch and the MBRS340 diode (L = 220 µH, C<sub>OUT</sub> = 100 µF, R<sub>LOAD</sub> = 10 Ω, D = 0.44)
2. Verify the switching-node voltage, inductor current, and output voltage
3. Compare simulated ripple and inductor-current limits with the Section 5.11 values
4. Sweep input voltage (10/12/14 V) and load (10/20/50 Ω)

---

## 14. Lessons Learned

This section will document design decisions, unexpected simulation behavior, debugging steps, and differences between theoretical calculations and simulation results as the project progresses.
