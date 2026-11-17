# Validation plan

**Status: no bench data yet.** This page lists each test, what it confirms, and its pass criterion. The "Result" column will be filled in as the board is brought up, and raw data will go in a `test-data/` folder.

Tests are ordered so that each one makes the next one safe to run.

## 1. Pack characterization (no board needed)

| # | Test | Method | Pass criterion | Calculated | Result |
|---|---|---|---|---|---|
| 1.1 | Usable energy | Discharge one pack at 22–25 A constant current to 15.0 V under load, log voltage and current | ≥ 63 Wh | 70 Wh assumed | pending |
| 1.2 | Pack resistance | Step the load, ΔV / ΔI | Informational | 70 mΩ assumed | pending |
| 1.3 | Rest voltage after cutoff | Measure open-circuit voltage 1 and 20 min after test 1.1 | < 17.09 V | ≈ 16.7 V | pending |
| 1.4 | Pack temperature | Thermocouple on the pack during test 1.1 | Within the cell's discharge limit | ≤ 47 °C rise, no-cooling bound | pending |

## 2. Power-up, unloaded

| # | Test | Method | Pass criterion | Calculated | Result |
|---|---|---|---|---|---|
| 2.1 | Power path only | Bench supply on input 1, current limited, buck disabled | VBUS ramps at about 5 V/ms after ~256 ms; status LED 1 lit | 9.4 A for 4.2 ms into the bulk capacitors | pending |
| 2.2 | UV and OV thresholds | Sweep the bench supply on each input | UV 14.35–15.76 V, OV 21.59–23.54 V, re-enable 17.09–19.96 V | 15.04 / 22.55 / 18.51 V nominal | pending |
| 2.3 | Buck enable | Sweep VBUS up and down | On at 14.65–16.81 V, off at 6.23–11.34 V | 15.71 V / 8.75 V nominal | pending |
| 2.4 | Output voltage, no load | DMM at the output | 11.4–12.6 V | 11.82–12.18 V | pending |

## 3. Buck converter under load

| # | Test | Method | Pass criterion | Calculated | Result |
|---|---|---|---|---|---|
| 3.1 | Switch-node waveform | Scope with a short ground spring at both switch nodes, full load, 23.5 V in | Peak below about 36 V (40 V FETs) | — | pending |
| 3.2 | Efficiency | Input and output power at 25, 50, 75 and 100 % load, at 13.5, 18 and 23.5 V in | ≥ 95 % at 350 W | ≈ 96.4 % worst case | pending |
| 3.3 | Load regulation and ripple | Electronic load 0–29.2 A | 11.4–12.6 V | ripple ≤ 32 mV | pending |
| 3.4 | Load step | 18 A step at 13.5 V in | Undershoot ≤ 0.42 V, no ringing | 0.35 V | pending |
| 3.5 | Current limit | Raise load past 29.2 A at 13.5 V in | No trip at full load; limit at 21.3–26.7 A per phase | 9 % margin | pending |
| 3.6 | Phase current sharing | Voltage across each shunt | Within 10 % | — | pending |

## 4. Thermal

| # | Test | Method | Pass criterion | Calculated | Result |
|---|---|---|---|---|---|
| 4.1 | 30 minute soak | Full load for 30 min at 13.5 V and again at 23.5 V in; thermocouples on buck high-side FETs, inductors, shunts, switch-node copper | FET case below 125 °C, inductor below 125 °C | FET junction ≈ 126 °C plus inductor heating; inductor ≈ 80 °C | pending |
| 4.2 | Power-path FETs | Thermocouple on a drain tab at 25 A through one channel | Junction ≤ 125 °C | 109–115 °C | pending |
| 4.3 | FET thermal resistance | From 4.2: (T_tab − T_ambient) / (V_DS × I) | ≤ 40 °C/W | 40 °C/W assumed | pending |
| 4.4 | Thermal image | IR camera over the whole board at the end of 4.1 | No unexpected hot spot; switch-node neck checked | — | pending |

## 5. Pack switchover

| # | Test | Method | Pass criterion | Calculated | Result |
|---|---|---|---|---|---|
| 5.1 | Normal switchover | Drain pack 1 at full load with pack 2 present; scope VBUS, the 12 V output and the input pin | Bus dip ≤ 0.5 V; output stays in tolerance; no false UV trip on pack 2 | 0.26 V nominal, 0.33 V worst | pending |
| 5.2 | Input pin peak | Same event, scope the input pin of the pack being disconnected | ≤ 42 V | ≈ 32 V | pending |
| 5.3 | No bounce | Leave the drained pack connected for 20 min | Its channel stays off | 0.39 V margin | pending |
| 5.4 | Hot insertion | Insert a charged pack into a higher-priority slot | Takes over after ~256 ms, no output disturbance | — | pending |
| 5.5 | Active pack removal (abnormal) | Pull the active pack at full load | Buck stays on; next pack takes over | Bus ≥ 12.8 V | pending |
| 5.6 | Reverse and over-voltage | Reversed bench supply; supply above the OV threshold | Channel stays off, no damage | — | pending |

## 6. System

| # | Test | Method | Pass criterion | Calculated | Result |
|---|---|---|---|---|---|
| 6.1 | Full runtime | Three charged packs, 350 W load until the last pack cuts off | ≥ 30 min | ≈ 33 min | pending |

## Equipment

Programmable DC supply (30 V, 30 A), electronic load (≥ 400 W), four-channel oscilloscope with short-ground probe tips, current probe, thermocouple logger, IR camera, DMM.
