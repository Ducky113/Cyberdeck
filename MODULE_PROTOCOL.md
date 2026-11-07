# Pocket Workstation OS — Module Protocol v0 Draft

**Status:** proposed wire/API contract; implementation and interoperability tests pending. This revision replaces the earlier unsolicited-HELLO design. The first implementation freezes wire major version 1; v0 describes this document's draft status.

## Scope and transport

The MCU owns sensor drivers, ADC conversion, debounce, and local timing. The host discovers typed capabilities and exposes them to applications. Start with USB serial: native USB CDC or a documented USB-to-UART bridge, using 115200 baud, 8N1, no flow control for the first bridge prototype. Record board-specific DTR/RTS/reset behavior. Do not assume every ESP32 has native USB or that firmware knows when Linux opens a bridge's serial port.

The broker opens only configured candidate interfaces. A generic bridge VID/PID is not sufficient to identify a Pocket module: use a configured physical port or serial identifier and validate the application handshake. Do not probe every unrelated serial device. USB enumeration and the application handshake are separate steps.

Keep standard HID, storage, cameras, display, and audio on existing interfaces. This protocol covers low-rate sensors, controls, and module management.

## Framing

```text
COBS(version:u8 | type:u8 | sequence:u16le | session:u32le |
     payload_length:u16le | payload | crc:u16le) | 0x00
```

- Header: 10 bytes. Maximum payload: 128 bytes. Maximum decoded frame: 140 bytes; maximum encoded data: 141 bytes, followed by one delimiter.
- CRC: CRC-16/CCITT-FALSE; polynomial 0x1021, initial 0xFFFF, no reflection, final XOR 0x0000. CRC covers the unencoded header and payload; serialize the result little-endian. The check value for ASCII `123456789` is 0x29B1.
- Validate frame size, version, payload length, CRC, type, session, and payload schema before using values.
- Discard an overlong or malformed frame through the next delimiter. Expire a partial frame after 500 ms. Counters record errors without unbounded logs.
- Requests use nonzero sequence IDs. Responses echo the request ID. Unsolicited events use sequence 0 and an independent event counter in their payload. Do not reuse an outstanding ID; start a fresh session before exhausting request IDs.
- Session IDs distinguish connection lifetimes, not authentication. Generate a new nonzero session on each handshake/restart and reject traffic belonging to older sessions.

## Discovery and session state

```text
ABSENT → OPENING → HANDSHAKE → DESCRIBING → READY
                     ↓             ↓        ↓
                   FAILED ← timeout/error ← STALE
                     ↓                      ↓
                 bounded retry         teardown/reconnect
```

1. Broker subscribes to device events, enumerates already-connected candidates, and deduplicates both paths.
2. Broker opens a candidate, selects a session ID, and sends `HELLO_REQUEST`. This works after a broker restart without unplugging hardware.
3. Module replies with `HELLO`: echoed session, persistent module ID, boot ID, firmware version, supported application schema versions, and capability count.
4. Broker sends `HELLO_ACK` with selected schema, event-rate limit, and heartbeat/lease settings.
5. Broker requests each capability descriptor by index. Only register a usable device after required descriptors validate.
6. Publish `device_added`, then accept subscriptions/commands. Start a fresh session after timeout, module reboot, or serial failure.

Use a fixed version-1 bootstrap envelope. Unsupported wire versions fail cleanly; do not claim that a version field alone negotiates unknown framing. Duplicate handshake requests for the same session repeat the reply without changing outputs; a new session invalidates old commands and resets outputs to their safe defaults.

Initial timeout policy: 1 s per handshake request, at most 3 attempts, then reconnect backoff of 1, 2, 4, 8, 16, and 30 s. Retry only while the interface exists. Record failed attempts in discovery results. A missing serial node causes immediate teardown.

## Messages

