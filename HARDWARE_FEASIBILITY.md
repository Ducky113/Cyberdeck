# Pocket Workstation OS — Hardware Feasibility

**Scope:** Zero 2-family handheld supporting an OS/service prototype. This is a design assessment, not physical validation.

## Feasibility matrix

| Feature | Assessment | Condition |
|---|---|---|
| Lite-derived OS, SDK, module service | Feasible | Measure total footprint and latency on Zero 2 W |
| Small editor and graphical session | Feasible with a narrow app set | Select screen/session and test readability, touch, input, and RAM |
| HDMI video plus USB touch | Suitable common path | Select a Linux-compatible controller; include its board, cables, power |
| Same-family compute replacement | Feasible with power off | Verify exact Zero 2-family revisions, mounts, connectors, header height, image |
| USB sensor/controller modules | Feasible | Real MCU firmware, supported serial interface, documented descriptors |
| Automatic generic sensor use | Feasible for known capability schemas | SDK is preinstalled; unknown sensors are not magically identified |
| Camera | Optional | Correct CSI cable, supported Zero 2-family revision, powered-off connection |
| Removable user storage | Feasible through USB reader | OS card stays installed; unmount user card before removal |
| Battery operation | Possible, not yet engineered | Select parts after whole-system loads and runtime target are known |
| Full carrier PCB | Optional future work | Standard cables/hub support the OS demonstration first |

The Zero 2 W has 512 MB RAM, mini-HDMI, USB OTG, and a CSI camera connector. The official Touch Display 2 is incompatible with the Zero family, so the plan uses HDMI plus USB touch. [Zero 2 W specifications](https://www.raspberrypi.com/products/raspberry-pi-zero-2-w/), [Touch Display 2 compatibility](https://www.raspberrypi.com/documentation/accessories/touch-display-2.html).

## Zero 2-family interchangeability

The enclosure and compute tray target verified Pi Zero 2-family boards only. The Pi 4B is a separate development/comparison machine. Replacing a board requires shutdown, disconnecting power, and reconnecting its verified interfaces.

Record compatibility for each actual revision:

- Mounting-hole and connector location, cable access, and board thickness.
- Populated/unpopulated GPIO header height and clearance.
- Camera connector and flex cable compatibility for the selected Zero 2-family board and any future replacement.
- Wireless capability and antenna clearance; preserve a wired input/transfer option where absent.
- CPU architecture, OS boot files, compiled application target, and usable performance.
- Cooling clearance and peak/idle electrical load.

Use a recorded 64-bit Lite image and ARM64 packages, validated first on Zero 2 W. Original Zero/Zero W support is outside scope. The design assumes a future Zero 3 retains the same connection method and supports 64-bit software; its physical fit, power, and kernel/firmware support remain to be verified. A shared card/image is only claimed after boot testing on each supported model. [Raspberry Pi OS architecture guidance](https://www.raspberrypi.com/documentation/computers/os.html).

## USB topology and power boundary

The Zero data port is the upstream host link to a powered USB hub. Count ports for touch, data-card reader, MCU module(s), and wired keyboard. The separate power connector is not a second general-purpose USB data port. Do not promise simultaneous host-hub operation and an independent gadget connection through the same data link.

Use a hub/power arrangement with documented upstream power behavior. The eventual schematic must identify the single intended board supply path, peripheral supply path, common ground, current limits, and protection against unintended back-powering. Validate voltage at the board during boot, peripheral attach, and sustained load.

Size the system rail using measured Zero 2-family board, screen/controller, hub, and module loads, including transients. Include the regulator's usable current, conversion losses, connector ratings, and wiring drop. Pi 4 loads do not set the handheld's power requirement.

A known external supply is sufficient for the OS prototype. The battery concept needs a selected chemistry/capacity, protection, charging/power-path components, safe shutdown behavior, and physical space. Do not claim runtime or charging performance from a render or nominal battery capacity.

## Extension examples

The first real module should provide a scalar sensor and LED output. This validates discovery, reads, commands, shared library access, and failure cleanup. A second controller demonstrates buttons/ADC axes and standard Linux input integration. An ESP32-family board with native USB or a documented USB-to-UART bridge is suitable; select the exact board before firmware work.

The MCU identifies sensors through firmware. Generic I2C address scans cannot reliably identify all devices and do not replace capability descriptors. For raw GPIO/I2C/SPI attachment, define the device explicitly, respect voltage levels, and power down before changing connections. The Pi has digital GPIO, not general-purpose analog inputs; analog axes need an ADC or MCU.

Standard HID devices use normal Linux drivers. Camera video, storage, audio, and display traffic stay outside the low-rate capability protocol. The SDK unifies supported application operations without pretending every device belongs to the same kernel subsystem.

## Storage and mechanical validation

Keep the OS microSD installed inside the compute tray. Provide one USB reader for removable project files and a safe eject action. Handle removal during file access as an application error; never imply arbitrary unplugging preserves buffered writes.

Use a full-size enclosure mock-up to test reading code, typing, holding the device, reaching connectors, and supporting tabletop use. Include cable bends, panel controller, hub, cooling, and battery volume. Record proposed mass, dimensions, cost, and runtime once actual parts are selected. A Pi Zero-sized compute board does not determine finished product size.

## Evidence limits

A schematic and CAD show design intent. KiCad checks do not prove high-speed signal integrity or physical fit; SPICE models only the selected subcircuit and assumptions. QEMU does not validate handheld power, USB wiring, thermals, or actual emulator speed. Label bench measurements, datasheet facts, mock-ups, and proposed hardware separately.
