# Comprehensive System Architecture & Engineering Documentation: Custom CM5 Modular Handheld
**Authoritative Implementation Plan, Wiring Topologies, and Structural Specifications**

---

## 1. Executive System Overview & Block Diagram

This document serves as the absolute engineering blueprint for the construction of a custom, modular, multi-environment handheld gaming and computing console powered by the Raspberry Pi Compute Module 4/5 (CM5) architecture. The design balances industrial modularity, robust structural integrity, highly integrated dual-source power handling, precise battery fuel gauging, dual-input KVM data/video switches, and isolated multi-environment firmware/OS execution.

### 1.1 Structural Taxonomy (The Three Modules)
The physical device is segmented into three distinct structural modules engineered with a matte black or dark gray premium high-quality finish to approximate consumer electronics grade tolerances:
1. **The Main Unit (Core Frame):** Centrally houses the core stack, processing unit, primary thermal solution, high-speed internal USB topology, primary power storage, display controller, and the KVM routing sub-systems.
2. **Modular Side Controllers:** Symmetric input modules that attach mechanically and electrically to the left and right flanks of the Main Unit. They contain zero processing units locally, functioning as pure electromechanical routing systems connecting directly back to a centralized MCU.
3. **Standalone Bottom Keyboard Module:** A detachable productivity and input expansion deck that locks into the base of the Main Unit. It contains its own processing power, local energy storage, dual-mode communications engine, and dual-purpose input array.

### 1.2 Unified System Architecture Block Diagram
The schematic topology below outlines the data backbone, power paths, and switching mechanisms:

```
+---------------------------------------------------------------------------------------+
|                                      MAIN UNIT                                        |
|                                                                                       |
|   +-----------------------+      HDMI      +-------------+   HDMI   +-------------+   |
|   |                       |===============>|             |=========>|             |   |
|   |  Raspberry Pi CM5     |                | Manual HDMI |          | Display     |   |
|   |  (Core Brains)        |   [External]   | Switch      |          | Controller  |   |
|   |                       |==[HDMI Port]==>|             |          | Board       |   |
|   +-----------------------+                +-------------+          +-------------+   |
|         ||           ^                                                     ||         |
|      (GPIO)          | (Regulated 5V/3.3V)                                 ||         |
|         ||           |                                                     ||         |
|         \/           |                                                     \/         |
|   +------------+     |      +---------------+   USB   +------------+   +----------+   |
|   | Internal   |     +------| Buck          |<========|            |<==| Touch    |   |
|   | OS SD Slot |            | Converters    |         | Manual USB |   | Panel    |   |
|   +------------+            +---------------+         | Switch     |   +----------+   |
|                                ^         ||           +------------+                  |
|                                |         ||                 ||                        |
|                         (Unified Power)  || (Internal USB)  || (External USB-C Port)  |
|                                |         ||                 \/                        |
|   +---------------+      +-----------+   ||    +----------------------------------+   |
|   | Main Battery  |=====>|           |<==+     | Internal USB 2.0 Hub (GL850G)    |   |
|   | (2S 18650)    |      | Ideal     |         +----------------------------------+   |
|   +---------------+      | Diode     |               ||              ||               |
|         ||               | Controller|               ||              ||               |
|    (I2C Telemetry)       |           |               \/              \/               |
|         ||               +-----------+         +------------+  +------------------+   |
|         \/                     ^               | Controller |  | GL823K External  |   |
|   +---------------+            |               | MCU        |  | Hot-Swap SD Slot |   |
|   | MAX17048      |            |               | (RP2040)   |  +------------------+   |
|   | Fuel Gauge    |            |               +------------+                         |
|   +---------------+            |                     ||                               |
+--------------------------------|---------------------||-------------------------------+
                                 |                     ||
                           (Pogo Interconnect:         || (Analog/GPIO Lines via Pogo)
                            V-BAT, V-CHG, GND,         ||
                            I2C SDA/SCL, USB D+/D-)    ||
                                 |                     ||
                                 \/                    \/
+---------------------------------------------------------------------------------------+
|                                DETACHABLE CONTROLLERS                                 |
|   Left Gamepad Array <=========================================> Right Gamepad Array |
+---------------------------------------------------------------------------------------+
                                 ||
                                 || (Modular Mechanical and Pogo Array Interface)
                                 \/
+---------------------------------------------------------------------------------------+
|                              BOTTOM KEYBOARD MODULE                                   |
|                                                                                       |
|   +---------------------+   Bluetooth    +----------------------+                     |
|   | Keyboard Matrix +   |--------------->| External Desktop/PC  |                     |
|   | Thumbstick Mouse    |                +----------------------+                     |
|   +---------------------+                                                             |
|         ||          ^                                                                 |
|         \/          | (Wired UI Override / Internal Charge Bus)                       |
|   +---------------------+      Power      +---------------+     I2C     +---------+   |
|   | Wireless Core MCU   |================>| Local Battery |============>|MAX17048 |   |
|   | (nRF52840)          |                 | (1S Li-ion)   |             | Fuel    |   |
|   +---------------------+                 +---------------+             | Gauge   |   |
+-------------------------------------------------------------------------+---------+   |
```

