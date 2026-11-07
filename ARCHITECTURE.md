# Pocket Workstation OS — Architecture

## Software boundaries

```text
Application / editor-run program / dashboard
             ↕ bundled SDK or CLI
Versioned Unix-domain-socket API
             ↕
pocket-moduled.service
  registry · capability adapters · session state
  bounded queues · permissions · timeouts · reconnect
             ↕
Linux USB serial drivers + device-event monitoring
             ↕
USB hub → MCU firmware → sensor / LED / controller

Controller adapter → Linux uinput → games / normal input consumers
Standard HID / storage / camera / audio → existing Linux stacks
```

Original OS components are userspace packages shipped with the image. Existing USB drivers remain essential. A standard HID gamepad works without passing through the broker.

## Responsibilities

| Layer | Owns | Does not own |
|---|---|---|
| MCU | Hardware driver, sampling, debounce, unit conversion, local output timeout | Linux scheduling or editor integration |
| Kernel | Enumeration, class drivers, memory, scheduling, normal input devices | Project-specific sensor semantics |
| udev | Device metadata/permissions and events | Persistent protocol loops or package installation |
| systemd | Startup/restart, logging, service identity, resource policy | Sensor sampling |
| Broker | Handshake, registry, multiplexing, validation, recovery | Loading code supplied by a module |
| SDK / CLI | Application API, types, bounded waits, errors | Separate serial connections per application |
| Applications | Choose capabilities, request readings/actions, handle loss | Assume hardware is always present |

Run one broker at boot, even with no modules. Subscribe to device events before enumerating existing devices; deduplicate the paths. This handles boot attachments and broker restarts. Device rules set permissions and identify configured interfaces; the broker handshakes. Keep long-lived work outside udev rules. [systemd udev documentation source](https://github.com/systemd/systemd/blob/main/man/udev.xml).

## Planned packages

- `pocket-os-config`: profile manifests, systemd policy, device rules, release metadata.
- `pocket-moduled`: C++ event-driven daemon with bounded buffers and local IPC.
- `python3-pocket-modules`: Python library, typing information, examples.
- `pocket-module-tools`: `pocket-modules` CLI and offline help.
- `pocket-examples`: logger, LED example, optional dashboard.
- `libpocket-modules`: optional C/C++ client after the Python contract stabilizes.

These names are proposed; no package is implemented. See [MODULE_SERVICE_AND_SDK.md](MODULE_SERVICE_AND_SDK.md).

## Board compatibility

| Board | Physical role | Image policy | Status |
|---|---|---|---|
| Zero 2 W | Primary handheld | Recorded 64-bit Lite derivative; ARM64 packages | Available; image not tested |
| Another Zero 2 W | Powered-off replacement | Same recipe with per-device setup | Pending board availability |
| Future Zero 3 (assumed successor) | Prospective replacement using the same connection method | Assumed 64-bit; kernel/firmware and image support to verify | Design assumption; specifications and compatibility not validated |
| Pi 4B | External build/comparison | Same software/ABI for fair comparison where possible | Available; outside chassis interface |

Support is limited to the Pi Zero 2 family, with a forward-looking assumption that a future Zero 3 retains the connection method and 64-bit architecture. Check revision, connectors, header height, antenna clearance, mounting, cable access, power, and boot support before adding any replacement. Keep the OS card with its validated board/image unless shared-image compatibility is tested. Original Zero/Zero W boards are outside scope.

## Hardware topology

The chassis retains HDMI panel, keyboard/pointer, powered hub, external card reader, and module ports. The tray accepts only verified Zero 2-family boards with power off. No Pi 4 tray or Pi 4-sized shell is planned.

The Zero's USB data port operates as host for the hub. Ports serve touch, removable storage, MCU modules, and wired keyboard if selected. Bluetooth input is optional only where supported. CSI uses its own ribbon and camera stack. Pixels and storage blocks do not travel through the module protocol.

Use standard USB/HDMI cables for the prototype. A later carrier can tidy the interconnect after power and signal-routing review. Battery/charging work is separate; a known supply supports OS experiments.

## Resource and failure boundaries

Use sibling cgroups for interactive service/client work and background jobs when evaluating CPU weights. Weights operate relative to sibling contention and provide no real-time guarantee. Compare equal weights against the selected policy with identical software/workloads and unchanged memory settings.

Bound queues per module and client. Faulty devices cannot block other modules. Slow clients get an explicit overflow/resynchronization indication or disconnect. Removal invalidates handles, fails pending operations, and releases virtual inputs. Firmware returns outputs to the declared default after its host lease expires.

## Image and UI

Use 64-bit Raspberry Pi OS Lite and ARM64 application packages for the Zero 2-family target. Bundle service/library packages and a small editor workflow. Retain console operation; add a small graphical session for touch/dashboard/games after measuring footprint. SDK imports work in the bundled interpreter; independent virtual environments require documented installation or system-package access.

Supported modules become visible in registry/UI. Applications subscribe to readings/events. Connection does not launch arbitrary programs, inject code into editors, or download drivers from devices.
