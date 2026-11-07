# Pocket Workstation — Project Brief

**Presentation date:** 25 November 2026  
**Project type:** low-power handheld mini-PC concept with software prototypes and measured evaluation

## Project question

How much everyday computing can be made practical on a Raspberry Pi Zero 2 W when the operating system and interface are designed around its limits, and how should a user-upgradable handheld accommodate a more capable Raspberry Pi 4B?

## Project aim

Design a compact handheld computer for Python and C++ development, lightweight emulation, and basic desktop work. Treat the Pi Zero 2 W as the low-power baseline and the Pi 4B as a higher-performance replacement compute cartridge. Keep the screen, keyboard, pointing device, battery system, and expansion interface with the handheld so the compute board can be replaced without redesigning the whole device.

Start with Raspberry Pi OS Lite, which is already a command-line-only base. Audit its packages and services, remove optional software, and retain the boot, kernel, HDMI, USB, storage, input, and device-management support the handheld needs. Make a small touch-friendly desktop an optional user profile; a browser is optional and may be omitted from the Zero image. Use USB HID touch over an HDMI display because the Zero 2 W has no DSI connector.

This is a **finished design concept with working software demonstrations and measured results**, not a claim that the complete handheld or custom PCB has been manufactured.

## Learning objectives

- Understand Linux from firmware and boot configuration through the kernel, device tree, systemd, user processes, virtual memory, page faults, and swap.
- Build a minimal usable desktop from Raspberry Pi OS Lite and document the packages and services retained.
- Build and boot a custom Raspberry Pi kernel with one carefully selected configuration change while keeping the stock kernel as a fallback. Also compare one optional service change at user level.
- Design a versioned module protocol so ESP32-based modules can announce capabilities and exchange events with one small service on the main Pi.
- Compare a Pi Zero 2 W and Pi 4B using repeatable tasks, memory, latency, and measured energy.
- Design a protected battery and power path plus a documented expansion interface.
- Create a KiCad schematic and board layout, simulate a selected power circuit, and show the PCB in 3D.
- Design a custom console emulator around guest-to-ARM64 dynamic recompilation. Use a limited ARM/Thumb translator as the presentation prototype and mGBA for working GBA gameplay on the Zero 2 W; treat N64 as a stretch comparison.
- Explain how a console emulator models CPU instructions/devices for a Linux host, and how that differs from booting a guest OS in a full-system emulator.
- Build a compact CHIP-8 VM or limited RV32I interpreter only if it helps teach the JIT design or the console target slips; Arduino AVR simulation is another stretch task.
- Demonstrate Python and C++ development in a lightweight editor and terminal.

## Scope for the presentation

### Must be complete

1. Boot both boards into the same minimal desktop class and record the exact software setup.
2. Demonstrate Python and C++ editing/build/run using COL333 A1 and the COL106 Social Network program. Treat A3 and COL861 workloads as optional follow-ons.
3. Demonstrate daily Python and C++ work from the lightweight desktop; browser use is optional.
4. Present a controlled Pi Zero 2 W versus Pi 4B comparison, including a memory/page-fault experiment and measured energy.
5. Show the stock and custom kernels booting, explain the kernel configuration change, and compare boot/idle behavior. Also explain the firmware-to-user-space boot path and page-fault measurements.
6. Show the concept enclosure and powered-off compute-cartridge swap while preserving a handheld mini-PC form factor.
7. Show how a CSI camera, joystick, HDMI-plus-USB touchscreen, and one removable microSD data card connect. Keep the OS card installed separately and include a powered hub.
8. Demonstrate the module protocol with an ESP32 module or a software mock, including discovery, capability exchange, an event, and removal.
9. Run GBA gameplay with mGBA on the Zero 2 W if it reaches usable speed; show the custom ARM/Thumb-to-ARM64 JIT proof-of-concept separately. Record N64 performance as a stretch result on both boards.
10. Show a KiCad power/expansion design and a simulation or electrical calculation with assumptions stated.
11. Explain which tasks are comfortable, slow, or impractical on the Zero 2 W.

### Stretch goals

- Try NetSurf for static/light web pages after measuring the desktop baseline; omit it if it consumes too much memory or fails on required sites.
- Run a Nintendo 64 emulator on the Zero 2 W and compare it with the Pi 4B.
- Extend the custom emulator from its guest-instruction JIT proof-of-concept toward broader GBA hardware compatibility. A complete custom GBA or N64 emulator is follow-on work unless it is finished and tested.
- Add a device-tree overlay for a GPIO module if time allows.
- Use simavr or a comparable AVR simulator to compile and run a simple Arduino Uno sketch.
- Add a lightweight editor/launcher designed for the handheld. Keep an existing editor as a fallback.
- Measure the Pi 4B cartridge's power and thermal tradeoffs.
- Fabricate and test one expansion module after the concept PCB and interface pass review.

## Evidence rules

- Mark each result as measured on a board, simulated, inferred from a datasheet, or proposed.
- Do not equate FPGA simulated cycle counts with Raspberry Pi execution speed.
- Do not claim that removing desktop packages changes Linux paging mechanisms; study paging with measured memory-pressure and swap workloads.
- Treat a single host protocol as a way to centralize discovery and common commands, not as a replacement for Linux device drivers or high-bandwidth USB/CSI interfaces.
- Do not claim that a circuit simulation proves a finished board works.
- Report the tested Pi 4B RAM size, OS image, package set, storage, display resolution, power source, ambient conditions, and exact command for each benchmark.

## Proposed project claim

The project should establish a specific result, not assume one: a small, carefully configured Linux handheld can perform a useful subset of programming and emulation tasks on the Pi Zero 2 W, while the Pi 4B cartridge offers a clear upgrade path with higher power use. The measurements decide which tasks meet the usability target.

## Sources

- Raspberry Pi Zero 2 W hardware details: https://www.raspberrypi.com/documentation/computers/zero.html
- Raspberry Pi OS editions and Lite-to-desktop configuration: https://www.raspberrypi.com/documentation/computers/os.html
- Zero 2 W ports and camera interface: https://www.raspberrypi.com/documentation/computers/zero.html
- Official Zero 2 W touchscreen limitation: https://www.raspberrypi.com/documentation/accessories/touch-display-2.html
- Camera support in Raspberry Pi OS: https://www.raspberrypi.com/documentation/accessories/camera-software.html
- Raspberry Pi Linux kernel build and configuration: https://www.raspberrypi.com/documentation/computers/linux_kernel.html
- Linux memory-management documentation: https://www.kernel.org/doc/html/latest/admin-guide/mm/index.html
- mGBA ARM64 download: https://mgba.io/downloads.html
- Mupen64Plus ARM support and components: https://mupen64plus.org/
- NetSurf browser limitations and system support: https://www.netsurf-browser.org/documentation/info
- KiCad schematic, PCB, 3D viewer, and SPICE overview: https://docs.kicad.org/master/en/kicad/kicad.html
- QEMU Raspberry Pi board models and limitations: https://www.qemu.org/docs/master/system/arm/raspi.html
- simavr AVR simulator: https://github.com/buserror/simavr
