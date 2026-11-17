# Design decisions

Each choice, the alternatives, and the reason. Math: [calculations.md](calculations.md)

## Requirements

| ID | Requirement | Value |
|---|---|---|
| R1 | Inputs | 3 × Milwaukee M18 5 Ah packs (5S Li-ion), 15.0–21.0 V |
| R2 | Load | 12 V, 350 W continuous (29.2 A): lights, fan supply, controllers |
| R3 | Drain order | One pack at a time, priority 1 → 2 → 3, no current between packs |
| R4 | Pack cutoff | 15.0 V at the board input |
| R5 | Runtime | ≥ 30 min (hard requirement) |
| R6 | Hot swap | Idle packs can be swapped while running |
| R7 | Converter | Custom buck around a controller IC, efficiency ≥ 95 % |
| R8 | Heat | No heatsink |
| R9 | Integration | Power path and buck on one board |
| R10 | Output tolerance | 12 V ±5 % |

## Power path

### Priority switching instead of diode-OR

In a diode-OR each diode drops about 0.5 V, which is 11 W at the average pack current and 14 W at end of pack. It also lets whichever pack is highest supply the load, so packs drain together and there is no clear "swap this one" signal. The LTC4417 connects one input at a time through back-to-back P-channel FETs. Path loss falls to about 4.3 W, packs never feed each other, and a reversed pack is blocked by the FET pair.

### FET selection by on-resistance at 4.5 V

The LTC4417 clamps its gate drive and only guarantees 4.5 V. A FET's headline current rating and its 10 V on-resistance are irrelevant; what matters is R_DS(on) at −4.5 V and how well the package gets heat into copper.

