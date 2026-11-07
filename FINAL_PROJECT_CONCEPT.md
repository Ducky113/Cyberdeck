# Pocket Workstation — Final Project Concept

**Presentation:** 25 November 2026  
**Primary computer:** Raspberry Pi Zero 2 W  
**Upgrade/comparison computer:** Raspberry Pi 4 Model B  
**Project form:** handheld computer concept, Pi OS/kernel work, a modular-device protocol, and an emulation research prototype

## Project in one paragraph

Pocket Workstation is a low-power handheld mini-PC centered on the Raspberry Pi Zero 2 W. It starts from Raspberry Pi OS Lite, preserves a usable HDMI/keyboard system, adds a small optional touch desktop, and lets a user add USB-connected ESP32 modules through one documented protocol. The project studies how Linux boots, discovers devices, manages memory, and schedules programs. Its longer-term gaming goal is a custom emulator that dynamically translates console CPU instruction blocks into native ARM64 code while emulating the console's hardware. The Pi 4B is the faster, higher-power upgrade and comparison board.

## The central emulation idea

The translation you describe is **dynamic recompilation**, also called a JIT (just-in-time) emulator. A game contains instructions for its original console CPU. The emulator decodes guest instructions, translates blocks into native ARM64 instructions for the Pi, caches those translated blocks, and executes them on the Cortex-A53 or Cortex-A72. QEMU's TCG documentation describes this same guest-to-host translation model and its translation-block cache. [QEMU TCG internals](https://www.qemu.org/docs/master/devel/tcg.html)

The emulator still has to model the target console around that CPU. A game can read or write memory-mapped registers for graphics, audio, timers, interrupts, DMA, controllers, save hardware, and other devices. Those accesses must produce behavior and timing the game expects. Fast paths can handle ordinary RAM; accesses to hardware registers go through device-emulation code. Generated code must also be invalidated if the guest changes instructions that have already been translated.

