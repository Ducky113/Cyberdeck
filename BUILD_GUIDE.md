# Pocket Workstation OS — Work Plan

**Presentation:** 25 November 2026  
**Plan revised:** 8 November 2026  
**Priority:** working custom OS image, integrated service/SDK, and a controlled OS experiment

Implementation has not started in this repository. The schedule is a proposed sequence, not a record of completed milestones. The user handles the Arduino/GBA emulator work independently; the OS can demonstrate gaming with an existing core.

## 1. Freeze the base, boundaries, and targets

- Record a supported 64-bit ARM64 Lite image/checksum, Zero 2 W revision, storage, input, display, and power equipment.
- Keep compute interchangeability within tested Zero 2-family boards; Pi 4B is a separate machine.
- Select one real MCU board and scalar sensor/LED example. Confirm its USB mode and reset behavior.
- Freeze capability schemas, initial API, candidate-port matching, and timeout/queue policies.
- Establish a base manifest and initial acceptance baselines. Select screen/keyboard and propose size, mass, cost, and runtime targets.

**Done when:** the base boots, hardware inputs are recorded, and the project brief has frozen provisional targets.

## 2. Make the OS customization repeatable

- Preserve a known-good Lite image and implement an idempotent provisioning recipe.
- Package configuration, service, CLI, Python SDK, and examples as they become available.
- Keep a reasoned service audit; remove only unused components and preserve dependency/recovery paths.
- Add a small editor/terminal workflow first; add the measured graphical session for touch/dashboard/game use.
- Run the recipe on a fresh base and record package/configuration differences. Move to a pinned pi-gen stage once packages stabilize if time permits.

**Done when:** a fresh target can reproduce the development environment with manifests and recovery instructions.

## 3. Implement the common module path

- Build the frame parser and a serial mock for deterministic tests.
- Implement host-initiated handshake, indexed descriptors, identity/session tracking, and registry.
- Connect one physical MCU and demonstrate sensor readings and LED writes.
- Add bounded Unix-socket IPC, CLI discovery, and Python API examples.
- Integrate systemd startup/restart, permissions, startup enumeration, and offline operation.
- Validate timeout, module reboot, unplug, reconnect, broker restart, malformed frames, and slow clients.

**Done when:** an unprivileged editor-run program uses the module without serial setup or package installation, and loss/recovery is explicit.

## 4. Measure the OS design

- Capture L/A/B baselines as defined in [BENCHMARK_PLAN.md](BENCHMARK_PLAN.md).
- Verify CPU-controller support and set up the same sibling-cgroup hierarchy in both policy conditions.
- Compare equal weights against the selected policy under controlled CPU contention, reporting latency and background throughput.
- Perform a bounded memory/page-fault observation with memory policy unchanged.
- Run 20 reconnect cycles, restart/fault cases, and a one-hour reliability workload.
- Measure whole-system input power/energy where equipment permits; record instrumentation limits.

**Done when:** repeatable raw results explain what the design improves, what it costs, and where it fails.

## 5. Demonstrate usefulness and physical design

- Run selected Python and C++ edit/build/run tasks with correctness checks.
- Show sensor inspection/logging and LED control through the SDK.
- Test one retro game first. Add GB/GBC, NES, then selected Sega systems as time permits. Integrate the user's GBA build if ready.
- Prefer a second controller example with uinput; use a standard HID controller for the first game if needed.
- Draw the Zero 2-family hardware interface, power boundary, and USB topology.
- Build a full-size enclosure mock-up and check readable code, typing, cable access, and tabletop use.

**Done when:** one coherent demo connects boot, discovery, code, resource pressure, recovery, and practical applications.

## Proposed schedule and gates

| Date | Focus | Gate |
|---|---|---|
| 8–10 Nov | Base/parts/requirements; service audit; framing and API contract | Bootable base and targets fixed |
| 11–13 Nov | Broker + physical sensor/LED + CLI | Real discovery/read/write works |
| 14–16 Nov | SDK, packaging, permissions, recovery; fresh-image setup | Offline Python demo and lifecycle tests pass |
| 17–19 Nov | Resource-policy and memory experiments; reliability/energy | Raw results and bounded failure behavior |
| 20–21 Nov | Gaming, optional second module; interface drawing/mock-up | Useful end-to-end demo |
| 22–23 Nov | Repeat trials, verify fresh setup, charts and rehearsal | Claims trace to evidence |
| 24 Nov | Freeze artifacts/slides; save offline demo recording and recovery image | Recoverable presentation |
| 25 Nov | Presentation | Demonstrate measured system |

If the physical module path is not working by 13 November, stop adding capabilities and focus on one sensor/LED device. If packaging/recovery is not stable by 16 November, defer extra emulators, Pi 4 comparisons, C++ SDK, and all optional kernel work.

## What to cut first

1. Additional console cores, browser, and heavy workload ports.
2. Pi 4 benchmarking and validation of additional Zero 2-family boards or a future Zero 3.
3. C/C++ client library, editor plugin, and extra physical modules.
4. Kernel rebuild, zram comparison, custom PCB layout, regulator simulation, and fabrication.

Keep the fresh-image recipe, real module, Python SDK/CLI, lifecycle recovery, one controlled OS experiment, and basic useful demo. A mock alone is a contingency and must be labeled; it cannot replace the physical embedded claim. No new JIT or complete emulator rewrite is on the critical path.

## Presentation flow

1. Boot the customized image and show its version/manifest.
2. Attach a sensor, inspect its descriptor, and run a short SDK program.
3. Start background work, demonstrate the policy experiment, and show its tradeoff.
4. Unplug/replug and restart the service; show bounded errors and recovery.
5. Demonstrate programming and a game using normal controller input.
6. Explain the Zero 2-family interface, power budget, compatibility limits, and actual versus proposed hardware.

Every graph and feature must identify whether it is measured, simulated, datasheet-derived, or proposed. A concept enclosure suffices only if the course permits a design/software prototype; the fabrication rubric remains unconfirmed.
