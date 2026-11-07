# Pocket Workstation OS — Benchmark Plan

## Purpose and controls

The primary comparison is **the same Zero 2 W running controlled OS configurations**. A Pi 4B comparison is secondary and does not prove that the customization improved the Zero. Use the acceptance targets in [PROJECT_BRIEF.md](PROJECT_BRIEF.md); all targets remain proposed until measured.

| Configuration | Purpose |
|---|---|
| L: recorded unmodified Lite | Baseline boot, idle footprint, and packages; no module API exists here |
| A: Lite + same editor/runtime + project service/SDK, default service policy | Functional baseline for the custom OS |
| B: A with documented service/package trimming | Isolate the effect of trimming while preserving the same workflow |
| C: B with CPU policy enabled | Isolate the resource-allocation change with identical packages/workloads |

Do not compare a bare console to a graphical session and attribute the whole difference to optimization. Compare A/B/C with identical display, applications, drivers, modules, swap configuration, and workload. L is a context baseline, not a module-latency comparator. Use the same 64-bit ARM64 image/ABI throughout; no legacy-architecture comparison is planned.

## Core measurements

1. **Image and boot:** recipe/base checksum, installed footprint, enabled units, boot-to-console and boot-to-ready. Record actual readiness of the API/editor, not just systemd target completion. Repeat cold boots separately from warm workload tests.
2. **Service and SDK overhead:** broker RSS/CPU, Python client RSS/CPU, and whole-system MemAvailable. Record no-module, connected-idle, and streaming conditions. Normalize CPU percentages to one core; do not add shared RSS blindly to infer total physical memory.
3. **Discovery and recovery:** 20 attach cycles, unplug during a request, broker restart while attached, and module reboot. Measure kernel device-add to usable registry entry. Include failures/timeouts in results.
4. **Resource policy:** equal versus selected CPU weights using the same cgroup hierarchy, a 50 Hz stream, and controlled background contention. Collect at least 10,000 events per trial. Report p50/p95/p99/max latency, missing events, queue high-water marks, and background throughput/completion time.
5. **Memory:** bounded page-touch workload with RSS, faults, MemAvailable, swap counters, task correctness, and responsiveness. Keep it separate from the CPU-policy comparison.
6. **Reliability:** one-hour run and malformed/flooded input tests. Track RSS over time, bounded buffers, rejected frames, reconnects, timeouts, stale handles, and virtual input cleanup.
7. **Useful programming:** edit/run selected Python, compile/run C++, then use the SDK to log a sensor and control an LED offline. Report time and usability, not just successful launch.
8. **Gaming:** one working selected title/test scene first, then expand according to [EMULATION_PLAN.md](EMULATION_PLAN.md). Record emulated speed, frame pacing, audio underruns, controls, save behavior, memory, temperature, and power.
9. **Energy:** whole-system idle and energy per completed workload, including display/hub/modules. Record meter resolution/sample rate and measurement boundary.

## Latency definitions

- **Host delivery latency:** broker's first read containing an event to its delivery to the SDK caller, using a shared host monotonic clock. This excludes MCU sampling, USB transport, and time queued before the broker reads; label it accordingly.
- **Host round-trip time:** host request submission to receipt of its matching module response, using one host clock. This includes bidirectional transport plus firmware work but is not one-way sensor latency.
- **Physical end-to-end latency:** optional button/signal edge to observable application response. Requires common-clock external instrumentation or a validated synchronization method. Do not subtract unsynchronized MCU and Pi clocks.
- **Input latency:** SDK delivery is not display response. An emulator's rendering/audio adds separate delays.

Use event sequence counters to expose drops; a fast percentile calculated only from surviving events can conceal overload. Report trace instrumentation overhead and distinguish a logging-enabled benchmark from normal operation.

## Test method

Record board revision, OS/ABI, kernel, CPU governor/clock, thermal state, storage, display resolution/brightness, Wi-Fi/Bluetooth, peripherals, and every relevant config. Keep voltage and cooling consistent. Monitor throttling and undervoltage.

