# Pocket Workstation OS — Final Project Concept

**Presentation:** 25 November 2026  
**Primary target:** Raspberry Pi Zero 2 W  
**Compute interchangeability:** compatible Pi Zero 2-family boards, powered off  
**Pi 4B:** separate build and comparison machine  
**Status:** proposed design; implementation and measurements pending

## Project in one paragraph

Pocket Workstation OS is a customized Raspberry Pi OS Lite–based Linux system for a small, modular handheld. It contains the services and applications needed for selected workflows, plus a project-owned module service and programming SDK. When a supported USB module is connected, Linux enumerates it, the service negotiates its capabilities, and applications can use them through a stable API without choosing serial ports or installing a sensor-specific package. Hardware modules demonstrate this OS interface. Editing programs, collecting sensor data, controlling outputs, and playing retro games demonstrate practical use.

## Central design question

> How can a resource-constrained Linux system make supported embedded modules available through a consistent programming interface while maintaining responsive interaction under competing CPU and memory workloads?

The intended user is a student who wants to connect a sensor or controller, inspect it, and use it in a short program on a self-contained device. The handheld also provides offline programming and gaming. Evaluate these concrete workflows before making a general laptop-replacement claim.

## What makes the OS our own

Raspberry Pi OS Lite is the base distribution and image, rather than one monolithic source-code project. The deliverable is a named derivative with a repeatable customization recipe, package/service manifest, lightweight user environment, original service and SDK packages, and measured resource-allocation policies. Upstream Linux, firmware, and drivers remain identifiable components.

The project owns:

1. The selected package set, service policy, boot configuration, and image recipe.
2. `pocket-moduled`, a systemd-managed service for supported module discovery and lifecycle handling.
3. A versioned capability contract, local application API, and bundled Python SDK and CLI.
4. A tested resource-management policy and experiments explaining its tradeoffs.
5. Documentation, example applications, recovery instructions, and a compatible handheld design.

A custom kernel is optional when a specific experiment needs it. A reliable system service, device abstraction, IPC, permissions, scheduling policy, and failure recovery are substantive OS work. They do not require moving sensor logic into the kernel. The image approach is detailed in [OS_STUDY.md](OS_STUDY.md); Raspberry Pi's [pi-gen](https://github.com/RPi-Distro/pi-gen) supports custom Raspberry Pi OS images.

## Discovery and application architecture

```text
Sensor / buttons / LED
        ↕ device-specific firmware
ESP32 or another supported MCU
        ↕ USB serial + versioned capability protocol
Linux USB/class driver → device events
        ↕
pocket-moduled (systemd service)
  discovery · sessions · registry · validation · recovery
        ├── local IPC → bundled Python SDK / CLI → user programs
        └── input adapter → uinput → normal Linux controller input
```

The image preinstalls handlers for scalar readings, button state, joystick axes, and LED output. A module advertises instances of these types with units, ranges, and stable identifiers. Supported capabilities become available offline, without per-sensor installation. A dashboard can show them immediately; an editor can run code importing the SDK.

The service does not infer arbitrary electronics from a cable. Firmware must identify capabilities and the host must understand their schema. A new sensor using an existing type can reuse the application interface; a fundamentally new capability needs a service/SDK update. Editors do not automatically understand physical devices: bundled examples, Python typing information, and a discovery command provide the integration.

