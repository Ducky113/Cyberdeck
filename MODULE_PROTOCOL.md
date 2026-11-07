# Pocket Workstation — Module Protocol v0

## Purpose

Let small ESP32 modules own their sensors, buttons, motors, and other low-level hardware while the Raspberry Pi runs one small module service. The module reports a standard identity and capability description; the Pi handles discovery, generic commands, and events. Adding a module should not require installing a separate Linux protocol stack or kernel driver for every sensor family.

The protocol centralizes the interface. It cannot remove all module-specific work: every ESP32 still needs firmware for its hardware, and a new capability may need a host-side handler or user interface.

## v0 transport and discovery

- Use USB serial for the first prototype. Select an ESP32-family board with native USB CDC or an onboard USB-to-serial bridge. The Zero 2 W reaches USB devices through its OTG host port and a powered hub.
- USB requires the Pi host to enumerate the device first. After Linux reports a new serial device and the module service opens it, the ESP32 sends its protocol `HELLO` message. This is the module's first application-level message.
- Start with one USB serial interface per module. Do not add BLE or Wi-Fi discovery to v0; those transports add pairing, security, and reconnection work.
- CSI camera, HDMI/touch, mass storage, and standard USB HID devices keep their normal Linux interfaces. The protocol is for low-rate controls, sensors, and module management, not video or disk traffic.

## Messages

Use a bounded binary frame so the host service and ESP32 firmware can parse messages without a large runtime or per-device library. A proposed frame is:

```text
COBS(version:u8 | message_type:u8 | sequence:u16le | payload_length:u16le | payload | CRC16-CCITT) | 0x00
```

Compute CRC16-CCITT over the unencoded header and payload, then COBS-encode the frame and terminate it with `0x00`. Set a v0 maximum payload of 128 bytes and reject unsupported versions, oversized frames, malformed fields, and bad CRCs. Fields inside each payload use small numeric IDs and fixed-width integers or length-delimited byte values. Publish a schema table with the protocol implementation.

| Message | Direction | Purpose |
|---|---|---|
| `HELLO` | Module → host | Module ID, firmware version, and supported protocol range; sent first after the serial port opens |
| `HELLO_ACK` | Host → module | Selected protocol version and session options |
| `CAPABILITIES` | Module → host | List of typed, versioned functions offered by the module |
| `COMMAND` | Host → module | Request a capability action, such as set LED or change sampling interval |
| `RESPONSE` | Module → host | Reply to a command using the matching sequence number |
| `EVENT` | Module → host | Report a button transition, joystick axis, or sensor sample without polling |
| `ERROR` | Either direction | Report an unsupported capability, invalid value, or failed operation |
| `GOODBYE` | Either direction | Optional orderly shutdown before a planned disconnect |

Use event-driven reports instead of frequent polling. Include units and value ranges in capability descriptors. Define timeouts and release all held buttons when the USB connection disappears.

Suggested starter capability IDs are `0x0001` button state, `0x0002` signed joystick axis, `0x0010` scalar sensor reading, and `0x0020` LED output. Give each capability a schema version; include axis range or sensor units in its descriptor. Keep unknown IDs harmlessly ignorable.

## Host service behavior

1. Watch for a USB serial device-add event.
2. Open the serial interface with bounded buffers and wait for the module's `HELLO`.
3. Validate the offered protocol range and reply with `HELLO_ACK` selecting one supported version.
4. Read and validate `CAPABILITIES`; show the module ID and understood capabilities.
5. Route common capability IDs to generic handlers. For example, convert joystick button/axis events into Linux input events with `uinput`.
6. On unplug or serial failure, stop commands, release virtual input state, close the device, and remove its UI entries.

Do not execute code, shell commands, or package instructions received from a module. Allow only known capability IDs and validated values. Unknown capabilities should be shown as unsupported and ignored safely.

## Suggested first prototype

Build one ESP32 joystick module with two analog axes and a few buttons. The ESP32 scans its ADC/GPIO locally and sends only changed button states and axis events. A small Linux broker translates those events to a virtual gamepad using the standard `uinput` interface. Demonstrate attach, capability negotiation, live input, unplug cleanup, daemon memory use, and CPU use.

This gives the module a small hardware-specific firmware while keeping the Pi's protocol handling in one service. Add an LED or temperature-sensor module only after the first module works.

## Resource and scope expectations

- The Linux kernel already provides serial drivers for USB CDC and common USB-to-serial bridges; the design avoids installing a separate protocol library and daemon per module family.
- The broker still consumes memory, and the GUI, kernel, and device drivers remain present. Measure the broker's resident memory and CPU use instead of claiming a large resource saving in advance.
- Reusing the same frame format does not make different modules interchangeable. Capability IDs and their behavior must be defined, implemented in firmware, and supported by the host where needed.
- Keep camera frames, storage blocks, and display pixels on their standard high-bandwidth interfaces rather than routing them through this control protocol.

## Later extensions

After USB serial v0 works, consider other transports only if the concept needs them. Keep the message schema independent of the transport, and specify transport-specific discovery, security, maximum payload, and reconnect behavior before claiming that a BLE or Wi-Fi module follows the same protocol.
