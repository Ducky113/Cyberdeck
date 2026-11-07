# Pocket Workstation OS — Module Service and SDK

**Status:** implementation design. Package names, commands, and examples below are proposed interfaces, not working software.

## User experience

1. Boot the project image and open its terminal or editor.
2. Connect a supported sensor/LED module.
3. The service discovers its capabilities; `pocket-modules list` shows them.
4. Run the supplied Python example without installing a sensor-specific library or selecting a serial port.
5. Unplugging produces a clear device-loss error. Reconnecting makes a new valid handle available.

This works offline because the image contains the common service and handlers. The MCU carries the sensor-specific driver. Library installation belongs to image/package management, not the hot-plug path.

## Package and process design

`pocket-moduled` is a C++ daemon managed by systemd. Its event loop monitors device events, serial ports, client sockets, and timers without one polling process per module. Begin with one process and statically packaged capability handlers. Measure overhead before considering dynamic plugins.

Run under a dedicated service user. Give it access only to configured candidate serial nodes and, if controller mapping is enabled, `/dev/uinput`. Authorized local users connect through a group-accessible socket at `/run/pocket-moduled/control.sock`; the socket is not world-writable. Editor programs do not run as root. Package installation owns group setup, device rules, the runtime directory, and the unit file.

Use service restart-on-failure with a restart limit, bounded journal output, and a separate configuration directory. Startup registers the device monitor before enumerating present candidates. Do not implement the daemon inside a udev `RUN` rule. Hardware code supplied by a module is never executed on the host.

## Local API

Use a Unix-domain stream socket with newline-delimited JSON for the initial host API. This is distinct from the bounded binary MCU protocol. JSON simplifies inspecting traces and writing clients; measure its cost rather than assuming it is free.

- Version 1 envelopes carry an API version, request ID, method/event name, and parameters/result/error.
- Maximum message: 4096 bytes including delimiter. Handle partial reads and several messages arriving together; reject overlong lines.
- Initial limits: 8 modules, 16 clients, 128 queued messages per client, and one outstanding request per client. Paginate lists so results fit the message bound.
- Each reply echoes the request ID. Subscriptions also receive events with stream counters and registry generation.
- Socket loss fails outstanding requests. Clients reconnect explicitly, list devices again, and reacquire handles.
- Share sensor samples among readers. Permit one writer lease per output capability; another writer receives `Busy`. Release the lease and restore the default when the client disconnects or its lease expires.
- Bound per-client request rates; reject excess requests with `Busy` rather than allowing unlimited backlog.
- Slow readers receive an overflow/resync indication; if that cannot be queued, disconnect them. Never silently present an incomplete stream as complete.

| Method | Meaning |
|---|---|
| `list_devices` | Paged identity, connection status, firmware, and supported capabilities |
| `list_capabilities` | Paged descriptors, filterable by type/quantity/unit |
| `read` | Fresh reading within a bounded timeout, or explicit timeout/stale error |
| `subscribe` / `unsubscribe` | Stream snapshots/updates at a negotiated rate |
| `acquire_output` / `release_output` | Exclusive bounded writer lease; SDK renews while active |
| `write` | Validated absolute output value under the writer lease |
| `watch_devices` | Registry snapshot followed by add/remove/change notifications |
| `get_stats` | Drops, errors, reconnects, queue depth, and module health |

A capability handle includes module ID, capability instance, and registry/session generation. Do not select the first matching sensor silently when several are present. Return candidates or `AmbiguousDevice`, and let the application use a persistent ID.

Define errors consistently: `NoDevice`, `AmbiguousDevice`, `DeviceDisconnected`, `ServiceDisconnected`, `UnsupportedCapability`, `InvalidValue`, `PermissionDenied`, `Busy`, `Timeout`, and `Overflow`. Cached readings carry age/validity and require an explicit cache request; `read` must not fabricate a fresh sample.

## Python programming contract

The image installs `python3-pocket-modules` for its bundled system interpreter, with typing information, docstrings, offline examples, and `py.typed` metadata. The SDK communicates with the broker; it does not open the device directly.

Illustrative proposed API, to implement against the contract above:

```python
from pocket_modules import Client, DeviceDisconnected

with Client() as modules:
    # Requires exactly one match; otherwise returns a clear selection error.
    sensor = modules.require("scalar", quantity="temperature", unit="degC")
    try:
        sample = sensor.read(timeout=1.0)
        print(sample.value, sample.unit)
    except DeviceDisconnected:
        print("Reconnect the sensor and select it again.")
```

The `require` operation wraps capability enumeration/selection; it is not another wire-protocol message. When several sensors match, use `modules.list(...)` and select by module ID. `read` returns value, unit, validity, host timestamp, and source timestamp where provided. Source timestamps remain in their documented clock domain.

An output example acquires a writer lease in a context manager, sets an absolute LED state, and restores the descriptor's default on exit. A sensor logger subscribes at a selected rate and writes data with units and timestamps to a normal file. Firmware protects against host failure using its separate lease timeout.

## Editor integration

An ordinary editor runs these programs with the bundled interpreter. Type hints provide API completion where the editor supports it; the CLI and a small inspector provide live device discovery. This does not require writing a new editor or language server.

Document the interpreter path and a virtual-environment recipe. An isolated virtual environment does not automatically inherit system-installed packages; use the packaged wheel or explicitly enable system-site packages. Do not bypass distribution Python package management globally.

A dedicated editor panel is optional. Generic applications use the SDK; applications expecting standard controllers receive normal Linux input through the broker's `uinput` adapter. Arbitrary sensor types do not automatically become standard kernel devices.

## Capability reuse and implementation order

1. Scalar sensor plus LED: discovery, fresh reads, validated writes, CLI, Python SDK, and removal.
2. A second scalar implementation or mock: the same program works without sensor-specific host changes.
3. Controller example: buttons/axes plus `uinput`, gaming, state snapshots, and held-button cleanup.
4. Optional C/C++ library over the same IPC contract, followed by editor UI conveniences.

Keep transport, registry, and capability handlers separate in source. New firmware for an existing schema should need no host update. New capability semantics require a reviewed package update. Preserve explicit Unsupported status instead of guessing how unfamiliar descriptors work.

## Evidence

Ship CLI traces, example source, package manifests, protocol tests, and a real MCU demonstration. Measure broker RSS/CPU separately from Python SDK/client cost and whole-system cost. Compare direct serial access only as an optional measurement baseline; a shared broker is a consistency/lifecycle design choice, not an assumed RAM saving.
