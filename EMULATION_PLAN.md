# Pocket Workstation OS — Emulation and Application Plan

## Role in the project

Gaming is a useful OS workload and demonstration of controller integration, audio/display behavior, resource pressure, and application launching. The OS does not depend on developing a new emulator or JIT. Run all selected emulators as normal applications in the same installation.

The user will handle the Arduino/GBA emulator project. Its implementation has not been inspected here, and finishing or reviewing it is not a prerequisite for the OS service. Integrate its Linux executable when ready, retaining an independent emulator baseline.

## Systems and implementation candidates

| System | Candidate | Priority and validation |
|---|---|---|
| Game Boy / Game Boy Color | Gambatte or an already-integrated compatible core | First additional systems; choose one fixed workload and measure |
| NES / Famicom | FCEUmm | Next coverage target; test controller mapping and save behavior |
| Game Boy Advance | User-owned project; mGBA baseline | Keep GBA as a product target; custom integration conditional on readiness |
| Master System / Mega Drive (Genesis) | PicoDrive | Follow-on after core OS and first games work; do not include CD/32X just because the core supports them |

These are candidates, not verified installed packages or promises of full-speed emulation on every target board. Confirm availability/build support for the selected 64-bit image and exact Zero 2-family board. Use native ARM64/AArch64 builds or compile from source for that target.

Primary references: [Gambatte](https://docs.libretro.com/library/gambatte/), [FCEUmm](https://docs.libretro.com/library/fceumm/), [PicoDrive](https://docs.libretro.com/library/picodrive/), and [mGBA downloads](https://mgba.io/downloads.html).

N64 is removed from the active plan. Additional systems increase coverage only when they work well enough to use; a long untested launcher menu is not a deliverable. SNES and more demanding platforms can be considered later after the above matrix is measured.

## GBA application integration boundary

An Arduino-targeted emulator may contain portable CPU/device logic plus platform-specific code. Arduino identifies an ecosystem rather than one CPU or memory configuration, so do not assume the entire program or its binary ports unchanged.

When the user's Linux build is ready, agree on:

- **Display:** framebuffer/frame submission to the chosen Linux frontend, with explicit scaling and frame pacing.
- **Audio:** sample format/rate, bounded buffering, and underrun handling.
- **Input:** standard Linux controller events so both ordinary USB HID and broker-created controllers work.
- **Storage:** normal files for game images and saves; safe behavior when removable media disappears.
- **Timing:** a host monotonic clock and pacing appropriate to the emulated machine.
- **Packaging:** supported ABI, launcher entry, configuration/save paths, and clean exit back to the editor.

A standalone SDL-based port or a libretro frontend integration are possible approaches; select according to the existing code when available. Do not impose a rewrite as part of this OS plan.

## What the earlier JIT proposal meant

An interpreter decodes guest instructions and performs their effects in host code. A JIT instead translates guest instruction blocks into native host instructions and caches them. A small guest-to-ARM64 prototype could translate arithmetic and branches while leaving most console hardware unimplemented. That is a separate compiler/emulation project, not a prerequisite for running Linux apps or porting an interpreter.

No new ARM64 JIT is required. Build the OS service and applications for ARM64; the public SDK and module protocol do not expose host instruction-set details. Performance claims about the user's emulator wait for actual measurements.

## Launching and usability

Keep one simple game launcher or direct launch commands. If using libretro cores, evaluate the frontend's footprint alongside each core. Do not replace the OS with a separate gaming distribution. Install only the selected cores, keep configuration reproducible, and avoid expensive visual effects until the baseline runs well.

Require a clean return to the editor, predictable save paths, a known controller mapping, and workable audio. A physical HID controller is an acceptable first gameplay input while the custom controller example is built. A GPIO/serial joystick needs a deliberate Linux input adapter; connecting it alone does not make it visible to games.

## Measurement and acceptance

Use a fixed homebrew/test scene or another available test workload per system. Record core/version, board/image ABI, resolution, scaling, audio setup, frame skipping, and all speed settings. Report emulated speed and frame pacing rather than renderer FPS alone, together with audio quality, memory, temperature, input behavior, and power.

Commit to one working game demonstration. Add other systems only after testing; report compatibility per workload rather than claiming universal console support. Keep a failure/limitation table and a saved offline demo recording.

## Other applications with the same OS value

- Sensor inspector and timestamped logger using the bundled SDK.
- Python script that discovers a sensor and controls an LED without serial setup.
- Small C++ edit/build/run task and offline API documentation.
- File transfer through the removable data card with explicit safe eject.

These workflows provide usefulness even when a particular emulator port is unfinished.