Use existing Linux interfaces where appropriate: input through `uinput`, cameras through their camera stack, storage through filesystems, and audio through the selected audio stack. Generic sensors initially live in the project API; they are not automatically kernel IIO or hwmon devices. Linux supports [userspace input-device creation](https://www.kernel.org/doc/html/latest/input/uinput.html).

## OS profiles and efficiency

- **Base:** boot, storage, USB, device management, module service, CLI, and diagnostics.
- **Development:** base plus Python, C/C++ tools, small editor/terminal, offline API help, and an optional lightweight graphical session.
- **Gaming:** selected emulator packages and controller mapping, launched on demand from the same installation.

Profiles are package/service choices, not separate operating systems. Installed libraries occupy storage without requiring a resident process. The broker is resident; dashboards and emulators run when requested. Retain networking, Bluetooth, audio, and graphics where the workflow needs them. Record a reason for every disabled service.

The principal experiment compares module responsiveness under background CPU load with and without a cgroup CPU-allocation policy. A bounded memory-pressure experiment explains faults, reclaim, and process survival. Zram or memory-controller policies are follow-on comparisons, not simultaneous changes. See [OS_STUDY.md](OS_STUDY.md) and [BENCHMARK_PLAN.md](BENCHMARK_PLAN.md).

## Hardware role and interchangeability

The enclosure is sized around the Zero 2 family. Screen, keyboard, USB hub, module ports, and regulated power remain with the chassis when a compatible Zero 2-family board is replaced with power off. Validate revision, connectors, GPIO header height, mounting, cooling clearance, and power. The Pi 4B stays outside this mechanical interface.

Use a recorded **64-bit Raspberry Pi OS Lite (ARM64/AArch64)** base, validated on the available Zero 2 W. The supported hardware scope is the Pi Zero 2 family. The forward-looking design assumes a future Pi Zero 3 would retain the same connection method and support 64-bit software. This is a project assumption, not a verified specification: confirm connector/mechanical compatibility, power requirements, and kernel/firmware support before declaring a future board supported. Original Zero/Zero W support is outside scope. [Raspberry Pi OS architecture guidance](https://www.raspberrypi.com/documentation/computers/os.html).

Use HDMI video and USB touch, an internal OS microSD, and one external USB reader for user files. USB modules are the live-plug examples. CSI and raw GPIO/I2C/SPI changes remain powered-off operations. Standard cables and a powered hub suffice for the bench prototype. Hardware deliverables are an interface schematic, power budget, and full-size enclosure mock-up; custom carrier fabrication and battery electronics are later work.

## Emulation and useful applications

Gaming is an application and input/latency workload. The user will handle the Arduino/GBA emulator project. Define a Linux application boundary for display, audio, input, files, and timing; its internal implementation and portability are not assumed. Use mGBA as an independent gameplay baseline if that integration is not ready.

Expand gaming coverage with **Game Boy/Game Boy Color and NES**, then **Master System/Mega Drive** after measurement. Candidate implementations and acceptance rules are in [EMULATION_PLAN.md](EMULATION_PLAN.md). Supporting a console does not establish a particular game's speed. N64 is removed from the active target list.

The previous phrase “small guest-to-ARM64 JIT” meant writing a dynamic translator: decode a few guest arithmetic, load/store, and branch instructions, generate native AArch64 blocks, and check results against an interpreter. That would not constitute a complete console emulator. It is no longer required and is separate from porting an Arduino emulator. The OS and application builds target ARM64; the module protocol and public SDK remain independent of CPU instruction encoding.

Everyday applications include a sensor inspector/logger, a short Python program using a newly attached sensor, C++ build/run, safe removable-file access, and a controller-driven game. Browser support, heavy IDEs, ML training, and large simulator stacks remain outside the core.

## Academic contribution and evidence

| Design decision | OS / embedded learning | Evidence |
|---|---|---|
| Minimal derivative image | Boot, dependencies, service lifecycle, repeatable configuration | Base checksum, recipe, manifest diff, fresh boot |
| USB discovery to registry | Kernel/userspace boundary, enumeration, identity, versioning | Attach/restart/unplug traces; descriptor validation |
| Shared SDK and IPC | Device abstraction, multi-client access, permissions, errors | Same program with two compatible devices or a labeled mock |
| MCU-local hardware handling | Sampling, debounce, device drivers, host/MCU division | Firmware and one physical sensor/LED module |
| Resource policy | Scheduling, cgroups, memory pressure, fairness | Same-board comparison, including background-task cost |
| Removal and restart | Fault isolation, bounded queues, stale handles, recovery | Failure injection and repeated reconnect cycles |
| Useful workflows | Interface design and validation | Editing, logging, compile/run, gaming |
| Zero 2-family hardware | Maintainability, mechanical interfaces, power limits | Compatibility matrix, mock-up, power measurements |

Do not equate fewer packages with lower idle RAM, lower watts with lower energy per task, or service restart with preservation of device state. Each claim needs its own measurement.

## Presentation commitment

1. Bootable customized Zero 2 W image, repeatable recipe, and service/package manifest.
2. `pocket-moduled`, CLI, and Python SDK bundled into the image.
3. One real sensor/LED MCU module, with discovery, readings, output control, unplug, and reconnect working offline.
4. A second compatible firmware example or labeled mock showing that application code does not change; a physical controller is the preferred extension.
5. Controlled OS policy experiment, reliability tests, and whole-system power measurements where instrumentation permits.
6. Programming workflow and at least one working retro-game demonstration; the GBA integration and additional systems are gated by time/results.
7. Zero 2-family interface/power schematic, compatibility record, and enclosure mock-up.

The [project brief](PROJECT_BRIEF.md) owns provisional acceptance targets. The [work plan](BUILD_GUIDE.md) makes Pi 4 comparisons, extra emulators, custom kernels, and elaborate hardware conditional on the OS service and experiment working first.

## Presentation claim

Pocket Workstation OS is a Raspberry Pi OS Lite derivative that makes supported embedded modules available through a consistent application API. Its evaluation shows the cost of that abstraction and how a deliberate OS resource policy affects responsiveness on a memory-constrained handheld. Hardware modules, programming tools, and games demonstrate the system's use; measured results determine its limits.
