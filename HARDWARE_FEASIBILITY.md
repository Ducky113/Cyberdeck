# Pocket Workstation — Hardware and OS Feasibility

This review targets the Raspberry Pi Zero 2 W as the primary low-power board, with the Raspberry Pi 4B as an upgrade cartridge. It is a design feasibility assessment, not a tested prototype.

## Feature assessment

| Feature | Feasibility | Design choice and limit |
|---|---|---|
| Raspberry Pi OS Lite base | Yes | Lite is command-line-only. Keep boot, kernel/firmware, device management, HDMI, USB, microSD, and any needed network support. Remove optional apps and services carefully. |
| Bare computing mode | Yes | Boot to a Linux console over mini-HDMI with keyboard input. Touch is useful in the graphical profile, not a practical console control by itself. |
| Mini-PC desktop | Yes, with a narrow app set | Add a lightweight graphical session and touch-friendly launcher, terminal, editor, file access, and required input support. The Zero 2 W has 512 MB RAM; omit a browser initially and benchmark optional lightweight browsing separately. |
| HDMI display output | Yes | The Zero 2 W has mini-HDMI video output. Include the graphics/display stack in the base OS; use a small panel resolution to preserve responsiveness. |
| Touchscreen | Yes, with a different panel interface | Use HDMI for video and a USB HID touch controller for touch events. The official Raspberry Pi Touch Display 2 uses DSI; Zero 2 W has no DSI connector, so that screen is incompatible. |
| CSI camera | Yes | The Zero 2 W includes a mini 22-pin CSI camera connector. Use the appropriate flex adapter and camera package. Raspberry Pi OS Lite includes the lightweight rpicam application package. The ribbon connection is not a hot-plug connector; power off before changing it. |
| Joysticks | Yes | Buttons can use GPIO. Analog stick axes need an ADC or an RP2040/other USB HID controller module; the Pi GPIO is digital and is not 5 V tolerant. USB HID is the simpler user-swappable design. |
| One hot-swappable microSD data card | Yes, using a USB reader | Keep the board's native microSD card for the OS. Connect one external USB reader through the powered hub and provide an unmount/eject action before removal. |
| Hot-swappable OS card | No, not while running from it | Removing the root/boot medium while the system is active is unsafe. A USB-boot arrangement is possible in principle, but then the boot card occupies one reader and cannot be removed. |
| User-swappable compute board | Yes, as a powered-off cartridge | Use separate mechanical/electrical adapters for the Zero 2 W and Pi 4B. They are not pin-compatible modules and differ in size, connectors, cooling, and power demand. |
| Hot-plug expansion modules | Some | Prefer USB for live insertion. Treat CSI ribbon and raw GPIO/I2C/SPI modules as power-off swaps unless the module connector includes deliberate sequencing, protection, and bus isolation. |

## USB and storage topology

The Zero 2 W exposes one USB 2.0 OTG data interface. A powered hub connects the touch controller, one USB card reader, ESP32 module links, and any wired keyboard. Budget hub ports and power for the actual inputs; Bluetooth can avoid a keyboard USB port. Camera data uses CSI, not the module protocol.

The storage design has two cards in total:

1. The board's microSD card holds the OS and stays installed while the handheld is running.
2. The exposed slot is a USB microSD reader for removable user files or projects.

Use the desktop's eject/unmount action before removing the card to flush writes and avoid filesystem corruption. Keep the OS card behind a service cover or inside the compute tray.

## OS profile design

Use Raspberry Pi OS Lite as the known bootable base. Lite already omits the desktop. The work is:

1. Audit enabled services and installed packages; remove optional parts and record the changes.
2. Preserve the boot chain, kernel and firmware, udev/systemd device handling, console video, USB, microSD, and required input drivers.
3. Add a small user-operable graphical profile with a touch launcher, terminal, file manager, and one editor. Add a browser only if measurements leave room.
4. Add feature profiles for camera, joystick controls, and removable-storage handling.
5. Measure boot-to-console, boot-to-launcher, idle RAM, page faults, memory pressure, swap activity, and power with each profile.

Removing the kernel's required device support does not make the system usefully modular; it can make a display or module disappear. Modularity should be expressed through optional packages/services and hardware interfaces while keeping the basic display and storage path working.

## OS-level work that fits the project

1. **Boot path:** document firmware and boot configuration, kernel image, device tree, kernel command line, PID 1/systemd, enabled services, and launch of the graphical session. Compare boot timing before and after one service change.
2. **Process and memory behavior:** use a small program that allocates and touches pages, then observe RSS, minor/major faults, `/proc/vmstat`, swap activity, and responsiveness under a controlled memory load. Explain demand paging and reclaim; package removal does not change paging itself.
3. **Device tree:** inspect `/proc/device-tree` and load a simple overlay only for an optional GPIO module. The USB serial module protocol needs no custom device-tree overlay.
4. **Custom kernel:** build one Raspberry Pi kernel with a separate filename and a unique local version. Change one configuration item, boot it from a backed-up development image, compare behavior, and retain the stock kernel for recovery. Raspberry Pi documents native and cross-compilation for the Zero 2 W and Pi 4B.
5. **Minimal root filesystem:** a BusyBox/initramfs boot experiment can show kernel-to-userspace handoff, but it is optional and separate from the user-operable Raspberry Pi OS Lite handheld profile.

