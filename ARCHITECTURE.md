# Pocket Workstation — Concept Architecture

This is the current concept architecture. The former CM5 build plan is archived under archive/ and is not a source of current requirements.

## Product idea

A handheld mini-PC that keeps its display, compact keyboard, pointing device, battery, and expansion connector when its compute board is replaced. The Raspberry Pi Zero 2 W is the low-power compute cartridge. The Raspberry Pi 4B is a larger, higher-power upgrade cartridge for tasks that need more CPU or memory.

The Zero 2 W has a 1 GHz quad-core Cortex-A53 and 512 MB RAM; Pi 4B boards have faster Cortex-A72 cores and ship with different RAM capacities. Record the specific Pi 4B model used in the evaluation. The two boards do not have the same physical layout, so the concept needs a shared carrier interface and a separate adapter or tray for each board.

## Main blocks

| Block | Concept role |
|---|---|
| Compute cartridge | Removable board tray for the Zero 2 W or Pi 4B. Swappable by the user with power off. |
| Common carrier | Standardizes the screen connection, USB input devices, storage, power, and expansion signals across board-specific adapters. On the Zero 2 W, provide a powered USB hub for touch, one removable-card reader, and user modules; choose a wireless keyboard or add hub ports for wired input. |
| Battery and power module | Replaceable battery pack, cell protection, charge control, and regulated system rail sized against measured screen and compute loads. No direct battery connection to GPIO. |
| Display and controls | Small HDMI display with a separate USB HID touch connection, compact keyboard, and pointing device attached to the chassis. Prefer a detachable keyboard so the unit can also be propped up as a mini-PC. |
| Expansion bay | A keyed connector for user-designed USB modules. ESP32-based modules own their sensor/actuator details and communicate through one versioned module protocol. Document power, USB, and any non-hot-plug GPIO buses separately. |
| Camera | A CSI camera module on the Zero 2 W's 22-pin camera connector, using the appropriate Zero camera flex cable. |
| Joysticks | Digital buttons can use protected GPIO. Analog sticks need an ADC or a USB HID microcontroller module; the Zero 2 W does not provide general-purpose analog inputs. |
| Storage | Keep the OS microSD installed in the board. Add one user-accessible microSD reader as USB mass storage through the powered hub, with safe unmount/eject before removal. |
| Enclosure | Concept CAD model with a compute bay, battery access, heat path, display hinge/stand, and expansion access. |

## Replaceable compute architecture

The common interface is a design boundary, not a finished connector selection. The concept schematic must show how each board adapter maps its native connectors to the same display, USB, power, and expansion signals. Board replacement is planned with the system off; hot-swapping a live computer board is outside the scope.

The shared shell should be sized around the larger Pi 4B cartridge if both boards must fit without changing the exterior. The Zero 2 W then uses a smaller adapter and leaves room for battery or expansion. If the intended product should stay as small as a Zero 2 W enclosure, show the Pi 4B as an alternative rear module or an enclosure revision instead of claiming both fit the same bay. The Pi 4B may need more cooling and a different power profile; do not claim it has the Zero 2 W's battery life. Treat the screen and carrier as common modules and calculate runtime from measured load.

## Software architecture

- Base: Raspberry Pi OS Lite of one recorded release and architecture. Audit packages and services; remove only optional software while retaining boot, kernel/firmware, device management, HDMI graphics, USB, microSD, and required network support.
- Bare-computing profile: boot to a Linux console with working HDMI output, keyboard input, storage, and module discovery. Touch input belongs to the graphical profile.
- User-operable profile: add a small graphical session, touch-friendly launcher, terminal, file access, and one lightweight editor. Raspberry Pi OS Lite is command-line-only; the graphical layer must be installed deliberately.
- Feature profiles: install/enable the camera tools, joystick mapping, and safe-removal UI only when the matching module is present. Keep hardware drivers and the small feature UI separate where practical.
- Development: Python, a small C++ compiler workflow, Git, and a lightweight editor or terminal editor.
- Browser: optional. Leave it out of the base profile; test NetSurf as a low-footprint option for simple pages if web access matters.
- Module broker: one small user-space service discovers USB serial modules, negotiates protocol version/capabilities, dispatches standard requests, and routes events to generic handlers. Device-specific sensor/actuator code stays on the ESP32 firmware. See MODULE_PROTOCOL.md.
- OS study: trace firmware-to-kernel-to-systemd boot, inspect services and device tree, build a custom kernel with one isolated configuration change while retaining stock fallback, and measure page faults/swap under memory pressure.
- Emulation: the product goal is a custom guest-to-ARM64 dynamic recompiler. Prototype a small ARM/Thumb guest instruction subset; use mGBA for real GBA gameplay baseline and measure N64 as a stretch on both boards. A complete console hardware model is separate follow-on work.
- Comparison: use the same task, image version, package set, display mode, storage class, and measurement method on both boards where possible.

## System boundaries

- The handheld is a concept model plus testable software setup; no custom PCB fabrication is required by the presentation date.
- KiCad 3D view and enclosure CAD show fit and routing intent. They do not emulate the Raspberry Pi board electrically.
- SPICE covers only the chosen power subcircuit and only to the accuracy of its component models and stated load assumptions.
- QEMU may support selected Pi 4 board behaviors, but its emulated peripherals and performance do not validate the physical Zero 2 W or handheld.
- Battery selection, charging, protection, connector current, and regulator design remain proposed until validated against suitable parts and measured loads.
- The official Raspberry Pi Touch Display 2 uses DSI and is not compatible with Zero 2 W. Use an HDMI panel with a USB touch controller for a common Zero/Pi 4 display path.
- The Zero 2 W has one USB 2.0 OTG data port. Touch, the single external card reader, the module broker link, and any USB keyboard/joystick share a powered hub and its bandwidth and power budget. Size the hub to the chosen peripherals. Keep CSI camera data on CSI rather than sending it through the low-rate module protocol.
