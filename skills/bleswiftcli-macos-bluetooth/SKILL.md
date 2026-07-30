---
name: bleswiftcli-macos-bluetooth
description: A macOS command-line tool for Bluetooth Low Energy operations—scan, connect, pair, read, write, inspect GATT, and L2CAP channels.
triggers:
  - scan for bluetooth devices on mac
  - connect to a ble peripheral
  - read bluetooth characteristic value
  - write to ble device
  - inspect gatt services
  - pair with bluetooth low energy device
  - set up l2cap channel
  - bluetooth le command line tool
---

# BLESwiftCLI — macOS Bluetooth LE Command-Line Tool

> Skill by [ara.so](https://ara.so) — Devtools Skills collection.

`ble` is a macOS command-line tool for Bluetooth Low Energy operations: scanning, connecting, pairing, reading/writing characteristics and descriptors, inspecting GATT databases, and opening L2CAP channels. Built on BLESwift and swift-argument-parser.

## Installation

### Homebrew

```sh
brew install kylebrowning/tap/ble
```

### Mint

```sh
mint install kylebrowning/BLESwiftCLI
```

### From Source

```sh
swift build -c release
cp .build/release/ble /usr/local/bin/
```

**Permissions**: On first run, macOS prompts for Bluetooth access. Grant it to your terminal app (Terminal, iTerm, etc.) in *System Settings → Privacy & Security → Bluetooth*.

## Core Concepts

- **Peripherals** are identified by UUID or name substring
- **Services** and **characteristics** use 16-bit SIG UUIDs (e.g. `180F`, `2A19`) or full 128-bit UUIDs
- Connections are held open only while the CLI process runs
- Pairing is triggered by accessing encrypted characteristics—no explicit pair API
- Output: data (scan results, values, JSON) → **stdout**; status/progress → **stderr**

## Commands

### Scanning for Devices

Scan for all nearby BLE devices:

```sh
ble scan
```

Scan for specific service (e.g. Heart Rate `180D`) with unlimited time:

```sh
ble scan -s 180D --timeout 0
```

Live RSSI updates with duplicate advertisements:

```sh
ble scan --allow-duplicates
```

Filter by minimum signal strength:

```sh
ble scan --min-rssi -70
```

JSON output for scripting:

```sh
ble scan --json | jq '.name, .rssi, .uuid'
```

**Output format** (interactive): Live table sorted by signal strength, showing name, RSSI (color-coded green/yellow/red), peripheral UUID, advertised services, manufacturer data.

**Output format** (piped/JSON): One line per advertisement event.

### Connecting to a Peripheral

Connect by UUID:

```sh
ble connect 6E400001-B5A3-F393-E0A9-E50E24DCCA9E
```

Connect by name substring (resolves via scan):

```sh
ble connect "Kyle's Sensor"
```

Connect and auto-reconnect on disconnect:

```sh
ble connect mydevice -s 180D --reconnect
```

The connection stays open until Ctrl-C or the process exits. Lifecycle events stream to stderr.

### Pairing with a Device

CoreBluetooth has no explicit pair API. Pairing is triggered by accessing a characteristic that requires encryption:

```sh
ble pair mydevice -s <service-uuid> -c <protected-characteristic-uuid>
```

To trigger pairing via write:

```sh
ble pair mydevice -s FFF0 -c FFF1 --write 0x01
```

This connects, attempts the read/write, and macOS shows the pairing dialog. Approve it to bond the device.

**Unpair**: *System Settings → Bluetooth*, click info button on device, select "Forget This Device".

### Inspecting GATT Database

Enumerate all services, characteristics, and descriptors:

```sh
ble inspect mydevice
```

Include current values of readable characteristics:

```sh
ble inspect mydevice --read
```

JSON output:

```sh
ble inspect mydevice --json | jq
```

**Example output**:

```
Service 180F — Battery
  2A19 — Battery Level  [read, notify] = 0x5A (1 byte, uint 90, "Z")
    Descriptor 2902 — Client Characteristic Configuration
```

### Reading Characteristics

One-time read:

```sh
ble read mydevice -s 180F -c 2A19
```

Subscribe to notifications:

```sh
ble read mydevice -s 180F -c 2A19 --notify
```

Read specific number of notifications:

```sh
ble read mydevice -s 180F -c 2A19 --notify --count 5
```

Read a descriptor:

```sh
ble read mydevice -s 180F -c 2A19 -d 2901
```

**Output**: Hex representation plus interpretations (byte count, little-endian uint, UTF-8 string).

### Writing Characteristics

Write hex value:

```sh
ble write mydevice -s FFF0 -c FFF1 --hex 0x01FF
```

Write string:

```sh
ble write mydevice -s FFF0 -c FFF1 --string "hello"
```

Write from payload file (YAML/JSON):

```sh
ble write mydevice -p command.yaml
```

Write and wait for notification on another characteristic:

```sh
ble write mydevice -p command.yaml --expect-reply-on FFF2
```

Dry-run (print encoded bytes without writing):

```sh
ble write mydevice -p command.yaml --dry-run
```

Write a descriptor:

```sh
ble write mydevice -s FFF0 -c FFF1 -d 2901 --string "label"
```

The tool automatically checks characteristic properties and chooses write-with-response or write-without-response. Use `--without-response` to force it.

### Payload Files

Structured payloads in YAML or JSON. Fields are encoded in order and concatenated.

**Example** (`command.yaml`):

```yaml
service: 180F
characteristic: 2A19
writeType: withResponse  # optional: withResponse | withoutResponse
fields:
  - { type: u8,     value: 1 }
  - { type: u16le,  value: 5000 }
  - { type: i32be,  value: -70 }
  - { type: string, value: "hello" }
  - { type: hex,    value: "DEADBEEF" }
  - { type: pad,    length: 2 }
```

**Supported types**:
- Integers: `u8`, `u16`, `u32`, `u64`, `i8`, `i16`, `i32`, `i64`
- Endianness: `u16le` (little-endian, default), `u16be` (big-endian)
- `string`: UTF-8 encoded
- `hex`: raw hex bytes
- `pad`: zero-fill N bytes

Command-line flags (`-s`, `-c`) override the file's `service`/`characteristic`.

### L2CAP Channels

Open a connection-oriented L2CAP channel:

```sh
ble l2cap mydevice --psm 0x0080
```

Send hex data once, then stream incoming:

```sh
ble l2cap mydevice --psm 128 --send-hex 0x01FF
```

Raw binary output (for piping):

```sh
ble l2cap mydevice --psm 128 --raw > capture.bin
```

The channel stays open until Ctrl-C or the peripheral closes it.

## Common Patterns

### Find and Connect to a Specific Device Type

```sh
# Scan for heart-rate monitors
ble scan -s 180D --timeout 5

# Connect to one by name
ble connect "Polar H10" --reconnect
```

### Read Battery Level

```sh
ble read mydevice -s 180F -c 2A19
```

### Monitor Heart Rate Notifications

```sh
ble read mydevice -s 180D -c 2A37 --notify
```

### Send Multi-Field Command

Create `command.yaml`:

```yaml
service: FFF0
characteristic: FFF1
fields:
  - { type: u8, value: 0x02 }       # Command ID
  - { type: u16le, value: 1000 }    # Parameter
```

Execute:

```sh
ble write mydevice -p command.yaml
```

### Automated Testing / Scripting

```sh
#!/bin/bash
# Scan for devices, filter by service, connect to first match
DEVICE=$(ble scan -s 180D --timeout 5 --json | jq -r '.uuid' | head -n1)
ble connect "$DEVICE" &
sleep 2
ble read "$DEVICE" -s 180D -c 2A37 --notify --count 10
```

### Full GATT Inspection

```sh
# Human-readable with values
ble inspect mydevice --read

# JSON for parsing
ble inspect mydevice --json | jq '.services[].characteristics[].uuid'
```

### Pair and Then Write to Protected Characteristic

```sh
# Trigger pairing
ble pair mydevice -s FFF0 -c FFF1 --write 0x00

# Subsequent writes succeed without re-pairing
ble write mydevice -s FFF0 -c FFF1 --hex 0x0102
```

## Troubleshooting

### "Bluetooth access denied"

Grant Bluetooth permission to your terminal app in *System Settings → Privacy & Security → Bluetooth*.

### "Peripheral not found"

- Ensure the device is powered on and in range
- Try `ble scan` first to confirm it's advertising
- Use the exact UUID or a unique name substring

### "Characteristic does not support write"

The characteristic's properties don't include `write` or `writeWithoutResponse`. Use `ble inspect` to confirm properties.

### "Value exceeds MTU"

The payload is larger than the negotiated maximum write length. Split the data or reduce the payload size. The tool warns about this but doesn't auto-split.

### "Connection timeout"

- Device may be out of range or turned off
- macOS Bluetooth stack may be busy—try toggling Bluetooth off/on in System Settings
- Some devices bond to one host at a time—unpair from other devices

### Pairing dialog doesn't appear

The characteristic isn't marked as requiring encryption. Check the device's GATT specification or try a different characteristic known to be protected.

### No notifications received

- Verify the characteristic supports notify/indicate: `ble inspect mydevice`
- Some devices require enabling notifications via the Client Characteristic Configuration Descriptor (CCCD `2902`)—the tool does this automatically

## Integration with Other Tools

### Pipe to `jq` for JSON filtering

```sh
ble scan --json | jq 'select(.rssi > -60) | {name, uuid, rssi}'
```

### Capture raw L2CAP stream

```sh
ble l2cap mydevice --psm 128 --raw | xxd
```

### Log notifications to file

```sh
ble read mydevice -s 180D -c 2A37 --notify > heart_rate.log
```

### Use in scripts with error handling

```sh
#!/bin/bash
set -e
DEVICE="MyDevice"
if ! ble connect "$DEVICE" --timeout 5 2>&1 | grep -q "Connected"; then
  echo "Failed to connect"
  exit 1
fi
ble write "$DEVICE" -p command.yaml
```

## Environment and Configuration

- **NO_COLOR**: Set to disable colored output
- Piping automatically disables colors and switches to line-by-line output
- Timeout defaults: scan 10s, connect 30s (override with `--timeout`)

## Development and Testing

Build from source:

```sh
git clone https://github.com/kylebrowning/BLESwiftCLI.git
cd BLESwiftCLI
swift build
.build/debug/ble --help
```

Run tests (no hardware required):

```sh
swift test
```

Tests use BLESwift's `FakeCentral` and `FakePeripheral` for logic validation.

## Reference

- **Standard BLE Services**: [Bluetooth SIG GATT Services](https://www.bluetooth.com/specifications/gatt/services/)
  - `180F` = Battery Service
  - `180D` = Heart Rate
  - `180A` = Device Information
- **Standard Characteristics**: [Bluetooth SIG GATT Characteristics](https://www.bluetooth.com/specifications/gatt/characteristics/)
  - `2A19` = Battery Level
  - `2A37` = Heart Rate Measurement
  - `2902` = Client Characteristic Configuration (CCCD)

---

**Key takeaway**: Use `ble scan` to discover, `ble inspect` to explore GATT, `ble read`/`write` for data, and payload files for complex structured writes.
