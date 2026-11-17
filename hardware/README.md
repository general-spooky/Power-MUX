# Hardware files

## Altium project (`altium/`)

Open `Battery ORing.PrjPcb` in Altium Designer.

| File | Contents |
|---|---|
| `BAT_Input.SchDoc` | Three battery inputs: fuse, TVS, input bypass, UV/OV divider and UV filter per channel |
| `FET_PowerPath.SchDoc` | Twelve P-channel FETs as three back-to-back channels, gate resistors, mid-node capacitors |
| `Battery_Logic.SchDoc` | LTC4417 controller, hysteresis resistor, status LEDs, VBUS bulk capacitors and TVS |
| `Buck_Converter.SchDoc` | LM5137-Q1 two-phase buck: controller, FETs, inductors, shunts, input and output capacitors, compensation |
| `Power_MUX_PCB.PcbDoc` | 4-layer layout |
| `Battery ORing_BOM.BomDoc` | Altium ActiveBOM document |
| `NetTie.SchLib`, `NetTie.PcbLib` | Analog-ground to power-ground net tie |

Component libraries came from manufacturer and Ultra Librarian downloads and are embedded in the documents.

## Bill of materials (`bom.csv`)

Generated from the four schematic sheets, grouped by block and part number.

## Datasheets

Datasheets are not redistributed here. Main parts:

- Analog Devices LTC4417 prioritized PowerPath controller
- Texas Instruments LM5137-Q1 dual-phase buck controller
- Toshiba TJ90S04M3L P-channel MOSFET
- Vishay IHLP-6767GZ-01 inductor
- Vishay WSLP3921 current-sense resistor
- Panasonic EEH-ZU hybrid capacitor
- Littelfuse SMCJ TVS diode
