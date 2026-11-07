# Pocket Workstation OS — Image and OS Study Plan

## Goal

Build an identifiable Raspberry Pi OS Lite derivative with a repeatable setup recipe, integrated module service/SDK, and a measured resource policy. Package selection supports the project; service lifecycle, IPC, abstraction, recovery, and scheduling provide its main OS depth.

## Base and build method

Use one recorded supported **64-bit Raspberry Pi OS Lite (ARM64/AArch64)** release for the Pi Zero 2-family profile. Validate on Zero 2 W first and build the service, SDK dependencies, and applications for ARM64. Original Zero/Zero W compatibility is outside scope. Assume a future Zero 3 retains the connection method and supports 64-bit software; verify its board support, boot files, and physical interface before adding it to the tested image matrix.

Start with a checksum-verified Lite image and an idempotent provisioning recipe that installs project packages, configuration, and examples. Run it against a fresh image twice and compare manifests and behavior. The first milestone is repeatable setup, not byte-for-byte identical filesystem images.

When packaging is stable, use a pinned [pi-gen](https://github.com/RPi-Distro/pi-gen) revision and Lite-derived custom stage to generate the image. Use the documented `arm64` branch and freeze its revision instead of assuming current defaults. Complete image generation is preferable, but do not block the service on a second build-system migration. Record whether the artifact came from provisioning a base image or generating one.

Preserve:

- Base image URL/checksum, release, ABI, kernel/firmware versions, and board revision.
- Recipe revision, project package versions, configuration files, and package/service manifests.
- Enabled/disabled service decisions and dependency checks.
- Build environment, repository snapshot/package archive where available, and artifact checksum.
- Per-device first-boot setup: user credentials, SSH host keys if enabled, machine identity, and module permissions. Do not distribute a personal configured SD-card clone with secrets.

If upstream package versions cannot be recovered, state that limitation; a manifest alone does not guarantee a bit-reproducible rebuild. Keep normal update sources and an explicit update/retest process. Freeze the benchmark image, then test updates on a copy.

## Profiles and service audit

| Profile | Required content | Optional content |
|---|---|---|
| Base | Kernel/firmware, boot/storage, USB/class drivers, udev/systemd, broker, CLI, diagnostics | Networking according to workflow |
| Development | Base plus Python SDK, compiler tools, editor, offline help/examples | Small graphical session, sensor dashboard |
| Gaming | Selected emulator/core, input mapping, required audio/display support | Additional validated consoles |

Record service purpose, dependencies, whether it is running, measured cost, and disable/restore procedure. Remove only unused components. Keep Bluetooth for a Bluetooth keyboard, networking for a chosen transfer workflow, and audio for games. Removing a dormant package mainly saves storage; disabling a resident service can affect RAM/CPU. Measure both separately.

A proposed lightweight session must be tested for touch, readability, input, audio, and memory before it becomes the default. Keep a console recovery path. Launch dashboards/games on demand rather than keeping all applications resident.

## Boot and device trace

```text
Firmware/config → kernel + device tree → root filesystem
    → systemd/udev → module service → SDK/CLI/editor
```

Record `uname`, OS release, boot arguments, mounted cgroups, enabled units, loaded drivers, and kernel logs. Use `systemd-analyze` for its measured portion of boot and an external power-on-to-ready measurement for complete device startup.

For module attachment, capture USB enumeration, class-driver binding, device notification, broker open, host HELLO_REQUEST, descriptor validation, registry publication, and SDK discovery. Repeat after restarting the broker with the module still attached. These are distinct kernel and application-level discovery stages.

## Primary experiment: responsiveness under CPU contention

**Question:** does a deliberate CPU-allocation policy preserve module-client responsiveness, and what does it cost background work?

1. Verify the CPU controller is available in the selected kernel/cgroup setup. Record systemd/kernel versions and controller state.
2. Use a controlled test harness with sibling `pocket-interactive.slice` and `pocket-background.slice` under `pocket.slice`. Place the broker and measured SDK client in the interactive branch; place background jobs in the other. Keep this hierarchy in both conditions.
3. Baseline: equal CPU weights, 100:100. Policy condition: 400:100. This changes only relative CPU allocation; it is a starting experimental setting, not a proven optimum.
4. Run a deterministic 50 Hz module stream while enough background CPU workers contend for the Zero 2 W's cores. Add the selected real compile/Python task as a separate workload.
5. Measure host delivery latency, host command round-trip time, sequence gaps, queue depth, task correctness, CPU/RSS, and background completion time/throughput.
6. Repeat without background load to identify the service's normal overhead.

Use system-managed benchmark units with an explicit unprivileged application user to place the controlled client correctly. A normal editor in a separate user-session cgroup is not automatically covered by this policy. Report exactly which processes the harness includes; extend the production launcher only after the controlled result is understood.

CPU weights distribute capacity under contention; they do not reserve a deadline. Do not raise the service to real-time scheduling just to improve a graph. If the policy has no effect or hurts throughput excessively, report that result and explain it. [Linux cgroup v2](https://docs.kernel.org/admin-guide/cgroup-v2.html), [systemd resource-control source](https://github.com/systemd/systemd/blob/main/man/systemd.resource-control.xml).

## Supporting memory observation

Run an editor/client workload with a bounded page-touch program. Record RSS, minor/major faults, MemAvailable, swap counters, and process exits. Use a fixed working-set size and duration, an enforced test limit, and a recovery console; do not deliberately exhaust the entire interactive system.

Write to anonymous pages when studying demand allocation. First-touch minor faults are expected; major faults are not guaranteed. Study disk-backed faults with a separate file-backed workload or actual swap activity, and label warm/cold cache state. Explain allocation, demand paging, reclaim, and swap without inferring swapping from virtual size alone.

Keep swap settings unchanged during the primary CPU experiment. Only after it passes, consider a separate no-swap versus zram test with identical workloads. Compressed RAM can trade memory capacity for processing cost; do not assume a win. [Linux zram documentation](https://docs.kernel.org/admin-guide/blockdev/zram.html).

## Optional kernel work

A custom kernel is justified if required controller/observability support is absent, or if one specific configuration hypothesis needs testing. Keep the stock kernel, matching modules, and a recoverable boot configuration. Record config/source diff and boot the result on real hardware. Do not remove required storage/display/USB/input support to reduce size.

Kernel compilation is not required to call this a custom Linux distribution. If the selected stock kernel supports the experiment, prioritize the working service, tests, and measurements. An initramfs/BusyBox exercise is optional after the main project.

## Deliverables

- Bootable derivative artifact and recipe, with package/service/configuration records.
- Bundled daemon, Python SDK, CLI, and example programs.
- Boot/discovery trace and lifecycle demonstration.
- Controlled resource-policy results and memory observation.
- Recovery procedure and known-good image.

The plan does not claim these artifacts already exist. The implementation schedule is in [BUILD_GUIDE.md](BUILD_GUIDE.md).
