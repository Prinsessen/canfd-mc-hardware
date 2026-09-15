# Review of rev 1.0 — 2026-09-15

A second pair of eyes on the design before the boards arrive, written on the
openHAB server with the firmware repository open beside it. Two parts: what was
checked mechanically, and an opinion on the three design decisions.

## 1. Net list, checked against the datasheets

Checked from `canfd-mc.kicad_pcb` — the pad-to-net assignment that was actually
fabricated — against the firmware's pin map and against Espressif's WROOM-1U
and TI's LM5164 datasheets, not against the design document.

**Everything matches.**

- ESP32 ↔ MCP2518FD is the firmware's map exactly: SCLK 12, MOSI 11, MISO 13,
  CS 10, INT 8. IO9/IO3 are the controller's INT0/INT1 outputs with no pull-ups,
  as on the LilyGO. MCP_INT has a 10 k pull-up, and GPIO8 is RTC-capable, so
  the ext0 wake in `sleep.cpp` works unchanged.
- All 41 module pins agree with the WROOM-1U datasheet, including USB_D− on 13,
  USB_D+ on 14, **RXD0 on 36 and TXD0 on 37** (checked twice; the datasheet
  says pin 36 is U0RXD/GPIO44 and pin 37 is U0TXD/GPIO43).
- LM5164 pin for pin: GND · VIN · EN/UVLO · RON · FB · PGOOD · BST · SW, EP to
  GND. Enable thresholds 1.5 V / 1.4 V and FB reference 1.2 V as used in §4.5.
  The type-3 ripple network is wired as the datasheet draws it: RA from SW to
  the injection node, CA from that node to VOUT, CB from that node to FB.
- Input chain VBAT → F1 → D1 (anode towards the fuse, cathode towards the buck,
  so the right way round) → D2 → C1/C2 → U3.
- Bus front end: TCAN332G → L1 (windings 1–2 and 4–3, Murata's numbering) →
  PESD2CAN on the bus side → pads. RT1/RT2/CT1 unpopulated and marked.
- The strain-relief holes sit between pad and board edge, as §6.0 describes.

## 2. Three things for rev 1.1 — none of them blocks bring-up

1. **The PSRAM open point is cheaper than §2 says.** The firmware is built for
   the PlatformIO board `esp32-s3-devkitc-1`, which is the *"N8, No PSRAM"*
   definition: partition table `default_8MB.csv`, no `BOARD_HAS_PSRAM`. The
   PSRAM is never initialised and only 8 MB of the flash is used. So the
   85 °C variants — **N16R2 or plain N8** — need **no firmware change at all**.
   Open point 7 can be closed as a purchasing decision; I would order rev 1.1
   with an 85 °C module without waiting for the temperature measurement.
2. **Leftover net `ACC_SENSE` on IO1 (pin 39).** One pad, no track, so DRC has
   nothing to complain about. Harmless — IO1 is not a strapping pin — but §4.1
   says IO1 is NC. Delete the label.
3. **BOM comment `22uF 10V`** on C3/C4/C12/C13. LCSC C45783 is Samsung
   CL21A226MAQNNNE: 22 µF **25 V** X5R 0805. The part number drives the order,
   so only the text is wrong.

The firmware side of the §9 checklist — the comment in `can_hal_mcp.cpp` about
Longan_CANFD assuming 40 MHz — is fixed in the firmware repository.

## 3. Opinion on the three decisions

**Yes: the minimal board is the better solution, and each of the three choices
is right for this job.** This is no longer a development board; it is a
permanently installed, single-purpose device on a vehicle that vibrates and
lives off a battery. Those are different requirements from the ones the LilyGO
was built for.

**One channel.** Obvious. The firmware has only ever used CAN A. Channel B was
dead weight, board area and current.

**Solder pads instead of screw terminals.** Right, on a motorcycle. Screw
terminals work loose under vibration, and a solder joint on a vehicle fails at
the root of the wire, not in the solder — which is exactly what the
strain-relief holes address. What you give up is that swapping a board becomes
a soldering job. With five boards and a small service loop in the wires, that
does not matter.

**No galvanic isolation.** Also right, for the reason §1 of the design document
gives: supply and bus come from the same service connector with the same
ground, so the isolation was bridged the moment it was wired in, and its DC/DC
cost three quarters of the sleep current. Isolation protects against ground
offsets between two systems. Here there is one system.

Where the price of those choices actually lies:

- **The one-ground rule has to hold all the way.** The single situation in
  which isolation would have saved something is USB ground to a mains-powered
  PC while the board is on the bike. Bring-up over the USB pads should happen
  with the board off the bike, or with a laptop on battery. Worth a line in
  BUILD-NOTES.
- **The starter motor.** UVLO releases at 6.5 V. A cold crank can dip below
  that. The consequence is a reboot at every start: lost seconds and a BLE
  reconnect. The design document says the wake path tolerates it, and it does.
  But measure the dip during cranking before rev 1.1 is frozen. If it is a
  problem, the answer is a regulator with a lower minimum input, not a bigger
  capacitor.
- **Sleep current.** Without the isolation module, 7–9 mA is expected. That is
  about 0.2 Ah a day — a couple of months of rest on a healthy battery, fine
  for a bike that is ridden. Going lower means the transceiver (the 332G has no
  standby pin) and the MCP2518FD in Normal mode, and that needs a different
  wake strategy. Not now.
- **Temperature.** See item 1 above: an 85 °C module costs nothing in firmware.
- **First-revision risks** are the two the design document already caught: the
  choke footprint with Murata's numbering, and the crystal load capacitors.
  Both are the classic "reads zero frames" faults, and both are handled.

So: not "good enough", but the right design. What is left, only the bench can
answer.
