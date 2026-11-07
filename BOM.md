# Pocket Workstation OS — Concept Bill of Materials

The immediate goal is a bench OS prototype plus a handheld design. This is not a purchasing-ready BOM; prices, part numbers, availability, and final power requirements remain to be recorded.

## Available and additional compute hardware

| Item | Status | Role |
|---|---|---|
| Raspberry Pi Zero 2 W | Available per existing plan | Primary OS target and handheld board |
| Raspberry Pi 4B | Available per existing plan | Separate build/comparison machine; no enclosure cartridge |
| Second compatible Zero 2-family board | Not confirmed | Optional physical replacement/compatibility test |
| OS microSD | Select/record model | Installed system card; preserve known-good image |
| Spare development microSD | Recommended, availability unconfirmed | Fresh-image/recovery tests |
| User microSD and USB reader | Select | Removable data, separate from OS storage |

## Prototype priorities

| Priority | Component | Status / selection requirement |
|---|---|---|
| Core | Known suitable board power supply | Record rating and voltage under measured load |
| Core | Powered USB hub and OTG host adapter | Enough ports; documented upstream power behavior |
| Core | HDMI display and compatible cable | Existing bench monitor acceptable first; handheld panel unselected |
| Core | Keyboard / pointing device | Existing input acceptable first; avoid dependence on unsupported wireless |
| Core | ESP32-family or other MCU development board | Select exact native-USB or bridge interface and reset behavior |
| Core | One sensor, LED, and required small components | Implement scalar input and absolute LED output |
| Measurement | Whole-system power meter | Record resolution, logging/energy capability, and measurement boundary |
| Preferred extension | Second MCU/controller, buttons, joystick | Proves distinct capability types and gaming input |
| Handheld concept | Small HDMI/USB-touch panel | Confirm dimensions, controller board, power, and Linux behavior |
| Handheld concept | Zero 2-family tray and full-size enclosure mock-up | Fit around selected parts; no Pi 4 accommodation |
| Handheld concept | Battery/protection/charging/regulator | Select only after power and runtime requirements are quantified |
| Optional | Camera with correct Zero cable | Validate exact board revision; powered-off connection |
| Optional | Custom carrier PCB | Interface schematic first; layout/fabrication after OS milestones |

Use standard USB/HDMI connectors and cables for the initial build. A custom combined high-speed expansion connector is not required. Keep GPIO/CSI interfaces separate from live-plug USB modules.

## Software and tools

| Item | Role |
|---|---|
| Recorded 64-bit Raspberry Pi OS Lite | ARM64 base recipe for the Zero 2 family |
| Provisioning recipe; later pinned pi-gen custom stage | Repeatable image setup and packaged release |
| Project daemon, Python SDK, CLI, examples | Original OS interface contribution; implementation pending |
| Lightweight editor, Python, C/C++ tools | Programming and usability workflows |
| systemd/cgroup tools and measurement harness | Controlled resource-allocation experiment |
| mGBA and selected GB/GBC/NES/Sega cores | Gaming candidates; verify chosen ABI/build and performance |
| User-owned GBA emulator | Integrate when its Linux interface is ready |
| KiCad / CAD | Interface/power drawing and enclosure mock-up |
| SPICE | Optional selected power subcircuit once parts/models are available |

## Cost and evidence record

For each selected part, record exact model, quantity, available/borrowed/purchased status, measured load, physical dimensions, and actual or sourced cost. Report both additional spend and full replacement cost; owning the Pi does not make the finished design free. Set the prototype cost ceiling, dimensions, mass, and runtime target by 10 November.

The archived CM5 design's batteries, charging arrangements, switches, and part numbers do not carry forward as validated hardware.
