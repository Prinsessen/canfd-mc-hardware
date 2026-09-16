# Wiring parts — antenna and service-connector cable

The two things the board needs that JLCPCB does not supply. Both are cheap; order
them with the boards so they arrive together.

## Antenna

**diymore 2.4G 5dBi FPC antenna, IPEX/U.FL, 50 Ω** —
[AliExpress 32997757781](https://www.aliexpress.com/item/32997757781.html).
The same flex antenna LilyGO ships with the T-2CANFD, and it has worked on the
bike since day one, so it is the known-good choice. ~US $0.35 each; buy five —
U.FL connectors are rated for about 30 mating cycles and the price is the same.

What to check on any substitute: connector **IPEX1 / U.FL / MHF1** (not MHF4),
2.4 GHz, 1.13 mm cable, 10–15 cm. Mount the flex on the inside of a plastic
cover, clear of steel and carbon fibre. Once the plug is seated, a drop of hot
glue or a piece of Kapton tape over the U.FL connector — it works loose under
vibration otherwise.

## Service connector (DIAG, C02)

The bike's diagnostic connector is an **Aptiv (Delphi) GT 150, 8-way**, male
header on the bike (Delphi 15326840). It is the same connector Polaris uses on
its ATVs, RZR and Slingshot — Indian is Polaris — so the cheap "Polaris OBD2
adapter" cables fit.

Pinout at the bike, from the firmware repo's FLASHING.md (cavities H G F E D C B A):

| Cavity | Wire | Signal | Board pad |
|---|---|---|---|
| H | C02-1 YE | CAN H | CANH |
| G | C02-2 DG | CAN L | CANL |
| F / E | GND3-03 BK | Ground | GND |
| (one more) | — | Permanent battery 12 V | VBAT — **identify with a meter before soldering** |

Do not connect the CAN shield; it is grounded at the ECM.

### Easiest: a ready-made adapter cable, OBD2 end cut off

**"OBD2 to 8 pin diagnostic adapter for Polaris ATV / Slingshot"** —
[AliExpress 1005004883918813](https://www.aliexpress.com/item/1005004883918813.html)
(also many eBay listings, "16 pin female to 8 pin OBD2 Polaris"). A moulded,
sealed GT 150 8-way plug with a 30–50 cm cable to an OBD2 socket, ~US $5–10.

Cut the OBD2 socket off and you have a sealed plug with exactly the wires the
board needs. Identify them with a meter against the OBD2 socket **before**
cutting: OBD2 pin 6 = CAN H, pin 14 = CAN L, pin 4/5 = ground, pin 16 = battery
12 V. Wire gauge is 22–20 AWG, plenty for < 200 mA. Leave a service loop.

An alternative with the same result: the official Indian OBD adapter harness,
part 2414868 — more expensive, same connector.

### If you would rather crimp your own

Aptiv GT 150 8-way sealed female housing **15326835** (black) or 15326839
(grey), with GT 150 female terminals and cable seals for 0.5–0.75 mm² wire, plus
the TPA lock. Kits with housing, terminals and seals are on AliExpress
("Delphi GT series 8 pin 15326835") for under US $1, and from Mouser, Waytek
or EFI Connection. Needs a proper open-barrel crimp tool — a pair of pliers does
not make a vibration-proof crimp. The adapter cable above avoids all of this.

### Bench check before connecting the board

Ignition off, measure across CAN H and CAN L at the plug: **about 60 Ω** (the
bus's two 120 Ω terminators in parallel). Open circuit means wrong pins.
