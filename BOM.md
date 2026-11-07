# Pocket Workstation — Concept Bill of Materials

This list separates hardware already available from concept components still to select. It is not a purchasing-ready design. Battery, charger, protection, and regulator part numbers require load calculations and datasheet checks before a real build.

## Available compute boards

| Item | Qty | Status | Role |
|---|---:|---|---|
| Raspberry Pi Zero 2 W | 1 | Available | Default low-power compute cartridge and primary target |
| Raspberry Pi 4 Model B | 1 | Available | Higher-performance comparison and upgrade cartridge |
| microSD cards | 2 | Select capacity/class | One remains installed for the OS; one is user-removable through an external USB reader. Record each card model for benchmarks. |

## Handheld concept hardware

| Subsystem | Concept item | Selection status | Notes |
|---|---|---|---|
| Carrier | Custom carrier PCB plus board-specific adapter trays | Design required | Common display, USB, storage, regulated power, and expansion interface |
| Power | User-replaceable battery cartridge | Not selected | Include proper cell protection and a charge path designed for the selected chemistry |
| Power | Regulator/power-path module | Not selected | Size from measured Zero 2 W, Pi 4B, screen, USB, and peak loads |
| Display | Small HDMI panel and controller | Not selected | Include the controller board's load and physical dimensions |
| Touch input | USB HID touchscreen controller integrated with/attached to HDMI panel | Not selected | Zero 2 W has no DSI; confirm Linux support and physical interface before selecting panel |
| Input | Compact keyboard and pointing device | Not selected | Detachable keyboard is preferred for handheld and tabletop use |
| USB expansion | Powered USB 2 hub | Not selected | Port count covers touch, one card reader, ESP32 module(s), and wired input if selected |
| Removable storage | One microSD-to-USB reader | Not selected | User data slot; show a safe unmount/eject action. OS microSD remains installed separately. |
| Camera | CSI camera and Zero 22-pin flex adapter | Not selected | Power off before changing CSI ribbon; confirm adapter and camera support for selected board |
| Joystick module | USB HID microcontroller or GPIO plus ADC | Not selected | USB HID is the easiest user-swappable option; analog sticks require an ADC |
| Protocol module | ESP32 board exposed as USB serial | Design required | Select native USB CDC or a board with an onboard USB-to-serial bridge; firmware owns device-specific handling |
| Expansion | Keyed board-to-board or cable connector | Not selected | Publish a pinout with ground, power rails, USB, I2C, SPI, UART, and protection limits |
| Cooling | Passive Zero 2 W solution; upgrade cooling for Pi 4B if needed | Validate by measurement | Enclosure airflow and thermal behavior must be considered |
| Enclosure | 3D CAD concept / mock-up | Design required | Show compute swap, battery access, display angle, keyboard, ports, and expansion bay |

## Software and measurement tools

| Item | Role |
|---|---|
| Raspberry Pi OS Lite | Minimal base image |
| Lightweight desktop/session and editor | Usable small-screen interface |
| Python, C++ compiler, Git | Everyday coding demonstration |
| Icarus Verilog and NumPy | Selected existing COL861 workload experiments, if installation fits |
| NetSurf browser | Optional lightweight web browsing experiment; omit if page compatibility or memory use is poor |
| mGBA / Mupen64Plus | GBA core target / N64 stretch test; measure actual Zero 2 W performance |
| USB power meter | Measure board and peripheral power at the supply input |
| KiCad and SPICE | Schematic, PCB layout, 3D board view, and selected power-circuit simulation |

## Do not reuse from the retired build plan

The former CM5-specific BOM included unvalidated component choices and topology assumptions for dual batteries, USB-C charging, ideal-diode sharing, HDMI switching, and CM5 carrier features. None of those part numbers or electrical claims carry forward as approved hardware.