The Pi's Linux system does not translate the console's OS into Linux. Linux runs the emulator as a native process and provides its memory, scheduler, input, display, and audio interfaces. The emulator presents the console CPU and devices to the game. QEMU full-system emulation is the separate technique for modelling a whole virtual computer so a guest OS can boot. [QEMU system emulation](https://www.qemu.org/docs/master/about/)

## Academic framing: what each step teaches

The project is strongest for a Computer Architecture and Operating Systems professor when each feature answers a technical question and produces evidence. Its central question is:

> How do the Linux runtime, device abstraction, and guest-to-host instruction translation affect the performance, memory use, and energy cost of useful computing on a low-power ARM handheld?

| Project step | Computer architecture learning | Operating systems learning | Evidence to show |
|---|---|---|---|
| Define the system and its interfaces | Separate CPU, memory, storage, display, input, power, and peripheral paths; reason about bandwidth and bottlenecks | Define which work belongs in firmware, kernel drivers, user-space services, or applications | Block diagram, bus map, and a table of design limits |
| Establish a clean baseline | Distinguish instruction-set architecture from implementation and compare execution cost | Identify the stock OS, services, RAM use, and boot behavior before tuning | Board model/RAM/OS record, boot time, idle RAM, task time, and input power |
| Start from Raspberry Pi OS Lite | Understand how software is hosted on a fixed CPU/memory platform | Trace firmware → kernel/device tree → PID 1/systemd → console/desktop; remove optional user-space services without removing required device support | Boot-flow diagram, package/service diff, and before/after measurements |
| Study virtual memory and build a kernel | Observe how loads/stores interact with page size, TLBs, and physical memory pressure | Relate virtual addresses, minor/major faults, demand paging, reclaim, swap, and executable-page permissions to measured behavior | Small page-touch program, `/proc/vmstat`/RSS/fault results, kernel configuration diff, and a custom kernel boot with stock fallback |
| Add the ESP32 module protocol | Separate MCU-local sampling/actuation from host computation; model events and data rates | Follow USB enumeration through the kernel/udev into one user-space broker; compare a standard class driver with a custom capability protocol | Protocol schema, attach/HELLO/capability/event/unplug trace, broker RSS and event latency |
| Build the custom JIT prototype | Decode guest opcodes, preserve register/flag semantics, generate AArch64, cache translated blocks, model branches and memory-mapped devices | Use `mmap`/page permissions for a code cache, observe faults, and use Linux scheduling and host I/O without moving game execution into the kernel | Reference-interpreter equivalence tests, generated guest→host mapping, translation/cache metrics, and timing |
| Run familiar Python/C++ work and game emulation | Compare CPU-bound, memory-sensitive, and graphics/device-heavy workloads | Observe the real app and emulator as Linux processes competing for memory, CPU, storage, and I/O | Correct outputs plus runtime, RSS, temperature, frame/audio behavior, and power |
| Design the carrier and power path | Connect processor/peripheral capabilities to real buses, electrical limits, and system bottlenecks | Show how the OS expects the display, storage, input, and modules to appear | KiCad schematic/layout/3D, interface table, power budget, and clearly labelled SPICE results |

The Zero 2 W's Cortex-A53 and Pi 4B's Cortex-A72 are different CPU implementations in the ARM family. A useful comparison keeps the host ABI and compiler target common, then measures the same workload on both boards. That helps separate code and OS differences from microarchitecture, clock, memory capacity, thermal, and power differences. Record those remaining variables instead of calling it a perfectly controlled CPU-only experiment. [Zero 2 W hardware](https://www.raspberrypi.com/documentation/computers/zero.html), [Pi 4 hardware](https://www.raspberrypi.com/documentation/computers/raspberry-pi.html)

The key distinction to explain in the presentation is:

- **ISA:** the instructions visible to software, such as guest ARM/Thumb, guest MIPS, or host AArch64.
- **Microarchitecture:** how a specific core implements those instructions, caches, pipelines, and memory accesses.
- **Operating system:** how processes receive address spaces, CPU time, files, devices, and permissions.
- **Emulator:** user-space software that models guest CPU/devices and translates guest code for the host.

The kernel experiment teaches OS internals and measurement. The JIT teaches ISA interpretation/translation and host execution. They meet at virtual memory, scheduling, and device I/O, but a custom kernel is not itself what makes guest instructions native.

## Emulation architecture

```text
Console game image
      ↓
ROM loader and memory map
      ↓
Guest CPU decoder → intermediate operations → ARM64 code cache/JIT
      ↓                                      ↓
Guest memory and device model ← CPU state and timing
      ↓
Graphics / audio / input / save-state backends
      ↓
Linux display, sound, input, files, and process services
```

Build the engine in layers:

1. **Correct reference interpreter:** implement a small subset of a guest ISA and verify register, flag, branch, and memory behavior against test vectors.
2. **Intermediate representation:** lower guest instructions into a small set of internal operations shared by the interpreter and code generator.
3. **ARM64 JIT:** translate basic blocks, cache them by guest address and CPU state, and fall back to the interpreter for unsupported or exceptional cases.
4. **Memory/device bus:** distinguish ordinary RAM from memory-mapped device registers. Keep device side effects in helpers and preserve guest-visible timing.
5. **Console subsystems:** add graphics, audio, timers, interrupts, DMA, controller input, save behavior, and the console's firmware calls used by the target games.
6. **Linux integration:** launch through the handheld's gaming profile, map controls, render at the panel's native resolution, and keep emulator files/configuration on the OS or removable data card as appropriate.

Optimizations to study include translation-block reuse, direct chaining between blocks, fewer interpreter dispatches, preallocated buffers, event-driven input, efficient RAM access, and a lightweight display/audio path. Profile before optimizing. A smaller desktop can reduce idle memory and competition for CPU time, but it does not replace guest-device emulation or guarantee faster games. Linux kernel changes can reduce system overhead or improve observability; the largest emulator speedups usually come from correct, efficient translation and device models.

Choose a 64-bit Raspberry Pi OS userland for the ARM64 JIT prototype, and record the host ABI used by generated code. The JIT itself is a normal Linux user-space process: it allocates a code cache, changes page permissions as needed, synchronizes generated instructions with the CPU instruction cache, and executes them. This is a useful point of contact with virtual memory and page protection, but the JIT does not require a custom kernel.

## Console targets and scope

### Game Boy Advance — first custom-console target

The GBA is the more sensible first target for a custom emulator. Its guest CPU is in the ARM family, but a GBA game is not automatically native Pi code: the emulator must reproduce guest CPU state, instruction behavior, memory mapping, graphics, audio, interrupts, and timing. mGBA publishes ARM64 Linux builds, so it provides a practical reference emulator for measuring what the Zero 2 W can run while the custom core develops. [mGBA downloads](https://mgba.io/downloads.html)

The project goal is eventually to boot a GBA game image directly in the device's custom emulator. For the 25 November presentation, use mGBA as the working gameplay baseline and aim for a custom translation proof-of-concept that executes a small ARM/Thumb instruction subset or test ROM. A complete, compatible, polished GBA JIT is a substantially larger follow-on project.

### Nintendo 64 — advanced stretch target

N64 is a longer-term target. It needs a MIPS CPU model and additional graphics and signal-processing emulation, plus synchronized input and audio. Mupen64Plus documents ARM dynamic recompilation, while its complete setup also depends on video, audio, input, and RSP components. Use it as a performance comparison on the Pi 4B and an experiment on the Zero 2 W; do not make full-speed N64 on the Zero a success criterion. [Mupen64Plus](https://mupen64plus.org/), [Mupen64Plus core requirements](https://github.com/mupen64plus/mupen64plus-core/blob/master/README.md)

For a small emulator that you can finish and understand by the presentation, implement a tiny interpreter such as CHIP-8 or a limited RV32I CPU separately. That demonstrates instruction fetch/decode/execute and memory without pretending to be a full GBA or N64 emulator.

## Device hardware and modularity

| Part | Concept |
|---|---|
| Compute | Zero 2 W low-power cartridge; Pi 4B larger/faster cartridge. Change boards with power off and board-specific adapters. |
| Display | HDMI video plus a separate USB HID touchscreen controller. The Zero 2 W has no DSI; keep the screen path common across boards. |
| Input | Compact keyboard plus touch and/or a USB gamepad module. A GUI is needed for practical touch operation; use a keyboard for the bare console profile. |
| Storage | One microSD stays installed for the OS. One exposed microSD slot uses a USB reader for removable files. Unmount/eject before removing it. |
| Expansion | Powered USB hub connects touch, the removable-card reader, keyboard if wired, and USB serial ESP32 modules. |
| Camera | CSI camera on the correct board-specific flex connector; power down before changing the ribbon. |
| Power | Replaceable battery concept, protected cell/charging path, and regulated supply sized from measured Zero/Pi4, screen, and hub loads. |
| Enclosure | CAD/mock-up shows handheld and tabletop mini-PC positions, screen, keyboard, card access, expansion bay, battery, heat path, and the two compute trays. |

The official Raspberry Pi Touch Display 2 uses DSI and does not connect to the Zero 2 W. Use an HDMI panel with USB touch instead. [Touch Display 2 compatibility](https://www.raspberrypi.com/documentation/accessories/touch-display-2.html)

USB serial modules are the main hot-plug expansion type. Treat CSI and raw GPIO/I2C/SPI modules as powered-off changes unless their electrical design explicitly supports live insertion. The compute board itself is also a powered-off swap.

## Shared ESP32 module protocol

One small Linux broker handles discovery, capability negotiation, commands, and events. The ESP32 firmware handles the hardware-specific details. Use USB serial for the first version: Linux enumerates the device, the broker opens its serial port, then the ESP32 sends its `HELLO` and capability list. A joystick module can send changed axis/button events; the broker can expose them as Linux input. The complete proposal is in [MODULE_PROTOCOL.md](MODULE_PROTOCOL.md).

This design avoids separate Linux packages, services, and protocol stacks for each simple sensor or controller. It will not eliminate all module-specific code: every ESP32 needs firmware, and novel capabilities may need a host handler. The protocol is for small control/sensor messages; keep camera video, storage blocks, display pixels, and ordinary USB HID on their standard interfaces. Measure the broker's memory and CPU use before claiming a resource saving.

## Linux OS and kernel work

Raspberry Pi OS Lite is already a command-line system. The project will use a recorded 64-bit release for the shared ARM64 development/emulation path, audit it, remove optional applications/services, retain the kernel/firmware, HDMI, USB, storage, input, and device management needed by the handheld, then add a small desktop profile with a touch launcher, terminal, file manager, and editor. Leave the browser out at first. NetSurf is an optional constrained-system browser, but its documentation notes incomplete JavaScript support, so it may not work with modern sites. [NetSurf information](https://www.netsurf-browser.org/documentation/info)

The OS study will cover:

- boot firmware/configuration → Linux kernel/device tree/command line → initramfs/root filesystem → PID 1/systemd → console or graphical session;
- device discovery from USB enumeration and kernel drivers through udev and the module broker;
- one measured service/package change;
- a small memory program that allocates and touches pages, with RSS, minor/major page faults, `/proc/vmstat`, swap, time, and responsiveness recorded;
- building and booting a custom Raspberry Pi kernel with one carefully chosen configuration change and a unique version, keeping the stock kernel available as fallback;
- optional GPIO device-tree overlay or BusyBox/initramfs demonstration if time allows.

Build the kernel on the Pi 4B or cross-compile on another Linux machine, then boot it on the target board. Raspberry Pi documents kernel configuration and native/cross builds for the Zero 2 W and Pi 4B. [Raspberry Pi kernel documentation](https://www.raspberrypi.com/documentation/computers/linux_kernel.html). The custom kernel is for OS learning and controlled measurement; do not remove support required for display, boot, USB, storage, networking, or input.

## Everyday work and benchmarks

Use the assignments you already know:

- **Python:** COL333 A1 scheduler (`part_a.py` and `part_b.py`), standard library, with its included verifier/checker. Run selected cases first.
- **C++:** COL106 Social Network graph program. Compile from source on both boards and use a fixed sequence covering graph operations and shortest path.
- **Optional heavy task:** COL333 A3 requires porting its native simulator/proxy stack to ARM; COL861 training and full generated suites are not Zero 2 W baseline tasks.
- **GBA:** compare mGBA on both boards with fixed settings/scene; record frame rate, audio, CPU, memory, temperature, and power.
- **N64:** stretch comparison; record the actual title/settings/results.
- **Custom JIT:** record guest instructions/block count, translation time, execution time, cache hit rate, correctness against the interpreter, memory, and power for a small test program.
- **System:** compare stock and minimal user profiles and stock/custom kernel; measure boot time, idle RAM, workload RSS, page faults, input power, and temperature.
- **Module broker:** record discovery time, command/event latency, CPU use, RSS, and behavior after hot unplug.

Use the same tasks and peripherals on both boards where possible. A lean Zero 2 W may win on idle power or memory footprint; that does not establish that it is faster than the Pi 4B. Report the metric that was actually measured.

## Simulation and evidence

- **KiCad:** carrier schematic, connector mapping, PCB layout, 3D board view, and board-specific compute adapters.
- **SPICE:** a selected regulator/power-path model with stated assumptions and estimated loads.
- **CAD:** enclosure mock-up and component placement.
- **QEMU:** optional CPU/guest-OS learning. A virtual-machine model does not electrically simulate the physical Pi or validate the handheld PCB.
- **Physical board measurements:** actual boot, performance, power, thermal, module, camera, touch, and emulator behavior.

Label every presentation result as measured, simulated, datasheet-based, or proposed. KiCad/3D/SPICE results show design intent and component behavior; they are not a proof that the physical build works.

## Deliverables by 25 November

1. A minimal Raspberry Pi OS Lite-derived setup booting on the Zero 2 W and Pi 4B, with configuration and package/service notes.
2. One OS-level service/memory study and one bootable custom kernel experiment with stock fallback.
3. Repeatable Python and C++ task results and a measured Zero 2 W/Pi 4B comparison.
4. A written module protocol plus an ESP32 module or software mock demonstrating discovery, capability exchange, an event/command, and removal.
5. A custom guest-to-ARM64 translation proof-of-concept. Use mGBA for working GBA gameplay; label a complete custom GBA emulator as follow-on work unless it is actually finished.
6. GBA measurements and an N64 stretch result if time and performance allow.
7. KiCad schematic/layout/3D views, selected power simulation or budget, and enclosure CAD/mock-up.
8. Presentation that explains the architecture, learning, measurements, limitations, and which features are demonstrated versus proposed.

## Feasibility and honest limits

The handheld concept, Lite-derived user profile, Python/C++ workflow, one-card hot-swap, ESP32 protocol, kernel build experiment, KiCad design, and a small JIT demonstration are all suitable project objectives. The Zero 2 W has only 512 MB RAM, so keep the desktop and apps narrow. GBA gameplay is the first emulator target to benchmark. N64 on the Zero is uncertain; full N64 compatibility and a complete bespoke GBA/N64 dynarec are larger projects than this presentation window. Use those as the long-term product direction and show measurable prototypes without presenting a partial CPU translator as a complete console emulator.

## Source references

- Raspberry Pi Zero 2 W hardware: https://www.raspberrypi.com/documentation/computers/zero.html
- Raspberry Pi OS Lite: https://www.raspberrypi.com/documentation/computers/os.html
- Raspberry Pi custom kernel: https://www.raspberrypi.com/documentation/computers/linux_kernel.html
- Linux memory management: https://www.kernel.org/doc/html/latest/admin-guide/mm/index.html
- QEMU dynamic translator internals: https://www.qemu.org/docs/master/devel/tcg.html
- QEMU system emulation: https://www.qemu.org/docs/master/about/
- mGBA ARM64 downloads: https://mgba.io/downloads.html
- Mupen64Plus ARM dynarec: https://mupen64plus.org/
- NetSurf limitations: https://www.netsurf-browser.org/documentation/info
- Touch Display 2 compatibility: https://www.raspberrypi.com/documentation/accessories/touch-display-2.html
