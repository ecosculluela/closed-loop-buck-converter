# Closed-Loop Buck Converter

Design and simulation of a voltage-mode controlled
DC-DC buck converter using LTspice.

## Specifications

| Parameter | Value |
|---|---|
| Input voltage | 12 V |
| Output voltage | 5 V |
| Output current | 0.5 A |
| Switching frequency | 100 kHz |
| Control | Voltage-mode |
| Topology | Nonsynchronous buck |

## Design Process

1. Define requirements
2. Calculate duty cycle
3. Select L and C
4. Build open-loop converter
5. Add voltage feedback
6. Add PI controller
7. Implement PWM
8. Close feedback loop
9. Perform load-step test
10. Analyze efficiency
