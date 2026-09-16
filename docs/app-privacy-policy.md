# Springfield app — privacy policy

*Last updated 16 September 2026*

The Springfield app (package `dk.agesen.springfield`) is an instrument display
for Indian Thunder Stroke motorcycles fitted with the CANFD-MC interface. It is
published by Nanna Agesen, Denmark.

## What the app collects

**Nothing leaves your phone.** The app has no server, no account, no analytics,
no advertising and no crash reporting. It does not connect to the internet.

Data the app handles stays on the device:

- **Motorcycle data** (speed, revs, fuel, tyre pressures, fault codes and so on)
  is received over Bluetooth Low Energy from the interface on the bike and shown
  on screen. Some of it — tyre readings, the trip meter, the service interval —
  is stored on the phone so the app can show trends. It is never transmitted
  anywhere.
- **Heated-clothing control** (Keis) is sent over Bluetooth to your own garment
  controller.

## Permissions

| Permission | Why |
|---|---|
| Bluetooth (scan, connect) | To find and talk to the interface on the bike and to the heated clothing. The scan is declared *never for location*: the app does not derive, store or use your position |
| Foreground service, notifications | To keep the Bluetooth link alive while the screen is off, with a persistent notification while it runs |
| Vibrate | A short tick on a gear change and at the redline |

The app does not request location, camera, microphone, contacts, storage or any
other permission.

## Data sharing

None. No data is shared with the publisher or any third party.

## Deleting your data

Uninstalling the app removes everything it stored. There is nothing to delete
anywhere else.

## Contact

Nanna Agesen — via the GitHub project
[Prinsessen/indian-thunderstroke-canbus](https://github.com/Prinsessen/indian-thunderstroke-canbus).
