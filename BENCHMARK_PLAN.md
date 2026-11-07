# Workload Review and Benchmark Plan

Reviewed read-only from these folders:

- /home/meesa/Desktop/COL333/A1
- /home/meesa/Desktop/COL333/A3
- /home/meesa/Desktop/Assignments/Old
- /home/meesa/Desktop/COL861/Project/Files

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

## Core benchmark set

1. **Python CSP task:** run A1 Part A and Part B on the same selected sample cases. Measure each part separately and check each output with the included verifier/checker. Avoid the checker's full default suite on the Zero until its runtime is known.
2. **C++ graph task:** compile the COL106 social-network program from source and feed it a fixed command sequence that exercises add-user, add-friend, friend suggestions, shortest path, and output. Record build time separately from run time.
3. **Everyday editor task:** edit one of those programs on the handheld, run it, inspect the output, and report the interaction experience. Keep this separate from raw compute timing.
4. **OS footprint/kernel:** compare standard and minimal user profiles, then stock and custom kernels at idle and during the same task. Record the kernel config diff, image size, boot timing, memory, and any behavior affected by the selected option.
5. **Memory/paging:** keep the same editor and task open, apply a controlled memory load, and record available RAM, swap-in/out, major faults, and responsiveness. Removing packages alone does not alter the kernel's paging implementation.
6. **Energy:** measure input power for boot/idle, the Python solve, the C++ compile, and the same GUI load using the same screen and hub peripherals.
7. **Module service:** measure daemon idle RSS, discovery time, and memory with one and several connected/mock modules. Record messages per second and CPU use for a joystick or sensor stream.
8. **GBA baseline:** run the same homebrew/test workload in mGBA at native resolution on the Zero 2 W and Pi 4B. Record frame rate, audio quality, CPU use, memory, and power.
9. **N64 stretch:** try one representative title on both boards at native resolution and report speed/compatibility as observed, without making N64 support a core pass condition.
10. **Custom JIT prototype:** compare the same guest instruction stream in the reference interpreter and translated ARM64 blocks. Check outputs/register state, then measure translation time, execution time, cache hits, RSS, and power.
11. **Browser optional:** compare desktop idle and a simple static page with NetSurf, then omit it if the benefit or site compatibility is poor.

## Optional follow-on tasks

- COL333 A3: first establish an ARM64-compatible simulator/proxy build; then run one short smoke match. Do not compare its strategy score as a CPU benchmark.
- COL861: run one small fixed-point inference or one small Icarus test. Do not use generated FPGA cycles as Pi wall time.
- Complete custom GBA/N64 hardware compatibility is beyond the presentation prototype; continue from the limited ARM/Thumb JIT after the core deliverables.
- simavr: try one basic Arduino AVR program if installation and memory use are acceptable.

## Fair measurement procedure

- Record each board's exact RAM size, OS release and architecture, SD card, desktop packages, screen resolution, and connected peripherals.
- Use the same OS release and architecture where supported. Prefer a 2x2 comparison: Pi Zero 2 W and Pi 4B each on standard desktop and lean desktop.
- Keep display brightness, Wi-Fi state, cooling, storage, and inputs fixed.
- Run at least five repetitions after warm-up; report median and range. For emulation, include a fixed scene or test ROM and the same emulator settings.
- Record wall time, peak resident memory, input power, temperature, and CPU clock during longer loads.
- Run correctness checks first; preserve exact commands and output artifacts.
- Report concrete tradeoffs. Do not claim a general Zero 2 W speed advantage based on comparing one lean configuration against a heavier Pi 4B setup.