| Candidate | R_DS(on) at −4.5 V | Outcome |
|---|---|---|
| FDS4685 (used in the controller's datasheet examples) | 35 mΩ | Rejected: would need 36 FETs and still lose 11 W |
| Vishay SQJ147ELP | 18 mΩ | Fallback at three per side |
| Infineon IPB180P04P4L-02 | 3.9 mΩ | Rejected on cost |
| Toshiba XPH4R714MC | 6.9 mΩ | Runner-up, smaller 5 × 5 mm package |
| **Toshiba TJ90S04M3L** | **6.0 mΩ** | **Chosen: large DPAK+ tab, 0.83 °C/W junction to case** |

### Two FETs in parallel per side

One FET per side would dissipate about 6.5 W at end of pack, which board copper cannot remove. Two share the current, each dissipates a quarter of that, and the pair stays near 115 °C at worst case. Mismatch helps rather than hurts: the lower-resistance FET takes more current, but on-resistance rises with temperature, so sharing balances itself.

Heatsinks were considered and set aside. They lower temperature but not the wasted power, the drain tabs sit on different nets so each would need insulation, and tall parts are a liability under vibration. The layout instead uses large single-net pours with via arrays under every tab.

### Cutoff at 15 V, measured under load

The under-voltage comparator sees the pack while it is delivering about 25 A, when its internal resistance drops roughly 1.7 V. A reading of 15 V therefore corresponds to about 16.8 V open-circuit, or 3.36 V per cell. Raising the cutoff would leave 10–15 % of each pack unused and miss the 30 minute requirement.

### Large hysteresis

After cutoff a pack recovers toward about 16.7 V. The controller's default 30 mV of hysteresis would let it reconnect and oscillate. A 182 kΩ resistor on the HYS pin sets 3.46 V of hysteresis, putting the worst-case re-enable threshold at 17.09 V, above any rested pack.

### Enable and shutdown pins left open

Both pins act on all three channels at once. Using either would break the drain-one-at-a-time behaviour, so both are left on their internal pull-ups. If per-pack over-temperature protection is added later, it belongs on each channel's UV pin.

### A small UV filter instead of the alternatives

When a fresh pack takes over, inrush into the bus capacitors dips its own sense pin below the cutoff for about 166 µs. Three fixes were evaluated:

| Option | Result |
|---|---|
| Input bulk capacitor on each channel (the datasheet's equation) | Needs about 730 µF at worst case, must survive a reversed pack, and creates a ~260 A spark on insertion. Rejected |
| Gate slew-rate network to slow the turn-on | Works, but adds parts and stretches the switchover gap. Kept in reserve |
| **RC filter on the UV pin** | **One 100 pF capacitor per channel. Chosen** |

The capacitor value has a narrow window: 47 pF leaves too little margin when a fresh pack takes over, and 220 pF lets the bus fall too far when the active pack is pulled.

### Four hybrid capacitors on the bus

The requirement is 1226 µF. Four 470 µF hybrid polymer-aluminium parts give 1880 µF with a quarter of the ESR and inductance of a single can, enough ripple-current rating to share the buck's input ripple, low ESR when cold, and a short, vibration-tolerant package. If one fails open the droop rises from 0.33 V to 0.35 V and the system keeps working.

## Buck converter

### Controller: TI LM5137-Q1

The controller had to do four things at once: reach 100 % duty (12 V from a 13.5 V bus needs D ≈ 0.94), run two phases into one output, tolerate the bus TVS clamp voltage plus switch-node ringing, and provide an enable input with programmable hysteresis so the buck's turn-on and turn-off could be fitted around the power path's thresholds. Of the parts compared from their datasheets, only the LM5137-Q1 did all four.

### Two phases

Splitting 29.2 A into two 14.6 A phases halves the conduction loss in each FET and inductor, and interleaving the phases 180° apart roughly halves the input ripple current (7.3 A against 14.3 A for a single phase).

### 202.6 kHz

Switching loss scales with frequency. At about 200 kHz the high-side FET loss is still dominated by conduction. The inductor gets no larger as a result, because its value is set by the controller's slope compensation rather than by ripple.

### Shunt sensing instead of DCR sensing

Sensing across the inductor's own resistance saves two parts but inherits copper's temperature coefficient and the inductor's tolerance. The resulting current limit would range from 15.5 A to 26.4 A, low enough to trip at full load when hot. A 2.5 mΩ, 1 % shunt gives 21.3–26.7 A.

### Inductor: Vishay IHLP6767GZER6R8M01

| Candidate | Outcome |
|---|---|
| Würth WE-HCI ferrite, 6.8 µH | Rejected: ferrite saturates abruptly near the current limit, worse when hot |
| CODACA CSBX1809-6R8M | Lowest resistance; no verified footprint available |
| Coilcraft XAL1510 | Passes; rejected on cost |
| **Vishay IHLP6767GZ, 6.8 µH** | **Chosen: soft saturation, holds 90 % of its inductance at 20 A** |

The trade is copper loss: the IHLP has about twice the resistance of the CODACA part, costing roughly 2.3 W. Efficiency still clears 95 %, and the layout separates the inductors thermally from the high-side FETs.

### FETs: one logic-level part in all four positions

The gate driver runs from about 5 V, so the FETs must be specified at 4.5 V gate drive. Because the duty cycle is high (0.54–0.94), the high-side FET conducts most of the time and low on-resistance matters more than low gate charge, so the same part is used high and low. The first selection was a 5 × 6 mm power-SON part. After the design review it was replaced with a TO-252 part with one pad per terminal, which makes the high-current routing simpler and matches the footprint style already used in the power path.

### Bootstrap capacitors: 0.47 µF

Sized so each gate-charge delivery sags the bootstrap rail by 100–200 mV. The usual 0.1 µF would sag by roughly 400 mV with FETs of this gate charge.

### No snubber footprints

The board carries no unpopulated parts. Switch-node ringing is addressed by layout (a four-layer vertical hot loop with local ceramics at each high-side drain). This is a deliberate risk, listed under [known limitations](design-review.md#known-limitations), and the switch nodes are the first thing probed at bring-up.

## PCB

### Four layers with ground on L2

On a two-layer board the switching current's return path is far from its forward path, which raises loop inductance and switch-node overshoot toward the FETs' 40 V rating. With a solid plane 2.8 mil under the top layer, the return current flows directly beneath the forward current.

### 2 oz outside, 0.5 oz inside

Current and heat stay on the outer layers. The inner layers only provide the reference plane and a quiet routing layer, so thin copper is enough.

## Mistakes caught during validation

Each of these passed a first check at nominal values and failed when the whole design was re-run at worst case.

| What was wrong | How it was caught | Fix |
|---|---|---|
| OV divider could reject a full pack | Adding pin leakage to the tolerance stack put the worst-case trip at 21.06 V | 226 kΩ / 487 kΩ → 237 kΩ / 475 kΩ |
| A drained pack could reconnect | Hysteresis current tolerance put the worst-case re-enable at 15.83 V | R_HYS 316 kΩ → 182 kΩ |
| Input TVS exceeded the pin rating | Clamp voltage had been checked for energy only; at the real 25.6 A it reached about 50 V against 42 V | SMBJ26CA → SMCJ22CA |
| Status LED current at the pin's limit | 1.9–2.2 mA against a 2 mA absolute maximum | 10 kΩ → 20 kΩ |
| UV filter capacitor far too large | Time-step model showed the bus collapsing when the active pack is removed | tens of nF → 100 pF |
| Buck turn-on threshold impossible to meet | The controller's ±5 % enable tolerance did not fit the originally derived window | Window redefined; soft-start lengthened to 20 ms to cover the case it had protected |
| Inductor chosen before the controller | Slope compensation and current limit needed a different value and saturation rating | 5.5 µH → 6.8 µH, saturation ≥ 27 A |
