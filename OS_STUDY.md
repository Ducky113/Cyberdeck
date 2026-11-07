# Pocket Workstation — OS Study and Change Plan

## Goal

Show how Raspberry Pi OS moves from firmware to a usable Linux session, then make one measured change below the application layer. Keep the bootable Lite base and handheld display/input support intact.

## What to trace

```text
Pi boot firmware/config
        ↓
Linux kernel + device tree + kernel command line
        ↓
initramfs (when used) → mount root filesystem
        ↓
PID 1 / systemd → device discovery and services
        ↓
login shell or graphical session → editor and applications
```

Record the OS release, architecture, kernel release (`uname -a`), boot arguments (`/proc/cmdline`), service state (`systemctl`), loaded modules (`lsmod`), and boot timing (`systemd-analyze`). Use `dmesg` and `/proc/device-tree` to connect hardware discovery to the running kernel.

## Experiments ranked for the presentation

### Core: service and user-space trim

1. Save a clean Raspberry Pi OS Lite image and inventory enabled services/packages.
2. Disable one clearly optional service or package that is not used by the handheld.
3. Compare boot-to-console or boot-to-launcher time, idle RAM, CPU activity, and input power before and after.
4. Restore the service if the change breaks a required feature; record the result either way.

This demonstrates real OS configuration and service management. It changes startup and resident user-space load, not the kernel's paging algorithm.

### Core: virtual-memory/page-fault lab

Write a small C/C++ program that allocates memory in page-sized steps, touches one byte per page, then revisits the pages. Compare its behavior at low memory pressure and under a separate controlled load. Record:

- minor and major faults from `getrusage()`;
- peak RSS and virtual size from `/proc/self/status`;
- system page/swap counters from `/proc/vmstat` or `vmstat`;
- run time and visible responsiveness of the editor.

Explain that a page fault is the kernel handling a missing or disallowed virtual-to-physical mapping; many faults are normal. Demand paging, copy-on-write, reclaim, and swap have distinct behavior. Do not infer that a process has swapped merely from its virtual size.

### Core or stretch: device discovery

For the ESP32 USB-serial module, capture the plug sequence with `dmesg -w`, `udevadm monitor`, and `lsusb`. Follow the USB device into its serial interface, then show the module broker opening it and reading the protocol `HELLO`. This distinguishes kernel USB enumeration from the application's module-protocol handshake.

Only use a device-tree overlay for a GPIO-attached module that needs one. USB-serial discovery does not require a custom overlay. Raspberry Pi documents runtime overlay loading and inspection.

### Core: custom kernel

1. Preserve the stock boot image and copy the working system to a separate development image or backup.
2. Start from the Raspberry Pi configuration for the target board and OS architecture.
3. Change one optional kernel configuration item after checking its dependencies; do not remove storage, console/display, USB, input, network, or required filesystem support.
4. Build natively on the Pi 4B or cross-compile on another Linux machine. The Zero 2 W can also build, but it will take longer and puts more load on its limited RAM/storage.
5. Install the result as a separate kernel filename with a distinct local version; keep the stock kernel selected as a fallback.
6. Boot the custom kernel, confirm it with `uname -r`, and compare boot behavior, kernel image size, and the behavior affected by your selected option.

Do not promise a large speed or memory improvement from kernel trimming. The result is valuable as a kernel configuration/build/boot experiment even if runtime differences are small. If the selected kernel option cannot be changed safely, complete the service and paging labs and present the kernel build as a stretch rather than removing a required driver.

### JIT and virtual memory connection

The custom emulator's generated ARM64 code runs in user space. The JIT uses virtual memory for its code cache and follows page-execute permissions and instruction-cache synchronization requirements. Observe its RSS and page faults alongside translation time. The kernel provides the process and memory mappings; basic dynamic recompilation does not require moving game execution into the kernel.

### Optional: initramfs and minimal root

Use a small BusyBox initramfs to demonstrate the kernel starting its first user-space process and mounting or switching to a root filesystem. This is a separate experiment; keep Raspberry Pi OS Lite as the handheld's practical user-facing system.

## Keep the experiment recoverable

- Save a known-good image before kernel or boot changes.
- Keep the stock kernel and a known-good boot configuration available.
- Change one variable at a time and record exact commands and configuration diffs.
- Test the custom kernel and root filesystem on physical Pi hardware; QEMU does not reproduce every Raspberry Pi peripheral.

## Primary references

- Raspberry Pi kernel build/configuration: https://www.raspberrypi.com/documentation/computers/linux_kernel.html
- Raspberry Pi device-tree configuration and overlays: https://www.raspberrypi.com/documentation/configuration/
- Linux memory-management administration guide: https://www.kernel.org/doc/html/latest/admin-guide/mm/index.html
- Linux page-table and page-fault overview: https://www.kernel.org/doc/html/latest/mm/page_tables.html