Do not make a custom kernel the only boot option. The presentation should report which OS behavior changed and what measurements changed with it.

## Shared module protocol

One small Linux user-space broker can handle module discovery and common commands while ESP32 firmware owns sensor and actuator details. USB serial is the simplest v0 transport; select a microcontroller board with native USB CDC or an onboard USB-to-serial bridge. Linux enumerates the USB device first; after the broker opens its serial port, the ESP32 announces its ID and capabilities. The broker can map common joystick events to Linux input using `uinput`.

This reduces the need for per-module Linux protocol packages and polling services, but it does not remove the kernel, USB class driver, broker, or all module-specific code. Each ESP32 needs firmware, and new capability types need a host handler. Video, storage, display, and camera traffic stay on standard high-bandwidth interfaces.

## Browser and emulation

- A browser is optional in the baseline. NetSurf is available for Linux and targets constrained systems, but its official documentation describes incomplete JavaScript support; test the sites you need and skip it if compatibility or memory use is poor.
- mGBA publishes Linux ARM64 builds, making GBA the first retro-emulation target to try on the Zero 2 W. Installation availability does not guarantee full-speed gameplay, so measure a fixed game/test scene on both boards.
- Mupen64Plus supports ARM dynamic recompilation, but a complete setup also needs graphics, sound, input, and RSP plugins. N64 emulation should be a measured stretch on the Zero 2 W and a comparison workload on the Pi 4B.
- A console emulator does not translate the console's operating system into Linux. It models the console CPU and devices so original game code can run; Linux hosts the emulator process and handles real screen/input/audio. Booting a guest OS is a different, full-system-emulation experiment.
- The device's custom-emulator goal is guest CPU blocks translated into native ARM64 code plus emulated console devices. Use a limited ARM/Thumb-to-ARM64 JIT as the presentation prototype, with mGBA as the gameplay reference. A JIT is user-space software; Linux memory mappings, page protections, scheduling, and host device interfaces support it.
- QEMU system emulation models a virtual CPU, memory, and devices to boot a guest OS; QEMU user-mode emulation runs a foreign-architecture process under the existing host OS. Neither is a general-purpose OS-to-OS translator.

## Expansion interface rules

- Make USB the preferred hot-plug interface for user modules such as gamepad/joystick controllers and removable storage.
- Expose GPIO-based buses through a keyed, labeled connector with ground and a documented 3.3 V limit.
- Do not send 5 V into a Pi GPIO input. Use level shifting and current limits for modules with other voltage domains.
- Keep the CSI camera connector mechanically separate; its Zero-specific cable/adapter differs from the Pi 4B camera connector.
- Give each module a software profile: module identifier, driver/overlay if needed, package set, launcher entry, and removal procedure.

## Simulation and proof

- KiCad schematic checks and 3D view can show the carrier PCB, adapters, connectors, and enclosure placement.
- SPICE can model a selected regulator/power path under estimated loads.
- Neither KiCad nor QEMU proves USB hot-removal safety, camera operation, touchscreen compatibility, battery life, module protocol reliability, or full handheld performance. These need the physical boards and peripherals.

## Primary references

- Zero 2 W ports and specifications: https://www.raspberrypi.com/documentation/computers/zero.html
- Raspberry Pi OS Lite to desktop: https://www.raspberrypi.com/documentation/computers/os.html
- Touch Display 2 compatibility: https://www.raspberrypi.com/documentation/accessories/touch-display-2.html
- Camera software and Lite package: https://www.raspberrypi.com/documentation/accessories/camera-software.html
- USB mass-storage boot support and power considerations: https://www.raspberrypi.com/documentation/hardware/raspberrypi/pinout/getting-started.html
- GPIO 3.3 V limit: https://www.raspberrypi.com/documentation/computers/raspberry-pi.html
- Raspberry Pi kernel build/configuration: https://www.raspberrypi.com/documentation/computers/linux_kernel.html
- Linux memory management: https://www.kernel.org/doc/html/latest/admin-guide/mm/index.html
- NetSurf information and JavaScript limitations: https://www.netsurf-browser.org/documentation/info
- mGBA downloads: https://mgba.io/downloads.html
- Mupen64Plus: https://mupen64plus.org/
- QEMU system-emulation overview: https://www.qemu.org/docs/master/about/
