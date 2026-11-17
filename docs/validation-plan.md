# Bench test plan

**Status: no bench data yet.** Every "Result" cell is pending. Raw data will go in `test-data/` as each step is completed.

Contents: [Where power is applied](#where-power-is-applied) · [Test access](#test-access) · [Efficiency measurement](#efficiency-measurement) · [Equipment](#equipment) · [Setup](#setup) · [Test sequence](#test-sequence) · [Data](#data) · [Not planned](#not-planned) · [Assumptions](#assumptions)

## Where power is applied

The buck's input corners (13.5 V and 23.5 V) cannot be reached in steady state through the battery inputs:

- The LTC4417 opens a channel at 14.35–15.76 V. A 13.5 V bus only exists for about 1 ms during a pack change.
- It rejects an input above 21.59–23.54 V, so a 23.5 V supply on a battery input is rejected on most boards.
- A channel only connects after its input has been above the re-enable level, 17.09–19.96 V.

Buck corner tests are therefore fed directly into VBUS, bypassing the power path. Tests through the battery inputs run at 16–21 V. Injecting at VBUS with no packs fitted is safe: the battery-side FETs block, and the input pins see no voltage.

| Injection at VBUS | Setting |
|---|---|
| 23.5 V | Supply over-voltage protection at 24.0 V. The VBUS TVS has a 24.4 V minimum breakdown |
| 13.5 V | Start at 18 V, then lower the supply. The buck turns on at 14.65–16.81 V and stays on down to 6.23–11.34 V |

## Test access

| Access point | Location | Used for |
|---|---|---|
| TP_VBUS with ground pad | VBUS pour, between the bulk capacitors and the buck input capacitors | Kelvin input voltage for buck efficiency; switchover capture |
| TP_12V with ground pad | 12 V pour at the output | Kelvin output voltage for efficiency, regulation and load step |
| TP_VBAT1–3, each with a ground pad within 5 mm | At the LTC4417 input pins and their bypass capacitors, not at the fuses | Input pin peak during a pack change (42 V absolute maximum); power-path voltage drop |
| TP_EN, TP_VCC, TP_AGND, TP_GND | Enable divider and controller supply, away from the switch nodes | Buck enable state; bench disable by shorting TP_EN to TP_AGND; controller supply check |
| VBUS and ground wire pads, sized for a 10 AWG lead | At the bulk capacitors | VBUS injection (about 27 A) |
| Bare ground pad within 5 mm of each low-side FET tab | Ground pour beside each switch node | Spring-ground scope probing of the switch nodes |
| Mounting holes | Corners | Board on standoffs during test |

Nodes that get no test point, because a probe or added copper would disturb them: the UV and OV divider taps (a 10 MΩ probe shifts the UV threshold about 7 %), the current-sense pairs (60 mV full scale), compensation, feedback, timing and soft-start pins, the switch nodes, gate drives and bootstrap pins. These are probed on existing pads: switch node on the low-side FET tab, gates on the FET gate lead, shunt voltage on the shunt terminals.

## Efficiency measurement

The calculated efficiency is 96.4 % against a 95 % limit, a 1.4 % margin. Voltage drop in the bench leads is larger than that margin, so input and output voltage are read on the board copper at TP_VBUS and TP_12V.

```math
\eta = \frac{V_{out}\,I_{out}}{V_{in}\,I_{in}} \qquad \frac{\Delta\eta}{\eta} \approx -\frac{I_{out}R_{lead,out}}{V_{out}} - \frac{I_{in}R_{lead,in}}{V_{in}}
```

| Value | Number | Source |
|---|---|---|
| V_out, I_out | 12 V, 29.2 A | requirement |
| V_in | 13.5 V | lowest bus at the buck |
| I_in | 350 / (0.964 × 13.5) = 26.9 A | calculated efficiency |
| Lead resistance per side, 1 m leads | 8.6 mΩ | *(assumed)* 2 m of 10 AWG at 3.3 mΩ/m plus 2 mΩ of contacts |
| Lead resistance per side, 25 cm leads | 3.65 mΩ | *(assumed)* |

| Leads | Output error | Input error | A true 96.4 % reads |
|---|---|---|---|
| 1 m | 2.09 % | 1.71 % | 92.8 % |
| 25 cm | 0.89 % | 0.73 % | 94.9 % |

Method:

- **Voltage:** one DMM moved between TP_VBUS and TP_12V, so its gain error cancels.
- **Current:** supply readback for input, load readback for output, corrected by a null test. The supply is connected straight to the load with no board, stepped through 5, 10, 20 and 29 A, and the ratio of the two readings is recorded at each step. The input current is multiplied by that ratio.
- **Settling:** 5 minutes at each operating point before reading, since FET and inductor resistance rise with temperature.
- **Uncertainty target:** ±0.5 %.

Two figures are reported: buck efficiency from TP_VBUS to TP_12V, which is the ≥ 95 % requirement, and board efficiency from a battery input to TP_12V, which includes the power-path FETs.

## Equipment

| Item | Minimum | Used for |
|---|---|---|
| DC supply | 30 V, 30 A; plus a second small supply | All controlled tests; priority tests |
| Electronic load | 400 W at 12 V, 30 A, transient mode | Load, efficiency, load step |
| Oscilloscope | 4 channels, 100 MHz, spring-ground tips, single-shot | Switch nodes, ripple, switchover |
| Current probe | 100 A peak | Pack inrush at switchover (optional) |
| Thermocouple logger | 4 channels, type K | 30 minute soak |
| IR camera | — | Thermal image |
| DMM | 4½ digit | Kelvin voltages, shunt voltage |
| Battery packs and harness | Two charged, one near cutoff | Switchover, runtime |

## Setup

- Board on standoffs on a non-conductive surface.
- Scope grounds on board ground only. All measured nodes are ground-referenced.
- Switch-node and input-pin captures: spring ground, ×10 probe, full bandwidth. Output ripple: 20 MHz limit, measured across one output ceramic.
- Thermocouples: high-side FET tabs and power-path FET tabs are at DC potentials and can be measured directly, with Kapton under each bead. Low-side FET tabs are the switch node, so those thermocouples go on the plastic body.
- IR camera: bare copper and solder read low. Kapton dots on each part to be read.
- Pack discharge without the board: electronic load under-voltage cutoff at 15.0 V.

## Test sequence

Each step makes the next one safe to run.

| # | Test | Source | Method | Pass criterion (calculated value) | Result |
|---|---|---|---|---|---|
| 1 | Pack characterization | Pack into electronic load, no board | Discharge at 22–25 A to 15.0 V. Step the load for ΔV/ΔI. Thermocouple on the pack. Rest voltage at 1 and 20 min | ≥ 63 Wh usable (70 Wh assumed); rest voltage < 17.09 V (≈ 16.7 V); temperature rise ≤ 47 °C | pending |
| 2 | Unpowered checks | DMM | Resistance from each battery input, VBUS, 12 V and each switch node to ground; part orientation | No shorts | pending |
| 3 | First power | VBUS injection, 0.2 A limit | Buck disabled. Ramp 0 to 21 V, read supply current after 1 min | About 1 mA or less (bulk capacitor leakage) | pending |
| 4 | Power path | Supply on one battery input, limit ≥ 10 A, buck disabled | Capture VBUS at turn-on. Sweep the supply down and up. Repeat on each input, then two supplies for priority | Ramp at 5 V/ms after ~256 ms; UV 14.35–15.76 V; re-enable 17.09–19.96 V; OV 21.59–23.54 V | pending |
| 5 | Buck, no load | VBUS injection, 0.5 A limit | Enable the buck. Sweep VBUS up and down. Read TP_EN, TP_VCC, TP_12V | On at 14.65–16.81 V, off at 6.23–11.34 V; TP_VCC ≈ 5 V; output 11.4–12.6 V (11.82–12.18 V) | pending |
| 6 | Switch nodes under load | VBUS injection, 18 V then 23.5 V | Step 1, 5, 10, 20, 29.2 A. Capture both switch nodes at each step | Peak < 36 V; switching frequency 202.6 kHz ±10 % | pending |
| 7 | Efficiency | VBUS injection | 13.5, 18 and 23.5 V at 10, 25, 50, 75 and 100 % load | ≥ 95 % at 350 W (≈ 96.4 %) | pending |
| 8 | Regulation, ripple, load step, current sharing | VBUS injection | 0–29.2 A sweep. 18 A step at 13.5 V. DMM across each shunt | 11.4–12.6 V; ripple ≤ 32 mV; undershoot ≤ 0.42 V (0.35 V); shunt voltages within 10 % of each other (36.5 mV each); no current-limit trip | pending |
| 9 | Thermal soak | VBUS injection at 13.5 V and at 23.5 V; then one battery input lowered from 21 V to 16 V | 30 min each at full load. Thermocouples on both high-side FET tabs, one low-side FET body, both inductors, one shunt, the controller, one power-path FET tab, ambient. IR image at the end. V_DS of one power-path FET | Buck FET case and inductors < 125 °C (inductor ≈ 80 °C); power-path FET junction ≤ 125 °C (109–115 °C); power-path FET thermal resistance ≤ 40 °C/W | pending |
| 10 | Switchover, supply to pack | Supply on input 1, charged pack on input 2, full load | Lower the supply through the UV threshold. Single-shot trigger on VBUS falling through 14.5 V | Bus dip ≤ 0.5 V (0.26 V nominal, 0.33 V worst); output stays in tolerance; no false UV trip on input 2 | pending |
| 11 | Switchover, pack to pack | Nearly empty pack on input 1, charged pack on input 2 | Same capture. Leave the drained pack connected 20 min. Then insert a charged pack in slot 1 | Input pin peak ≤ 42 V (≈ 32 V); drained pack stays disconnected; inserted pack takes over after ~256 ms | pending |
| 12 | Abnormal conditions | Packs; supply at 50 mA limit | Pull the active pack at full load. Reversed supply on an input. Supply above the OV threshold | Bus ≥ 12.8 V and the buck stays on; channel stays off with no damage | pending |
| 13 | Runtime | Three charged packs, 350 W | Log VBUS, output and temperatures until the last pack disconnects | ≥ 30 min (≈ 33 min) | pending |

Notes:

- **Step 9:** the buck FET temperature prediction (≈ 126 °C junction plus inductor heating) was calculated for the FET used before the design review and will be replaced by the measurement.
- **Step 9:** power-path FET thermal resistance is (T_tab − T_ambient) / (V_DS × I/2).
- **Step 10:** the incoming source must be a real pack. A bench supply limits at 30 A, far below the 82–93 A inrush, so its output would collapse and trip UV on the incoming channel.
- **Steps 10 and 11, scope channels:** VBUS (DC, 2 V/div), 12 V output (AC, 100 mV/div), outgoing input pin (DC, 10 V/div), incoming input pin or pack current. Captured at 200 µs/div and 5 µs/div.

## Data

- `test-data/<date>_<test>_<condition>.csv` and `.png`, as exported by the instrument.
- `test-data/README.md` records instrument models, probe settings, ambient temperature and a setup photo.
- Plots are generated by script from the CSV files.
- Each measured value in the README's calculated-vs-measured table links to its raw file.
- A result that misses its prediction gets a row stating what was found and what changed.

## Not planned

- **Loop-gain (Bode) measurement.** The board has no injection resistor, and in fixed 12 V mode the controller senses feedback internally. The load-step response in step 8 is the stability evidence.
- **Current-limit trip point.** The criterion is no trip at full load at 13.5 V and 23.5 V.

## Assumptions

| Assumption | Value | Used in |
|---|---|---|
| Bench lead resistance per side | 8.6 mΩ (1 m) or 3.65 mΩ (25 cm) | Efficiency error without Kelvin points |
| Meter accuracy | DMM ±0.05 %; currents ±0.2 % after the null test | Efficiency uncertainty |
| Power-path FET current sharing | Half the channel current per FET | Thermal resistance measurement |
| Supply current limit during power-path turn-on | Must exceed the 9.4 A bulk-capacitor charging current, or the input sags and the channel retries | Step 4 supply setting |