| Message | Direction | Purpose |
|---|---|---|
| HELLO_REQUEST / HELLO / HELLO_ACK | Host / module / host | Establish identity, session, and protocol settings |
| CAPABILITY_REQUEST / CAPABILITY | Host / module | One descriptor per requested index; maximum 32 capabilities per module |
| COMMAND / RESPONSE | Host / module | Read current state, set output, or configure subscription; echo request ID |
| EVENT | Module → host | Typed update, source timestamp, and event counter |
| PING / PONG | Host / module | Check responsiveness and renew session lease |
| ERROR | Either | Bounded status code plus failed request ID where applicable |
| GOODBYE | Either | Optional orderly close; correctness must not depend on receiving it |

Assign numeric message IDs and exact payload layouts in a schema file before implementing either endpoint. Publish known frame vectors and schema tests alongside it. Text labels above describe the contract, not yet-frozen binary enum values.

## Capability descriptors

Each descriptor identifies module-local instance ID, type ID, schema version, readable/writable/event flags, measurement quantity, units, numeric scale and range, supported rate, bounded label, and output default where relevant. Limit encoded descriptors to 128 bytes; reject oversize descriptors rather than silently truncating fields. Prefer explicit numeric units/quantity IDs and fixed-point values on the wire; the SDK can present converted values.

| Type ID | Type | Required semantics |
|---|---|---|
| 0x0001 | Button state | Boolean state, debounce policy; snapshot on subscription |
| 0x0002 | Joystick axis | Signed range, center, deadband; snapshot on subscription |
| 0x0010 | Scalar reading | Quantity (e.g. temperature), unit, scale, validity, sample timestamp |
| 0x0020 | LED output | Absolute on/off value and safe default; avoid toggle commands |

Separate capability type from device instance: two temperature sensors must remain distinguishable. A hardware identity should be stable across reconnects; port names are not identities. Detect duplicate reported IDs and expose an ambiguity error instead of merging devices. Unknown types remain listed as unsupported; known types with unsupported schema versions are not used.

## Reliability and flow control

- Default supported event rate: 50 Hz; negotiate at most 100 Hz for the first prototype and validate total byte rate against the chosen serial link. Do not promise this rate for every capability on every module simultaneously.
- One host request in flight per module simplifies ordering. Initial command timeout is 1 s, with at most one retry using the same sequence. Firmware caches the last 32 completed request responses for the session and rejects a repeated ID with different contents.
- Commands are idempotent absolute operations such as `set_led(false)` and `set_interval(20 ms)`. An unanswered request returns a timeout/unknown-outcome error, never assumed success.
- PING every 1 s; declare stale after 3 s without valid PONG. Module lease expires after 3 s without PING and returns the LED to off. Timing is a prototype policy, not a safety-rated control mechanism.
- Per-module event queues are bounded at 128 entries. Continuous axis/scalar updates may coalesce to the latest value, with drop counts exposed. Button transitions must not silently disappear: overflow forces a full state resynchronization and virtual-input release.
- Include a state snapshot at subscription/reconnection and at least once per second while subscribed. Event counters reveal gaps. A client requests/resumes from a new snapshot after overflow.
- Module-local timestamps cannot be subtracted directly from Linux timestamps. Clock synchronization or external instrumentation is required for physical end-to-end latency.

## Removal and restart

On unplug or stale timeout: invalidate capability handles, fail pending requests, mark cached readings stale, release held virtual buttons, remove/destroy the virtual input device, close the serial connection, and emit `device_removed`. Never return cached data as a new reading.

A broker restart recreates the registry through enumeration/handshake. SDK connections receive a disconnect error and must reconnect; a new registry generation invalidates previous handles. No application command is automatically replayed. Firmware lease expiry handles broker death even if USB remains connected.

## Required validation

Test partial/concatenated frames, wrong length/CRC/version, unknown capabilities, oversized descriptors, repeated requests, duplicate identities, event floods, module reboot, unplug during a command, held-button removal, late responses from old sessions, and broker restart with hardware still attached. A mock helps fault injection; at least one physical MCU must validate USB and firmware behavior.
