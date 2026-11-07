# Pocket Workstation OS

A custom Raspberry Pi OS Lite–based system for Pi Zero 2-family handhelds, with an integrated module discovery service, shared programming libraries, and measured resource-management policies. Sensors and controllers demonstrate the OS interface; programming and gaming demonstrate its usefulness.

**Primary target:** Raspberry Pi Zero 2 W, using 64-bit Raspberry Pi OS Lite (ARM64). Compute replacement is limited to compatible Pi Zero 2-family boards, with power off. The Pi 4B is a separate build/comparison machine. The design assumes a future Zero 3 retains the same connection method and supports 64-bit software; physical fit, power, and kernel/firmware support will need validation. Original Zero/Zero W compatibility is outside scope.

## Current project documents

- [Final project concept](FINAL_PROJECT_CONCEPT.md) — consolidated OS-first proposal and academic contribution
- [Project brief](PROJECT_BRIEF.md) — scope, acceptance criteria, and evidence rules
- [Architecture](ARCHITECTURE.md) — Linux/service/library boundaries and Zero 2-family hardware
- [Module protocol](MODULE_PROTOCOL.md) — discovery, capability schemas, sessions, and failure handling
- [Module service and SDK](MODULE_SERVICE_AND_SDK.md) — packaged service, local API, editor workflow, and example code
- [OS study and image plan](OS_STUDY.md) — image customization, service audit, resource-management experiment, and recovery
- [Benchmark plan](BENCHMARK_PLAN.md) — same-board OS comparisons, responsiveness, energy, and useful workloads
- [Emulation plan](EMULATION_PLAN.md) — user-owned GBA work and additional retro systems
- [Hardware feasibility](HARDWARE_FEASIBILITY.md) — compatibility, interfaces, power, and physical design limits
- [Concept bill of materials](BOM.md) — available boards and unselected prototype components
- [Work plan](BUILD_GUIDE.md) — priorities and schedule through 25 November 2026

**Status, 8 November 2026:** design documents revised; the custom image, service, SDK, module firmware, measurements, and CAD remain to be implemented. Names, APIs, budgets, and examples are proposed contracts, not installed software. The user will handle the Arduino/GBA emulator work; OS development does not depend on reviewing or finishing it.

The former CM5 proposal remains [archived](archive/README.md) and is not a current wiring guide. Earlier workload-review notes are retained in the benchmark plan; this revision does not change those external projects.
