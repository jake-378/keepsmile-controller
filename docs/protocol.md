# Bluetooth protocol

Keepsmile Controller implements the Bluetooth Low Energy protocol used by compatible Keepsmile lights. The details below were recovered from version 2.3.5 of the Keepsmile iOS app. This is an independent interoperability implementation; device models and app versions may differ.

`RR`, `GG`, and `BB` are colour channels from 0 to 255. `LL` is brightness from 0 to 100.

## GATT services

The app looks for one of these services and writes commands to the matching characteristic:

| Service | Write | Notify |
| --- | --- | --- |
| `FFF0` | `FFF3` | `FFF3` |
| `AE00` | `AE01` | `AE02` |
| `AFD0` | `AFD1` | `AFD2` |

Writes use response when supported by the characteristic.

## Connection handshake and state

After connecting, newer lights using `AE00` or `AFD0` expect a state query. Without it, they may disconnect after roughly two seconds. The `FFF0` service uses a clock-sync packet instead.

| Service | Packet | Purpose |
| --- | --- | --- |
| `AE00` / `AFD0` | `5F 01 00 F5` | Query light state |
| `FFF0` | `7E 07 83 HH MM SS WD FF EF` | Set the light clock (`WD`: 1 = Sunday through 7 = Saturday) |

A state notification begins with `5F 06`. The response includes power, RGB colour, warm white, and brightness values. The client parses this report when available to initialize the UI with the light's current state.

## Commands

| Action | Legacy (`KS03-`, `KS04-`) | Modern (`KS03~` and other names) |
| --- | --- | --- |
| On | `7E 04 04 F0 01 01 FF 00 EF` | `5B F0 00 B5` |
| Off | `7E 04 04 10 01 00 FF 00 EF` | `5B 0F 00 B5` |
| Colour | `7E 07 05 03 RR GG BB 00 EF` | `5A 00 01 RR GG BB 00 LL 00 A5` |
| Brightness | `7E 04 01 LL FF FF FF 00 EF` | Same packet as colour, with the new brightness |

The modern protocol sends colour and brightness together. Changing either value requires sending the other value too. The app selects the protocol from the Bluetooth device name; the CLI can override it with `--protocol legacy` or `--protocol modern`.

## Compatibility notes

The modern protocol was tested with a `KS03~` strip on Windows 11. Legacy support is implemented from the app's protocol but has not been tested against real hardware. Names and behavior may vary across models.

For the reverse-engineering workflow, scripts, and notes about how these packets were identified, see [analysis/README.md](../analysis/README.md).