# Pocket Workstation — Work Plan

**Target presentation:** 25 November 2026  
**Current plan:** complete the concept, working software demonstrations, measured comparison, and design artifacts by the presentation. Manufacturing the final handheld is not a deadline requirement.

## Work packages

### 1. Freeze the scope and measurements

- Use the Zero 2 W as the low-power baseline and Pi 4B as the compute upgrade.
- Record Pi 4B RAM capacity, OS release and architecture, SD card, screen resolution, and power meter.
- Use COL333 A1 Part A/B as the Python task and COL106 Social Network as the C++ task. Keep one small COL861 workload optional.
- Pick one memory/paging experiment and one energy measurement.
- Decide whether the optional browser and N64 experiment fit after the base workload runs.
- Confirm the minimum editor/desktop setup and the concept handheld dimensions.

**Complete when:** the task list and comparison method fit on one page.

### 2. Build the minimal usable desktop

- Start from Raspberry Pi OS Lite.
- Audit Lite first; retain the kernel, firmware, HDMI console/display, USB, microSD, input, and device management. Remove optional services/packages, then add only the small graphical session, USB touch support, launcher, editor, and tools needed for the chosen tasks.
- Leave the browser out initially. If the core profile has memory headroom, try NetSurf on a simple static page and record compatibility and memory use.
- Keep a record of installed packages, enabled services, boot-to-editor time, idle RAM, and SD image size.
- Use a low-resolution desktop appropriate for the handheld screen.
- Test the same workflow on the Pi 4B.

**Complete when:** both boards boot to the editor, run the selected Python and C++ tasks, and have a saved configuration record.

### 3. Test workloads and daily-use flow

- Run the same COL333 A1 Python inputs and COL106 Social Network C++ command sequence on both boards. Build the C++ source on each board rather than copying a host binary.
- If the core work is complete, try mGBA with a fixed GBA workload on both boards. Treat N64 on the Zero as a stretch; compare a representative title with the Pi 4B and record actual speed and compatibility.
- If time allows, try a NumPy-only TinyStories integer inference or a small Icarus unit simulation from COL861. Keep PyTorch and the A3 full-match stack as stretch work only.
- Do not use simulated FPGA cycles as Pi CPU performance.
- Build a limited guest ARM/Thumb-to-ARM64 dynamic-recompiler proof-of-concept, with a reference interpreter and instruction-state checks. Use mGBA for the working GBA gameplay demonstration; do not present a partial translator as a complete GBA emulator.
- Attempt a simple AVR/Arduino simulation only after the core tasks are complete.

**Complete when:** each task has a status, command, wall time, memory reading, and short usability note.

### 4. Design the modular handheld

- Draw the block diagram for a common carrier and two board-specific compute adapters.
- Define a keyed expansion connector and its power/data pin groups. Mark USB modules as possible live-plug devices; plan CSI and raw GPIO module changes with power off.
- Write the module protocol v0 and implement a host daemon plus one ESP32 module or mock. Demonstrate attach, capability negotiation, one command/event, and removal.
- Show the USB topology: HDMI video, USB HID touch, a powered hub, one USB microSD reader, and the module port. Keep the boot microSD installed; demonstrate safe unmount before removing the data card.
- Trace boot to systemd, build/boot one custom kernel configuration, and compare it with the stock kernel. Keep the stock kernel available as fallback. Also measure one optional service change and the paging lab.
- Draw the power path with the battery cartridge, protection, charging, regulator, and loads.
- Create the KiCad schematic and a PCB layout with labeled connectors and board mounting points.
- Simulate the chosen regulator/load circuit in SPICE, or present a reasoned power budget if a usable component model is unavailable.
- Create the enclosure mock-up with the handheld and tabletop mini-PC modes.

**Complete when:** the concept can be explained from battery through compute, display, input, and expansion, and the evidence labels identify design versus simulation.

### 5. Benchmark and prepare presentation

- Run each benchmark several times after a clean boot; report median and spread.
- Capture idle and task power with the same display and peripherals.
- Record temperature and CPU clock during longer tasks to disclose throttling.
- Use a 2x2 comparison where feasible: Pi Zero 2 W and Pi 4B, each with standard desktop and lean desktop configurations. If the standard desktop is unusable on the Zero, label that result rather than changing the comparison silently.
- Present the specific metrics where the lean setup helps, plus the tasks where the Pi 4B remains faster or more capable.
- Keep screenshots, photos, measurements, source commands, and CAD/KiCad renders together.

**Complete when:** every claim on the slides points to a measured result, a simulation, a design drawing, or an explicitly labeled proposal.

## Suggested schedule

| Date | Focus |
|---|---|
| 7–10 Nov | Workload triage, OS inventory, protocol v0 and scope lock |
| 11–15 Nov | Minimal desktop, Python/C++ demos, custom kernel build, start guest-to-ARM64 JIT and first measurements |
| 16–18 Nov | ESP32 module demo, boot custom kernel on target, JIT correctness demo, GBA baseline test; N64 only if time remains |
| 19–21 Nov | KiCad power/expansion design, protocol diagram and enclosure renders |
| 22–23 Nov | Repeat measurements, create result charts and rehearse demo |
| 24 Nov | Freeze slides and evidence, prepare offline fallback |
| 25 Nov | Presentation |

## Risk controls

- The Zero 2 W has only 512 MB RAM; treat full IDEs, model training, and heavyweight browser use as poor fits. Browser support is optional.
- Do not expect the shared protocol to eliminate all module-specific logic: each ESP32 needs firmware, and new capability types may need a host UI handler.
- N64 performance on the Zero 2 W is uncertain; make GBA the main retro-emulation target and report measured N64 results as a stretch.
- Do not claim safe battery charging or hot-swap behavior from a schematic alone.
- Keep large training runs, Vivado synthesis, and full repository copies off the Zero 2 W.
- If time shrinks, keep: the working desktop, Python and C++ tasks, OS memory/kernel study, limited guest-to-ARM64 JIT demo, module-protocol mock, Pi 4B comparison, power/expansion schematic, and enclosure render. Drop COL861, browser, N64, and a complete custom GBA model first.
