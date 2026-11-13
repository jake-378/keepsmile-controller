# Keepsmile Controller

An independent Python project for controlling compatible Keepsmile Bluetooth LED strips and bulbs from Windows.

## Why I built it

I bought Bluetooth LED strips that I could control from an iOS app, but the app experience frustrated me: it showed ads when I opened it and required an account just to control lights over Bluetooth. I also couldn't find a native Windows app for the model I bought. I built Keepsmile Controller to give myself a straightforward way to control those lights from Windows.

## Features

- Desktop GUI, command-line tool, and reusable Python library
- Power, colour, and brightness controls
- Reads the light's reported state when connecting
- Remembers the last colour and brightness for the GUI and CLI

![Keepsmile Controller GUI](docs/screenshot.png)

## Compatibility

The client discovers Bluetooth LE lights whose names start with `KS` and selects a protocol from the name:

| Bluetooth name | Protocol |
| --- | --- |
| `KS03-…`, `KS04-…` | Legacy |
| `KS03~…` and other names | Modern |

Tested with a `KS03~` strip on Windows 11. Legacy support is implemented but has not been tested against real hardware. Device support can vary by model.

## Installation

Requirements: Windows 10 or 11, a Bluetooth LE adapter, and Python 3.10 or newer.

```powershell
git clone https://github.com/jake-378/keepsmile-controller.git
cd keepsmile-controller
pip install -r requirements.txt
```

Close the Keepsmile phone app before connecting. The lights accept only one Bluetooth connection at a time.

## Use the desktop app

```powershell
python -m keepsmile.gui
```

The GUI scans for nearby lights and connects to the nearest supported `KS03` strip. Use the controls to switch power, choose a colour, and adjust brightness.

## Use the command line

```powershell
python -m keepsmile scan
python -m keepsmile on
python -m keepsmile color ff8000
python -m keepsmile brightness 60
python -m keepsmile off
```

Colour accepts either hexadecimal `RRGGBB` or comma-separated RGB values, such as `255,128,0`. Brightness ranges from 0 to 100. Run `python -m keepsmile --help` for options.

## Use the Python library

```python
import asyncio
from keepsmile import KeepsmileLight, scan

async def main():
    lights = await scan(timeout=5)
    found = lights[0]

    async with KeepsmileLight(found.device, found.name) as light:
        await light.turn_on()
        await light.set_color(255, 0, 128)
        await light.set_brightness(40)
        await light.turn_off()

asyncio.run(main())
```

## Documentation and development

- [Bluetooth protocol reference](docs/protocol.md)
- [Reverse-engineering notes and scripts](analysis/README.md)
- [Run the protocol tests](#testing)

Install pytest and run the tests with:

```powershell
pip install pytest
python -m pytest -q
```

## License and attribution

Licensed under the [MIT License](LICENSE). Keepsmile Controller is an independent project and is not affiliated with or endorsed by Keepsmile. It implements a protocol for interoperability with lights you own. The original app and its code are not included in this repository.