---

## 2. Electromechanical Hardware Shortlist & BOM

The following components have been selected to fulfill strict budgetary requirements optimized for direct acquisition within Taiwanese electronic sourcing hubs (such as the Guanghua Digital Plaza) or high-volume regional suppliers. The target price envelope is established at **~$224.50 USD**.

### 2.1 Bill of Materials Breakdown

| Category | Component Description | Core IC / Standard | Target Qty | Sourcing Context | Est. Cost (USD) |
| :--- | :--- | :--- | :---: | :--- | :---: |
| **Core & Compute** | Raspberry Pi Compute Module (CM5) | Broadcom BCM2712 / Variant | 1 | Standard Sourcing | $45.00 |
| **Core & Compute** | CM5 Mini IO Carrier Board | Breakout Board Form Factor | 1 | Micro-electronics Market | $35.00 |
| **Core & Compute** | Internal High-Speed USB Hub Module | Realtek FE1.1s / Genesys GL850G | 1 | Bare PCB Module | $8.00 |
| **Storage** | Micro SD Card Slot (GPIO Breakout) | Standard Push-Push SDIO Slot | 1 | Internal OS Storage | $2.50 |
| **Storage** | Hot-Swappable USB Micro SD Reader | Genesys Logic GL823K | 1 | Stripped Bare PCB | $2.50 |
| **Power Logic** | Ideal Diode Dual-Channel Controller | Linear Technology LTC4412 / Similar | 1 | Power OR-ing Board | $6.50 |
| **Power Logic** | High-Precision System Fuel Gauge | Analog Devices MAX17048 | 2 | I2C Telemetry Boards | $10.00 |
| **Power Storage**| High-Drain 18650 Cells | Lithium-Ion 3.7V (2S Configuration) | 2 | Main Battery Array | $9.00 |
| **Power Storage**| 2S Lithium-Ion Battery Management | 2S Balanced BMS w/ Protection | 1 | Main Board Protection | $4.00 |
| **Power Conversion**| DC-DC Step-Down Buck Converters | Monolithic Power Systems MP1584 | 2 | 5V Rail & 3.3V Rail | $3.50 |
| **Power Charging**| In-Line Protection Rectifiers | Schottky Diodes SS34 (Surface Mount)| 2 | Bi-directional Isolation | $1.00 |
| **Display System**| High-Definition IPS Panel Array | 6-inch 720p IPS LCD | 1 | Panel Sub-assembly | $33.00 |
| **Display Logic** | Digital HDMI Drive Controller | HDMI-to-MIPI/LVDS Drive Board | 1 | Dedicated Screen Controller| $12.00 |
| **Display Switching**| Manual Hardware KVM Switch Array | 2-Port HDMI Matrix + 2-Port USB Switch| 1 | Bare Mechanics Module | $6.00 |
| **Control Logic** | Primary Gamepad Co-Processor | Raspberry Pi RP2040 | 1 | Waveshare RP2040-Zero/Pico | $4.00 |
| **Control Logic** | Wireless Composite Peripheral MCU | Nordic Semi nRF52840 | 1 | nice!nano / Seeed XIAO BLE | $9.00 |
| **Input Mechanics**| Dual-Axis Analog Pots | PSP-Style Low-Profile Joysticks | 3 | Gamepads & Thumb-Mouse | $4.50 |
| **Input Mechanics**| Electromechanical Switches | Tactile Micro-switches & Key Matrix | 1 | Full Device Input Set | $5.00 |
| **Connectors** | Spring-Loaded Contacts & Plates | Gold-Plated Pogo Pin Array (6-Pin/8-Pin)| 1 | Modular Connections | $4.50 |
| **Connectors** | Flexible Flat Cables & Sockets | 0.5mm Pitch FFC Ribbon Bundles | 1 | Static Internal Interconnect| $3.50 |
| **Thermal Mgmt** | Active Air Displacement Sub-system | 5V Low-Profile Blower Fan & Heatsink| 1 | Enclosed Forced Induction | $5.50 |
| **TOTAL** | **Estimated Budget Sourcing Threshold**| | | **Guanghua / Taobao Hubs** | **~$224.50** |