Run at least five trials per runtime/policy condition after controlled warm-up. Alternate baseline/policy order where practical. Report trial medians/ranges and per-trial latency distributions; do not infer rare-tail guarantees from a tiny sample. Use fixed inputs and seeds for stochastic Python tasks and verify outputs before timing. Save failures as data.

For energy, integrate sampled input power over the task, or use a meter's energy counter with sufficient resolution. Instantaneous watts alone do not measure energy per task. A board-only reading excludes screen/hub power; label it. Battery runtime estimates use usable pack energy and conversion losses and remain estimates until tested.

## Existing workload review

The following notes are retained from the earlier read-only review; they have not been remeasured by this plan revision. Source folders:

- `/home/meesa/Desktop/COL333/A1`
- `/home/meesa/Desktop/COL333/A3`
- `/home/meesa/Desktop/Assignments/Old`
- `/home/meesa/Desktop/COL861/Project/Files`

## Familiar workload review

| Workload | What it exercises | Pi Zero 2 W | Pi 4B | Recommendation |
|---|---|---|---|---|
| COL333 A1, part_a.py / part_b.py | Python standard-library CSP/backtracking and local search; reads small CSV instances and emits schedule JSON | Strong candidate | Strong candidate | Primary Python coding benchmark. Use the same sample inputs, check output validity, then measure runtime and memory. |
| Assignments/Old/COL106/Social Network/main.cpp | C++ standard containers, graph relationships, sorting, queues, BFS shortest path, command parsing; includes Tree.hpp | Strong candidate | Strong candidate | Primary C++ compile-and-run workload. Compile from source on each board; do not use the existing host binary. |
| Assignments/Old/COL106/Git/Assignment.cpp | C++ hashing, file operations, data structures | Candidate | Candidate | Good second C++ task if a deterministic input set and build command are prepared. |
| Assignments/Old/COP290/Assign_1 | Multi-file Qt C++ drawing application with GUI | Constrained | Better fit | Useful as a qualitative GUI/editor task, but Qt dependencies and memory make it a poor first Zero benchmark. |
| COL333 A3 football agent | Large Python strategy code; Python 3.11, gRPC, pyrusgeom, simulator/proxy binaries, match runner | Agent code may run; full match stack not ready for ARM | Possibly after rebuilding stack | The project README specifies Linux x86_64 or Apple Silicon. Treat full matches as a stretch porting task, not a Zero 2 W benchmark. The repository is 4.1 GB largely due to match logs/replays and generated artifacts; copy only selected code and required runtime. |
| COL861 TinyStories integer reference | NumPy fixed-point inference | Candidate | Candidate | Optional compute-heavy Python workload; run only after A1 and C++ baseline. |
| COL861 small Icarus testbench | HDL simulation runtime | Possible | Good candidate | Optional simulator/toolchain workload; its time is simulator CPU time, not FPGA performance. |
| COL861 training, full suite, Verilator transformer, Vivado | ML training, synthesis, large simulations and generated outputs | Poor fit | Mixed/heavy | Do not make these core claims for this handheld. |

The A1 directory is about 13 MB and is a particularly clean, dependency-light Python task. The old assignments directory is about 352 MB including generated files and binaries; copy only the selected C++ source, headers, and inputs. The A3 directory is about 4.1 GB and is not a good thing to clone wholesale onto the handheld.

## Selected workloads and optional comparison

Use A1 Part A/B on small fixed cases with their verifier/checker. Use COL106 Social Network with a fixed command sequence covering graph operations and shortest path; report compilation separately from execution. Compile for the target rather than copying an existing desktop binary.

If time remains, repeat a subset of B/C and the same workloads on Pi 4B, recording its RAM and all configuration differences. Same-family board swaps require a separately tested compatibility matrix. Keep A3 porting, COL861, browser trials, additional kernels, and further emulator cores outside the minimum benchmark set.

## Result artifacts

Store commands, manifests, inputs/seeds, logs, and raw tabular results beside each chart. Report whether a target passed, failed, or was not tested. The project succeeds academically through a justified design and honest evidence; a policy showing no improvement is not a reason to conceal its results.
