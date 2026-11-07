# Pocket Workstation OS — Project Brief

**Presentation:** 25 November 2026  
**Primary deliverable:** Raspberry Pi OS Lite derivative with integrated module service and SDK  
**Primary board:** Zero 2 W; replacement restricted to compatible Zero 2-family boards  
**OS baseline:** 64-bit Raspberry Pi OS Lite (ARM64/AArch64)  
**Future-board assumption:** a Zero 3 retains the connection method and supports 64-bit software; compatibility remains to be verified  
**Status:** proposed design and acceptance targets, not measured results

## Aim and user

Provide a student with a small Linux workstation where supported sensors and controllers become available to programs without choosing serial ports or installing separate sensor packages. Demonstrate editing/running code, reading a sensor, controlling an LED, and gaming. Hardware is the platform and example set for an OS design project.

Investigate how image configuration, device abstraction, and resource allocation affect usability on a 512 MB system. The Pi 4B is a separate optional comparison/build machine, not a chassis upgrade.

## Original work

- Versioned customization recipe, package manifest, service policy, and bootable image.
- Module service with discovery, registry, bounded IPC, lifecycle, and fault handling.
- Shared Python SDK and CLI; C/C++ client library is a later extension.
- MCU firmware using the capability contract and one physical module.
- Controlled resource-management experiment and repeatable measurements.
- Zero 2-family interface specification and handheld mock-up.

Keep upstream components attributed. A custom kernel is optional; the main OS contribution is the service/API and resource policy, beyond package selection or branding.

## Provisional acceptance targets

Freeze these engineering targets after the first baseline by 10 November. They are not guarantees. Record later revisions and reasons; do not silently redefine a target after seeing results. Measurement scopes are in [BENCHMARK_PLAN.md](BENCHMARK_PLAN.md).

| Requirement | Initial target | Evidence |
|---|---|---|
| Repeatable setup | Apply recipe to a clean recorded Lite base and pass the same smoke workflow twice | Logs, hashes, manifests, boot records |
| Offline discovery | Supported module visible within 3 s of kernel device-add | 20 attach trials; failures included |
| Host delivery latency | p95 at most 20 ms at selected 50 Hz stream, including controlled CPU load | At least 10,000 events per trial; broker-read to SDK-delivery only |
| Recovery | Removal visible within 3 s, stale reads rejected, reconnection works through 20 cycles | Physical unplug tests; no held virtual buttons |
| Broker restart | Attached module rediscovered within 5 s of service readiness; clients see restart/disconnect | Trace and new session ID |
| Broker footprint | At most 20 MiB idle RSS; at most 2% of one core averaged over 60 s at idle | Process measurements; SDK/client cost separate |
| Reliability | One-hour workload without crash, stuck state, or unbounded queues | RSS series, queue counts, errors and drops |
| Programming access | Bundled unprivileged Python environment discovers, reads, and controls offline | No per-module install; example demo |
| Device independence | Same program works with two implementations of one schema | Two devices, or physical device plus labeled mock |
| Core usability | Edit/run Python, build/run small C++ task, log sensor, safely eject data card | Checklist and task time |
| Gaming | At least one scene runs at intended emulated speed with usable controls/audio | Per-title/core/settings report |
| Hardware concept | Fit/interface matrix, power budget, full-size mock-up | Proposed versus checked dimensions labeled |

By 10 November, choose the screen/keyboard and record proposed size, mass, cost ceiling, and battery-runtime target. These depend on unselected parts and remain open design inputs. A bench supply is acceptable for the OS prototype; runtime is an estimate until the intended battery path is measured.

## Minimum presentation scope

1. Fresh boot of the customized image on the Zero 2 W.
2. One real MCU sensor/LED module through the packaged service, SDK, and CLI.
3. Attach, read, command, unplug, replug, and broker-restart demonstrations.
4. Same-board CPU-policy experiment with correctness and latency results; bounded memory observation supports the explanation.
5. Programming workflow and one tested game; extra emulators are optional.
6. Recipe/manifest, protocol/API docs, hardware diagram, power budget, and enclosure mock-up.

Prefer a second physical controller after the sensor path works. A mock supports protocol testing but does not establish USB, firmware, or electrical behavior.

## Conditional extensions

- Integrate the user-owned GBA emulator when its Linux build is ready.
- Add GB/GBC, NES, and selected Sega systems using existing cores, in [EMULATION_PLAN.md](EMULATION_PLAN.md) order.
- Add C/C++ SDK, editor discovery panel, or second physical module.
- Compare Pi 4B or another supported Zero 2-family board using the same ARM64 software baseline.
- Study zram, memory policy, or a purposeful custom kernel build.
- Complete detailed PCB layout, regulator simulation, or fabricated enclosure.

No new custom JIT, full emulator rewrite, N64 demo, or custom battery charger is required.

## Evidence and assessment

Separate measured, simulated, datasheet-derived, and proposed results. Keep commands, configuration diffs, inputs, versions, and raw measurements. Compare policies on the same board first. Report throughput losses alongside responsiveness improvements. Standard Linux with this service remains best-effort; do not claim hard real-time guarantees.

The course's fabrication rubric is unavailable. If a manufactured build is mandatory, revise that commitment and schedule before treating a mock-up as sufficient.