---

## 3. Power, Charging & Telemetry Architecture

The engineering of the power system demands explicit isolation between discharging topologies and charging paths to ensure absolute safety, eliminate component frying, and preserve accurate state-of-charge calculation across detachable modules.

### 3.1 Discharging Pipeline (Power OR-ing)
To integrate the energy storage of the main unit (2S Li-ion battery configured for a nominal 7.4V) and the keyboard module (1S Li-ion battery operating at nominal 3.7V–4.2V) without risk of unregulated over-current equalizations, an analog power OR-ing scheme is mandatory.
* **The Ideal Diode Controller Sub-system:** Rather than using passive Schottky rectifiers that exhibit considerable forward voltage drops ($V_f pprox 0.4\text{V}$) and generate thermal stress under load, an Ideal Diode Controller (LTC4412 or equivalent driving external P-channel MOSFETs) is wired to both inputs.
* **Load Balance Mechanics:** The main unit's battery array runs through its local BMS and feeds into Channel A of the Ideal Diode Controller. The keyboard battery line passes across the interface dock via the `V-BAT` contact pad into Channel B. The Ideal Diode circuit isolates the lines, evaluating which source presents higher potential.
* **Unified Downstream Regulation:** The single combined output rail originating from the Ideal Diode Controller serves as the source path for two high-frequency DC-DC buck converters (MP1584):
  * **5V Power Rail:** Step-down conversion optimized to deliver continuous $5\text{V} \pm 0.1\text{V}$ at up to $4\text{A}$ to feed the Raspberry Pi CM5 core logic, active cooling fans, and high-current USB segments.
  * **3.3V Power Rail:** Step-down conversion delivering $3.3\text{V}$ dedicated strictly to the digital logic circuits, GPIO states, pull-up terminations, and co-processor MCUs.

