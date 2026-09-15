# Build notes — CANFD-MC rev 1.0

What is in the repository, how to regenerate the production files, how the
board was ordered, and how to bring it up. The design itself — BOM, net list,
the LM5164 calculation, layout rules — is in
[design-document.html](design-document.html).

## Files

| File | What |
|---|---|
| `canfd-mc.kicad_pro` | The project — open this one. Design rules and net classes (Default / Power / CAN) are set for JLCPCB 4-layer |
| `canfd-mc.kicad_sch` | Schematic. Every connection is a local label, grouped in five rows. Symbols are embedded, so it opens without library errors |
| `canfd-mc.kicad_pcb` | Board: 55 × 32 mm, 4 layers, stackup ≈ JLC04161H-7628, placed and routed, DRC clean |
| `canfd_local.pretty/` | Three project footprints: solder pads with a strain-relief hole (Ø2.0 and Ø1.5) and the Murata DLW43SH with Murata's pin numbering (KiCad's built-in 4532 choke footprint uses TDK's numbering, which would short the windings) |
| `fp-lib-table` | Registers `canfd_local` for the project |
| `production/canfd-mc-gerber.zip` | Upload as the PCB file: 4 copper layers, mask, paste, silkscreen, outline, PTH + NPTH drill files |
| `production/canfd-mc-BOM-JLCPCB.csv` | BOM for assembly — 26 lines, 37 parts, all with LCSC numbers. RT1/RT2/CT1 (do not populate) are omitted |
| `production/canfd-mc-CPL-JLCPCB.csv` | Placement file — 37 parts, top side |

Routing: power loop, crystal and CAN chain on top; SPI/INT, CAN TX, MCP_INT,
flash pads and VBAT on the back; GND plane on In1, 3V3 plane on In2, GND fill on
the back. Flash pads on the back, left to right: GND · EN · TX · RX · IO0 ·
USB_D+ · USB_D− (2.54 mm pitch, names on the silkscreen).

## Regenerating the production files

Edit in KiCad, run DRC (`Inspect ▸ Design Rules Checker`), then
`File ▸ Fabrication Outputs ▸ Gerbers` (all copper layers, F/B.Silkscreen,
F/B.Mask, F.Paste, Edge.Cuts; tick *Generate Drill Files*) and
`File ▸ Fabrication Outputs ▸ Component Placement` (CSV, mm, top). The BOM
comes from `File ▸ Export ▸ Bill of Materials` with the `LCSC` field. The same
can be done from the command line with `kicad-cli`.

Board Setup → Constraints must have *Minimum drill size* at 0.3 mm — the ESP32
module's thermal vias were enlarged from the library's 0.2 mm so the board can
be made with JLCPCB's standard via option.

## Ordering at JLCPCB (as done for rev 1.0)

- Upload `canfd-mc-gerber.zip`. Detected automatically: 4 layers, 55 × 32 mm.
- **PCB:** FR4 TG135 · 1.6 mm · LeadFree HASL · 1 oz outer / 0.5 oz inner ·
  min via 0.3 mm/(0.4/0.45 mm) · via covering as offered (Plugged on 4-layer) ·
  *Confirm Production File: Yes* · *Mark on PCB: Remove Mark*.
  Do not pick the 0.2 mm via option — it silently adds a 4-wire Kelvin test and
  a TG155 laminate.
- **Assembly:** *Standard* PCBA (the ESP32 module is "Standard only"), top side,
  edge rails added by JLCPCB (Standard needs ≥ 70 mm per side; the rails are
  V-scored and snap off), *Confirm Parts Placement: Yes*.
  Leave *Conformal Coating* off — it would coat the flash pads, the solder pads
  and the U.FL connector, all of which you need bare until after bring-up.
- **Parts:** all 26 lines match on LCSC. If something is out of stock, take the
  suggested substitute for passives, not for U1–U4, X1, L1, L2.
- **Placement:** JLCPCB's rotation convention differs from KiCad's for SOIC
  packages; U2 and U4 needed a 90° turn in their viewer. Check pin 1 of U1–U4,
  D1 (cathode towards C1), D3 and L1 against the silkscreen marks on the
  engineer's confirmation image before approving.
- Shipping DDP if offered — the value is above the EU 150 € threshold, and DDP
  avoids the courier's import-handling fee at the door.

## Bring-up

1. Bench supply, 12 V, **current-limited to 100 mA** for the first power-up.
   Check 3.3 V on C3/C4, note the quiescent current.
2. First flash over native USB: a USB cable with the device end cut off —
   green D+ → `USB_DP`, white D− → `USB_DN`, black → `GND`, red not connected.
   Pogo pins or three temporary wires on the back-side pads. The ESP32-S3
   enumerates as a USB JTAG/serial port; `pio run -e sniffer-t2can -t upload`.
   If it will not enter the bootloader on its own: hold `IO0` low, pulse `EN`
   low, release `IO0`. A 3.3 V USB-TTL adapter on `TX`/`RX` works as a fallback,
   but with `ARDUINO_USB_CDC_ON_BOOT=1` the serial log only appears on USB.
3. Bench CAN against a USB-CAN adapter needs termination at both ends — fit two
   60.4 Ω on RT1/RT2 *temporarily*, and remove them before the bike.
4. On the bike: the 60 Ω check across CAN H/CAN L with the ignition off, as in
   the firmware repo's FLASHING.md, before the board is connected.
5. Solder the four wires (through the relief holes first), the antenna pigtail,
   then coat. Measure the sleep current in series with 12 V and compare with the
   17 mA baseline in SLEEP.md.

## Three things to know about the BOM

1. **Crystal and load capacitors.** JLCPCB's 40 MHz 3225 crystal (YXC
   X322540MPB4SI) has CL = 15 pF, not ~10 pF like LilyGO's. Hence C10/C11 are
   22 pF, not 12 pF. Rated −40…+85 °C — the ESP32 module is the limit at 65 °C anyway.
2. **The input TVS is bidirectional** (SMBJ24CA). It cannot be placed backwards,
   and D1 handles reverse polarity. It still clamps load dump at ~39 V.
3. **L1 (Murata DLW43SH) does not tolerate hard potting.** Murata's spec §13.4:
   cure stress from resin can change the inductance or break the wire. Use
   conformal coating or soft silicone — not epoxy — over L1 and X1.

## Open points

- N16R8 (65 °C) vs. N16R2 (85 °C): same footprint; measure the temperature at
  the mounting spot before deciding for a second revision.
