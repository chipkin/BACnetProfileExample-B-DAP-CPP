# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [1.1.1] - 2026-11-02

Built on CAS BACnet Stack 6.x (`1e7dffe2b`).

This is still a **Blocked Example**, blocked on
[chipkin/cas-bacnet-stack#2032](https://github.com/chipkin/cas-bacnet-stack/issues/2032).
It behaves as a B-SA: a single, non-routing BACnet/IP device.

### Changed

- **CAS BACnet Stack updated** to 6.x (`1e7dffe2b`). At this version the stack
  no longer gives the example an address for a broadcast it sends on its own,
  such as the I-Am that answers a Who-Is; the vendored helper chooses it (next
  item). This stack version also performs device address proxying when the
  stack is itself a router
  ([#4560](https://github.com/chipkin/cas-bacnet-stack/issues/4560)); this
  example is not a router, so its behaviour does not change.
- **Vendored `common/` updated to 3.6.1.** A broadcast the stack sends without
  a destination address (a network layer message, or an unconfirmed I-Am or
  I-Have) is broadcast on the subnet and also sent unicast to the last client
  heard from on another UDP port of the same host, so a tool running beside
  the example still hears the answer. Event notifications and BBMD-forwarded
  messages are not copied. See `common/CHANGELOG.md`.
- **README**: the Blocked Example note says what the stack now does and what
  is still missing; the profile example table is the same in every example,
  with the five blocked examples marked with their open stack issue.
- **TODO.md** brought up to date for this version.

## [1.1.0] - 2026-10-27

This is a **Blocked Example**. A B-DAP is a router that answers Who-Is on behalf of
the devices on a proxied network (ANSI/ASHRAE 135-2024 L.7.7 and cl. 19.9). The CAS
BACnet Stack does not implement that proxying, so this example cannot perform the
B-DAP Profile Function. See the note at the top of the README.

- **Does today:** a single, non-routing BACnet/IP device with three sensor inputs and
  three commandable outputs. It answers ReadProperty, accepts WriteProperty to the
  outputs, and answers Who-Is/I-Am and Who-Has/I-Have.
- **Does not do today:** it does not route and proxies nothing, so it behaves as a
  B-SA, not a B-DAP. Device Address Proxying (DM-DAP-B) is not claimed.

Built on CAS BACnet Stack 6.x (`69b6249f`, 6.0.23).

### Added

- **Automated discovery self-test** under `tests/` reads every property of every
  object.
- **TUTORIAL.md** (extending and reviewing the example, troubleshooting) and
  **docs/PICS.md** (Annex A style).

### Changed

- **Device renamed** from "Rainbow" to "Chipkin Example B-DAP" so examples on one
  network can be told apart. Object names such as "Bronze" are unchanged.
- **Build now compiles the stack source directly into the executable**:
  `cmake -B build -S .` then `cmake --build build --config Release` on every
  platform; the separate static-library step is gone.
- **Protocol_Revision is 30** (the documentation said 24), and the PICS now states
  that segmentation is `segmented-both` (Max_Segments_Accepted 16).
- **The missing BIBB is named correctly:** DM-DAP-B, Device Address Proxying
  (Annex K.5.37), not "DM-DAB-B".

### Fixed

- **The Device `Description`** no longer reads as an empty string.
- **Application_Software_Version and Firmware_Revision** report the real build
  version and stack version instead of a fixed "1.0.0".
- **Requests no longer go unanswered on Windows** in rare timing cases (shared
  `common/` 3.4.1).
- **README "Expected output"** matches what the example prints.

## [1.0.0] - 2026-09-15

### Added

- Initial **B-DAP (Device Address Proxy)** example in C++, built on CAS BACnet Stack
  6.x (`abd4cee1`, 6.0.21), linked as a static library.
- **DS-RP-B** (ReadProperty), **DS-WP-B** (WriteProperty to three commandable
  outputs with a 16-slot priority array and relinquish default), **DM-DDB-B**
  (Who-Is/I-Am) and **DM-DOB-B** (Who-Has/I-Have).
- Objects: Analog Input 1 "Bronze", Binary Input 1 "Emerald", Multi-State Input 1
  "Hot Pink", Analog Output 1 "Chartreuse", Binary Output 1 "Fuchsia", Multi-State
  Output 1 "Indigo" and Network Port 1 "Vermilion"; Device instance 389021.

### Not implemented

- **Device address proxying** (the profile's fifth BIBB). The stack has no
  customer-facing support for it. See [TODO.md](TODO.md).