### 3.2 Charging Infrastructure (Bi-Directional Shared V-CHG Bus)
Universal charging capability permits plugging a standard external USB-C Power Delivery source into either the Main Unit or the detachable Keyboard Module to feed both internal battery systems simultaneously.
* **The Shared Charge Pipeline:** A continuous electrical path designated as the `V-CHG` bus runs between both modules, crossing the mechanical interface via a dedicated, heavy-duty gold pogo connection pin.
* **Reverse Current Isolation (Double-Plug Protection):** To mitigate user error where active external charging blocks are inserted into both the Main Unit and the Keyboard Module simultaneously, surface-mount Schottky Diodes (SS34, rated for $3\text{A}$, $40\text{V}$) are implemented immediately downstream of each physical USB-C receptacle. The diodes function as one-way hardware gates: they allow external $5\text{V}$ standard charge current to safely enter the internal `V-CHG` bus but completely block power residing on the bus from shorting backwards out of an unpopulated or competing external connection.
* **Natural Balancing Model (Option A):** Charging follows simultaneous current allocation based on natural hardware properties. Because the main 18650 cells demand higher charging currents over extended periods compared to the minor capacity cells of the keyboard module, both charging ICs (the primary BMS/TP4056 array and the nRF52840's native charging management circuit) draw from the shared `V-CHG` rail concurrently. The smaller keyboard pack achieves full saturation ($100\%$ SoC) rapidly and naturally restricts its current draw, allowing the remaining current capacity to focus onto the primary cells without any software logic overhead.

### 3.3 Battery Telemetry & Smart Fuel Gauges
Because the Ideal Diode controller acts purely as a passive analog power switcher, the operating systems cannot natively determine the combined structural battery capacity based on simple voltage decay curves. When the keyboard is attached, voltage curves flatline, leading to OS UI lockup, frozen state percentages, and systemic calculation glitches.
* **Dual I2C Fuel Gauge Interconnect:** Independent Maxim Integrated MAX17048 Fuel Gauge ICs are wired directly across the terminal leads of each respective battery pack. These ICs leverage the ModelGauge algorithm to constantly track State of Charge (SoC) and remaining capacity in milliampere-hours ($\text{mAh}$) independent of current voltage sag or loading variables.
* **Telemetry Routing Matrix:** The MAX17048 assigned to the keyboard module streams its telemetry over a local $\text{I2C}$ bus. This bus runs through the structural pogo pin interface via the `SDA` and `SCL` pins, establishing a direct connection to the primary $\text{I2C}$ controller pins of the Raspberry Pi CM5.

---

## 4. Digital Data Backbone & Storage Topologies

To optimize bandwidth delivery, reduce overall electrical noise, and lower the system's idle power draw, a balanced USB topology is established alongside isolated storage pipelines.

```
                  +--------------------------+
                  |    Raspberry Pi CM5      |
                  +--------------------------+
                     |                    |
              (Native USB 2.0)      (Native GPIO)
                     |                    |
                     v                    v
         +-----------------------+   +-----------------------+
         | Internal USB 2.0 Hub  |   | Dedicated Internal    |
         | (FE1.1s / GL850G)     |   | Micro SD Slot (OS)    |
         +-----------------------+   +-----------------------+
           |        |        |
           |        |        +-------------------------+
           |        |                                  |
           v        v                                  v
  +-----------+  +----------------------+    +-------------------+
  | Controller|  | Hot-Swappable        |    | Manual USB Switch |
  | MCU       |  | External SD Reader   |    | (Touch Matrix)    |
  | (RP2040)  |  | (GL823K Chipset)     |    +-------------------+
  +-----------+  +----------------------+              |
                                                       |
                                            +----------+----------+
                                            v                     v
                                      [Position A]          [Position B]
                                      Internal CM5         External Shell
                                      Data Loop            USB Passthrough
```

### 4.1 Internal USB Architecture
The system breaks out the primary USB channel of the Compute Module 5 into a localized Genesys Logic GL850G or Realtek FE1.1s 4-port minimum USB 2.0 hub controller board. This architectural limitation to USB 2.0 speeds (~$480\text{Mbps}$) is deliberately selected to lower routing tolerances, eliminate high-frequency crosstalk with analog controls, and conserve valuable power.
1. **Downstream Port 1 (System Controller Link):** Hardwired to the on-board primary input co-processor (RP2040).
2. **Downstream Port 2 (External Hot-Swap Storage Link):** Directly mapped to a localized Genesys Logic GL823K bare PCB card reader module.
3. **Downstream Port 3 (KVM Touch Routing Input):** Connected to Terminal A of the manual USB D+/D- data switcher.
4. **Downstream Port 4 (Modular Expansion Path):** Routed directly to the lower pogo array interface to feed data parameters to the keyboard deck when docked.

### 4.2 Storage Isolation Strategy
To prevent system instabilities, operating system partition failure, and random write errors, storage is bifurcated into independent system roles:
* **Internal Secure OS Storage:** A standard SDIO push-push micro SD slot is wired directly to the native storage controller lines of the CM5 via designated GPIO pins (GPIO 22–27). This slot is mounted internally on the board and remains protected inside the core enclosure framework, accessible only via physical disassembly of the rear security backplate. This volume holds the boot sector, system configurations, kernel structures, and base operating files, ensuring it can never be ejected while active.
* **External Hot-Swappable Storage Expansion:** The GL823K USB card reader module breaks out to a physical micro SD slot positioned flush with the outer structural edge of the main chassis shell. Because this storage pipeline executes entirely over the USB mass storage class protocol, the underlying Linux kernel treats it natively as an un-fixated external media volume. Hot-swapping can be performed safely at any point; the OS natively detects the device insertion or deletion events and manages mount/unmount requests without risking kernel panics or core partition corruption.

### 4.3 On-The-Go (OTG) Gadget Routing
To enable high-speed direct data transfers from a standard external machine (laptop/desktop PC) into the handheld storage environment without requiring network links, the CM5’s primary OTG-capable USB link is isolated. It runs via a dedicated routing path completely separate from the internal USB hub, terminating at an external chassis-mounted USB-C port. When a host computer hooks into this port, the CM5 initiates Linux USB Gadget Core architectures (`g_mass_storage`), unmounting the secondary user partition locally and projecting it to the host machine as a standard, high-speed external hard drive volume.

## 5. Input, Control & Peripheral Co-Processors

The interface architecture must process both high-density mechanical switch matrices and analog potentiometer vectors with zero deadzone lag or signal jitter.

### 5.1 Primary Side Controller Engine (RP2040 Platform)
The left and right side analog modules feed their raw electrical lines into a centralized Raspberry Pi RP2040 microcontroller positioned inside the Main Unit.
* **Input Pin Assignments & Physical Topology:**
  * **Digital Switch Mapping:** Up to 16 independent tactile inputs (D-Pad, ABXY, bumpers, system controls) are wired directly to dedicated GPIO inputs. Each line features a physical hardware pull-up resistor ($10\text{k}\Omega$) tied to the $3.3\text{V}$ rail to guarantee clean state transitions.
  * **Analog Signal Splitting:** Two dual-axis PSP-style joysticks provide spatial coordinates via internal $10\text{k}\Omega$ potentiometers. The four wipers (Left X, Left Y, Right X, Right Y) are mapped straight to the four native analog-to-digital converter (ADC) inputs of the RP2040 (GPIO 26-29).
* **Signal Protection Layout:** To suppress EMI and RF interference generated by the adjacent active display driver and high-frequency buck converters, all analog joystick lines are run using short, point-to-point traces, wrapped in localized ground planes, and isolated from nearby digital traces by dedicated ground guard rings.
* **Firmware Configuration:** The RP2040 executes the open-source **TinyUSB** stack or a customized version of **QMK**, identifying over the internal USB hub controller as a high-polling-rate, standard human interface device (HID) gamepad.

### 5.2 Standalone Bottom Keyboard Module Engine (nRF52840 Platform)
The lower keyboard module operates as an independent, intelligent composite input device utilizing a Nordic Semiconductor nRF52840 processing core (such as a nice!nano or Seeed Studio XIAO board).
* **Firmware Stack & Core Communication:** Running the modern **ZMK Firmware** framework, the nRF52840 leverages its native Bluetooth Low Energy (BLE) transceiver to run as a universal wireless keyboard and mouse. It pairs seamlessly across multiple selectable host profiles with external PCs, mobile devices, or tablets.
* **Dual-Mode Connection Switching:** The ZMK framework continuously monitors the state of the local USB-C hardware connection connection pins. The moment the keyboard module registers a physical connection to the Main Unit, the firmware shifts communication paths: it suspends active BLE transmission bursts and dynamically re-routes all keyboard and pointer data packages over the wired USB data pins. Unplugging the module reverses this state instantly.
* **Integrated Pointer System (Thumbstick Mouse):** A third low-profile PSP-style analog joystick is embedded directly into the lower right corner of the keyboard matrix. The nRF52840 continuously samples the joystick's internal potentiometers via its local ADC pins. The firmware maps these absolute vector voltages into relative delta positioning coordinates, translating joystick deflections directly into smooth mouse pointer motions.

## 6. Display, KVM & Touchscreen Switching

The design incorporates a manual hardware KVM switching matrix, allowing the system to toggle between operating as a standalone mobile console or functioning as an external, touch-enabled portable monitor.

```
       +-------------------------------------------------------------+
       |                         MAIN UNIT                           |
       |                                                             |
       |  +--------------------+               +------------------+  |
       |  | Raspberry Pi CM5   |               | External Chassis |  |
       |  | (HDMI Video Out)   |               | HDMI Input Port  |  |
       |  +--------------------+               +------------------+  |
       |            |                                    |           |
       |            |                                    |           |
       |            v                                    v           |
       |     +-------------------------------------------------+     |
       |     |          MANUAL COHESIVE HARDWARE SWITCH        |     |
       |     |                                                 |     |
       |     |    [Position A]                  [Position B]   |     |
       |     |    Internal CM5 Source           External Host  |     |
       |     +-------------------------------------------------+     |
       |                              |                              |
       |                              v                              |
       |               +-----------------------------+               |
       |               | HDMI LCD Controller Board   |               |
       |               +-----------------------------+               |
       |                              |                              |
       |                              v                              |
       |               +-----------------------------+               |
       |               | 6-inch 720p IPS Display     |               |
       |               +-----------------------------+               |
       +-------------------------------------------------------------+
```

### 6.1 Display Specification & Video Routing
The handheld utilizes a 6-inch 720p ($1280 \times 720$ resolution) IPS display panel. This configuration provides wide viewing angles, crisp color accuracy, and matches the low-bandwidth, lower-power design goals of the system.
* **The Manual HDMI Switching Matrix:** Video signals are managed using a dual-port manual HDMI bi-directional switch module.
  * **Input Source A:** Fed directly by a high-density, shielded flexible flat cable (FFC) routing from the native HDMI transmitter port of the CM5.
  * **Input Source B:** Fed via a short internal link originating from an HDMI female port mounted flush onto the external shell chassis.
  * **Common Switch Output:** Connects directly into the input port of the primary HDMI LCD drive board, which scales the signal and drives the IPS panel. Changing inputs is handled via a physical manual toggle switch located on the top edge of the case, requiring zero software configurations or custom microcode.

### 6.2 Touchscreen Data Passthrough
To ensure the high-resolution touch panel maintains full utility when switching between internal and external video sources, its USB interface is routed through an isolated manual USB 2.0 D+/D- data switch.
* **Position A (Internal Execution):** The data path is routed straight into a downstream port of the main unit's internal GL850G hub, establishing touch control for the internal CM5 OS environment.
* **Position B (External Control Hand-off):** The switch breaks the internal link and re-routes the D+/D- data lines out to an external chassis-mounted USB pass-through port. When connected to an external laptop via a USB cable, the external host machine natively discovers the touch controller as a standard USB digitizer, allowing the user to interact with the external laptop using the handheld's integrated screen.

## 7. Operational Software Environments & Custom Automation

Thanks to the isolated storage architecture, users can run dedicated gaming setups or a full desktop environment using an automated single-card system config.

### 7.1 Single-Card Operating Systems Allocation
The internal, hidden system micro SD card is formatted using **PINN (PINN Is Not NOOBS)**, an advanced system bootloader manager optimized for Raspberry Pi systems.
* **Storage Partition Isolation Layout:**
  * **System Bootloader Partition (`/boot`):** Houses the core PINN initialization engine, network recovery drivers, and system selection maps.
  * **Partition Environment 1 (`/root_batocera`):** A highly stripped down, minimalist Linux environment optimized exclusively to execute the **Batocera** frontend. It blocks background desktop overhead to dedicate all system cycles toward running low-latency emulation engines.
  * **Partition Environment 2 (`/root_raspbios`):** A full 64-bit **Raspberry Pi OS** distribution containing a complete desktop environment, window management tools, full browser engines, programming compilers, and media decoders.

### 7.2 Automation Telemetry Aggregation Script Description
To overcome the telemetry blind spot created by the Ideal Diode power-sharing hardware, a background Python automation script runs on boot in both environments. This script polls both high-precision MAX17048 fuel gauges over the I2C bus, calculates the unified remaining runtimes, and injects this data directly into the system's UI elements.
The Python daemon executes a loop every 5 seconds, querying registers `0x04` (State of Charge) and `0x16` (Change Rate) across both target addresses (`0x36` and `0x37`). It scales these metrics relative to the respective module battery capacities (e.g., 5000mAh main vs 1200mAh keyboard) and saves the unified result directly to `/tmp/composite_battery_soc` for structural desktop panel rendering.

## 8. Interface Connectors & Pinout Specifications

```
             DOCKING CONTACT PLATES INTERFACE TOPOGRAPHY
             
             +-----------------------------------------+
             |  [Pin 1]   [Pin 2]   [Pin 3]   [Pin 4]  |
             |   V-BAT     V-CHG     GND       SDA     |
             |                                         |
             |            [Pin 5]   [Pin 6]            |
             |             SCL       USB-D             |
             +-----------------------------------------+
```

To maintain high data speeds and support quick detachment cycles without risking trace breakages or connector wear, we rely on heavy-duty gold-plated pogo pin blocks for our modular expansion rails.

### 8.1 6-Pin Mechanical Interface Layout Specification
The connection line linking the Main Unit to the lower Keyboard Module requires 6 distinct, isolated electrical tracks.
* **Pin 1: `V-BAT` (Analog Power Transfer Link):** Routes raw high-current battery power originating from the keyboard’s internal cell directly into Channel B of the Main Unit’s Ideal Diode Controller board.
* **Pin 2: `V-CHG` (Bi-directional Charging Pipeline):** Establishes the shared $5\text{V}$ charging rail. This pin lets either module share incoming wall charger power with the other module's battery charging sub-systems.
* **Pin 3: `GND` (Unified Common System Ground):** Establishes a common ground plane across both modules, ensuring clean logic references and reducing electrical noise.
* **Pin 4: `SDA` (System Telemetry Data Line):** Carries the serial data streams from the keyboard module's internal MAX17048 fuel gauge straight to the CM5's native $\text{I2C}$ controller.
* **Pin 5: `SCL` (System Telemetry Clock Line):** Carries the clock synchronization pulses generated by the CM5 over to the keyboard module's MAX17048 fuel gauge over the $\text{I2C}$ bus.
* **Pin 6: `USB-D` / Optional Signal Layer:** An isolated contact path reserved for secondary options, such as routing direct wired USB configurations or acting as an extra grounding shield.

## 9. Thermal Management & Active Cooling Layout

The Broadcom BCM2712 processor inside the Compute Module 5 creates a tight thermal envelope under sustained emulation loads, requiring an active cooling solution.

### 9.1 Physical Thermal Assembly Layout
1. **Thermal Interface Layer:** High-conductivity soft thermal pads ($6.0\text{W/mK}$) are applied directly to the primary SoC chip, RAM packages, and Power Management IC (PMIC) surfaces to eliminate micro-gaps.
2. **Convective Dissipation Component:** A low-profile, black anodized copper alloy pin-fin heatsink is mounted directly to the carrier board frame, applying consistent mechanical clamping force over the thermal pads.
3. **Forced Air Displacement Solution:** A micro $5\text{V}$ brushless blower fan is embedded immediately adjacent to the heatsink. The fan is connected directly to a PWM-capable header on the carrier board, dynamically mapping fan speeds to actual core temperatures to reduce system noise at idle.

### 9.2 Mechanical Airflow Path Routing
* **Induction Intakes:** Precision-machined horizontal slot intake arrays are positioned on the rear backplate of the Main Unit's 3D-printed chassis, drawing cool air directly into the blower fan's intake vortex.
* **Exhaust Distribution Porting:** Internal structural air channels redirect the high-velocity airflow horizontally across the copper cooling fins, exhausting heated air through large vents located along the top edge of the chassis. This thermal path prevents heat buildup near the hand grips, ensuring comfortable temperatures during extended gaming sessions.

## 10. Step-by-Step System Assembly & Commissioning Plan

Building this custom handheld follows a strict 6-phase plan, moving from initial bench validation over to final mechanical assembly and configuration.

### Phase 1: Bench Prototyping & Core Power Verification
* **Step 1:** Assemble the 18650 battery cells into their configuration, solder them to the input leads of the 2S balanced BMS, and route the output lines directly into Input Channel A of the Ideal Diode Controller board.
* **Step 2:** Solder the output from the Ideal Diode Controller into the input pads of both MP1584 DC-DC buck converter modules.
* **Step 3:** Energize the circuit. Use a digital multimeter to verify the output voltages. Adjust the buck converters' potentiometers until you read a stable $5.0\text{V} \pm 0.05\text{V}$ on the primary rail and $3.3\text{V} \pm 0.03\text{V}$ on the logic rail. **Do not connect any computing boards until these values are verified.**
* **Step 4:** Mount the Raspberry Pi CM5 into the socket of the Mini IO Carrier Board. Wire the regulated $5\text{V}$ and ground lines to the corresponding input pins on the carrier board.
* **Step 5:** Flash a standard image onto a micro SD card, insert it into the carrier board's slot, and power on the system to verify clean Linux boot sequences.

### Phase 2: USB Infrastructure & Storage Setup
* **Step 1:** Connect the primary USB 2.0 Hub board (FE1.1s/GL850G) directly to the native USB header pins on the CM5 carrier board.
* **Step 2:** Take the bare GL823K micro SD reader module and solder its power and data lines ($5\text{V}$, `GND`, `D+`, `D-`) over to Downstream Port 2 of the internal USB hub.
* **Step 3:** Insert an external expansion micro SD card into the GL823K reader slot. Power on the system and verify the storage mounts cleanly in the OS as a standard external drive path (`/dev/sdb`).
* **Step 4:** Break out the secondary internal SDIO card slot by wiring it to the CM5's native GPIO lines (GPIO 22–27). Flash this internal card with **PINN** to establish the secure, dual-boot OS environment.

### Phase 3: Input Firmware & Co-Processor Integration
* **Step 1:** Flash the RP2040 co-processor board with the **TinyUSB** HID Gamepad firmware stack.
* **Step 2:** Wire the buttons (using $10\text{k}\Omega$ pull-up resistors) and the analog joysticks' wipers to the corresponding GPIO and ADC pins on the RP2040.
* **Step 3:** Connect the RP2040's USB lines to Downstream Port 1 of the internal USB hub. Open an input testing utility in Linux (`jstest`) to verify that all button states and analog stick axes register correctly.
* **Step 4:** Flash the nRF52840 keyboard module with the **ZMK Firmware** stack. Solder its local battery to the board's charging pads, and verify clean wireless pairing over Bluetooth.

### Phase 4: Display Calibration & KVM Switching
* **Step 1:** Mount the 6-inch IPS screen and connect its display ribbon cable to the output socket of the HDMI driver controller board.
* **Step 2:** Connect the CM5's HDMI output to Input A of the manual hardware HDMI switch. Wire the external chassis HDMI input port over to Input B. Solder the common output of the switch to the input of the screen's HDMI driver board.
* **Step 3:** Route the touchscreen's USB data lines into the common input of the manual USB switch. Connect Output A to the internal USB hub, and route Output B out to the external chassis USB-C port.
* **Step 4:** Flip the physical hardware toggles. Verify that both the video feed and touch controls switch smoothly between the internal CM5 system and an external laptop host.

### Phase 5: Keyboard Enclosure Integration & Advanced Charging
* **Step 1:** Assemble the mechanical layout of the keyboard module, positioning the nRF52840, keyboard matrix, and thumbstick mouse joystick inside the bottom shell casing.
* **Step 2:** Wire the gold contact pads of the 6-pin pogo array inside the keyboard housing, ensuring correct pinout alignment (`V-BAT`, `V-CHG`, `GND`, `SDA`, `SCL`).
* **Step 3:** Install the protective Schottky diodes (SS34) immediately behind the USB-C charging receptacles on both the Main Unit and the keyboard module to activate the double-plug safety isolation.
* **Step 4:** Connect an external USB charger to the keyboard's port. Verify that the charge current travels up the pogo pins to charge the main unit's batteries, then reverse the test to ensure bi-directional charging functions safely.

### Phase 6: Final Mechanical Assembly & Thermal Calibration
* **Step 1:** Apply high-conductivity thermal pads to the CM5's core chips, mount the anodized copper heatsink, and screw down the active 5V blower fan assembly.
* **Step 2:** Replace all temporary breadboard connections with shielded FFC ribbon cables or tightly bundled wires to reduce signal cross-talk and keep internal routing clean.
* **Step 3:** Carefully position and secure all internal circuit boards, battery packs, and switches into the 3D-printed main chassis shell using micro fasteners.
* **Step 4:** Snap the side controllers and the bottom keyboard module onto the main frame. Verify that all mechanical tolerances line up, the pogo pins compress evenly, and the magnetic guide rails lock together firmly.
* **Step 5:** Boot into the OS, verify the custom Python telemetry script automatically aggregates both fuel gauge capacities, and run a sustained emulation workload to confirm the cooling fan maintains stable core temperatures.