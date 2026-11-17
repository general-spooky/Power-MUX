# Design review

The design went through two kinds of review: a full worst-case re-validation of every calculated value, and an external review of the schematic and layout by practicing engineers from industry.

## Internal validation pass

Before layout, every value in the power path was recomputed at worst case: resistor tolerance, comparator accuracy, pin leakage, capacitor tolerance, maximum datasheet timings, and hot on-resistance. Four parts that had passed at nominal values failed and were changed:

| Finding | Change |
|---|---|
| Worst-case over-voltage trip of 21.06 V could reject a fully charged pack | Divider changed to 237 kΩ / 475 kΩ |
| Worst-case re-enable threshold of 15.83 V let a drained pack reconnect | Hysteresis resistor changed to 182 kΩ |
| Input TVS clamped near 50 V at the real switching current, above the 42 V pin rating | Changed to SMCJ22CA (≈ 32 V) |
| Status LED current of up to 2.2 mA exceeded the pin's 2 mA limit | Series resistors changed to 20 kΩ |

Details are in [design-decisions.md](design-decisions.md#mistakes-caught-during-validation).

## External review

Schematic sheets and the layout carried short plain-language notes explaining the reason for each block, so reviewers could read the intent on the drawing.

| Review comment | Response |
|---|---|
| The buck FET package (5 × 6 mm with three separate source pins) is awkward to route at this current | Replaced all four with a TO-252 part with one pad per terminal, the same footprint style as the power-path FETs |

Issues identified in self-review and presented to the reviewers:

| Issue | Disposition |
|---|---|
| The inductors have high winding resistance and may heat the adjacent FETs | Quantified (about 2.5 W each, 8–28 °C at the FET). Layout steers inductor heat toward the output copper; thermocouple check added to the test plan |
| The switch-node copper narrows to about 1 mm at one point | Kept, with wide copper on both sides of the neck; added to the thermal-imaging check |
| The 10 µF input capacitors are about 25 mm from the low-side FET | Kept; the 10 nF capacitors at the drain close the high-frequency loop. Switch-node ringing is the first measurement at bring-up |

## Failure modes

| Fault | What happens | Outcome |
|---|---|---|
| Buck high-side FET shorts | 12 V rail rises to pack voltage. Output capacitors survive (25 V and 35 V ratings) | **Not protected on this board**; see limitations |
| Buck low-side FET shorts | High-side FET shoots through; this current bypasses the shunt. The high-side FET fails, then the pack fuse opens | Fuse is the backstop |
| Bulk capacitor shorts | Active pack's fuse opens; the controller connects the next pack into the same short and that fuse opens too | Fails safe, at the cost of three fuses |
| Bus-side power-path FET shorts | That channel behaves like a diode; a higher-voltage idle pack can feed the bus. Reverse current is still blocked | Degraded, safe |
| Battery-side power-path FET shorts | The bus can back-feed that pack through the other FET's body diode | **Single-point fault**, no on-board detection |
| Input TVS shorts | The fuse ahead of it opens | Safe |
| One bulk capacitor fails open | Switchover droop rises to 0.35 V | Still works |
| Pack inserted backwards | Back-to-back FETs block, the TVS is bidirectional, the input capacitor is a non-polar 50 V ceramic | Safe |
| Active pack pulled out under load | Bus dips to about 12.8 V worst case, the buck rides through, the next pack takes over. The connector arcs | Works, but only idle packs should be swapped |

## Known limitations

Stated up front, each with the plan that addresses it.

1. **Runtime depends on an unmeasured number.** The 30 minute requirement needs about 63 Wh of usable energy per pack at 25 A; 70 Wh is assumed. *Plan: pack discharge test, first on the list.*
2. **Buck high-side FET thermal margin is thin and rests on an assumed thermal resistance.** *Plan: thermocouples at full load for 30 minutes at both input-voltage extremes.*
3. **No snubber provision on the switch nodes.** *Plan: probe the switch nodes at first power-up. If they ring, snubber footprints are added in a board revision.*
4. **No output over-voltage protection** against a shorted high-side FET. Loads must tolerate pack voltage, or a crowbar is added in a later revision.
5. **Pack temperature.** An upper-bound estimate with no cooling gives a 47 °C rise over one discharge. *Plan: thermocouple on the pack during the discharge test.*
6. **Not yet simulated in SPICE.** Switchover behaviour was modelled with a worst-case time-step model. *Plan: LTspice run of the pack change, and a bench load-step for the buck loop.*
7. **Worst-case gain margin of the buck loop is estimated at about 8 dB** at end of pack, below the usual 10 dB target. *Plan: load-step test at 13.5 V in; the fix is a lower compensation resistor.*
8. **Battery and output connectors are placeholders.** The schematic carries an interim input connector to be replaced with parts rated for at least 40 A in and 30 A out.
