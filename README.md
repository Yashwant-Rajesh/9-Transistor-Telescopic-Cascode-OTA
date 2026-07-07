# 9-Transistor Telescopic Cascode OTA

A minimal telescopic cascode operational transconductance amplifier (OTA), built to explore how cascoding improves gain and to see the effect of using ideal, ground-referenced bias sources instead of a proper on-chip bias network.

This is intentionally a small, focused project — biasing is done with **ideal DC voltage/current sources** rather than a designed bias generator, which turns out to directly explain some of the measured non-idealities below (see PSRR).

## Topology
Classic single-ended telescopic cascode, 9 transistors total, all in a TSMC 018 (0.18µm) process:

| Device | Role | Size |
|---|---|---|
| `M1`, `M2` | NMOS input differential pair | l=2µ, w=6.3µ |
| `M3`, `M4` | NMOS cascode devices | l=2µ, w=2.2µ |
| `M5`, `M6` | PMOS cascode devices | l=2µ, w=9.8µ |
| `M7`, `M8` | PMOS current-mirror load | l=2µ, w=6.3µ |
| `M9` | NMOS tail current source | l=2µ, w=2.8µ |

Biasing: four ideal DC voltage sources (`V1`–`V4`: 1.859V, 1.509V, 0.639V, 1.385V) set the cascode gate biases, `V6` = 0.9V sets the input common-mode, `V7` = 2.5V is the supply, and `I1` = 5µA sets the tail reference. Load capacitor `C1` = 2p.

## Tests Conducted
| Test | Command | Result |
|---|---|---|
| Operating point | `.op` | Baseline bias check |
| AC response | `.ac dec 100 1 1G` | **DC Gain = 58.39dB**, **Phase Margin = 89.1°**, **GBW = 2.9MHz** |
| CMRR | (derived from AC sweep) | **36.28dB** — low, because the tail is a single transistor (`M9`), not a cascoded current source. CMRR here is fundamentally set by 2·g_m1·r_o9; a single MOSFET tail gives modest r_o and thus modest CMRR. Cascoding the tail (a 10th transistor) would improve this. |
| PSRR | (derived from AC sweep) | **−0.91dB** — poor, because the PMOS bias voltages are generated from ideal, ground-referenced sources rather than a Vdd-tracking bias network. V_sg of the PMOS current sources isn't held constant against Vdd variation, so supply noise couples straight to the output. A production bias network (Vdd-referenced or cascode-mirror biasing) would suppress this. |
| Transient step response | `V5` = PULSE(0 0.2 1u 10n 10n 8u 20u), `.tran 0 8u 0 1n` | Step response / settling behavior at the input |

## Repository Structure
```
9-Transistor-Telescopic-Cascode-OTA/
├── README.md
│
├── Schematic/
│   └── 9-Transistor Telescopic Cascode OTA.asc
│
└── Tests/
    ├── operating_point/         # screenshots
    ├── ac_response_gain_pm_gbw/
    ├── cmrr/
    ├── psrr/
    └── transient_step_response/
```
> Note: This design uses `tsmc018.lib` (TSMC 0.18µm PDK), included via `.include tsmc018.lib` in all three schematics. Since this is a licensed foundry PDK, not committing the library file itself to the public repo.
