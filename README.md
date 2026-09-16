# BACnet B-DAP (Device Address Proxy) - C++ example

A minimal, copy-paste-friendly example showing how to implement the BACnet
**B-DAP (BACnet Device Address Proxy)** device profile in C++ using the
[CAS BACnet Stack](https://store.chipkin.com/services/stacks/bacnet-stack).
It listens on **BACnet/IP (UDP 47808)**, answers **ReadProperty** requests,
accepts **WriteProperty** to its commandable outputs, and is discoverable via
**Who-Is / I-Am** and **Who-Has / I-Have**.

**[Download a prebuilt binary](https://github.com/chipkin/BACnetProfileExample-B-DAP-CPP/releases)**
(Windows and Linux x64) - or build it yourself, see [Build](#build) below.

- **[TUTORIAL.md](TUTORIAL.md)** - how to extend this example and how to review
  it for conformance. Read it when you start turning this into your own device.
- **[docs/PICS.md](docs/PICS.md)** - the Protocol Implementation Conformance
  Statement: every object, every property, and who answers it.

> **Versions:** this document describes **example v1.0.0**, built and verified
> against **CAS BACnet Stack 6.0.21** (`6.x` @ `abd4cee1`), at
> **Protocol_Revision 24**, with the vendored `common/` helper at **v2.5.0**.
> Running the example prints all three - if what it prints disagrees with this
> line, trust the program and check `CHANGELOG.md`.

B-DAP's required BIBB set - DS-RP-B, DS-WP-B, DM-DDB-B, DM-DOB-B - is
identical to [B-SA (Smart Actuator)](https://github.com/chipkin/BACnetProfileExample-B-SA-CPP)'s,
so the object model and service wiring below are the same: three read-only
sensor inputs plus three commandable outputs a client drives with
WriteProperty. B-DAP additionally requires **DM-DAB-B** (Dynamic Address
Binding - proxying address-binding requests on behalf of other devices);
**this example does not implement it** - see [TODO.md](TODO.md) for why.

## Commandable outputs (demonstrating DS-WP-B)

A BACnet output object's `Present_Value` is not written directly. Instead it is
**commandable**: it is resolved from a 16-slot **`Priority_Array`** plus a
**`Relinquish_Default`**.

- A `WriteProperty(Present_Value, value, priority)` stores `value` in slot
  `priority` (1 = highest .. 16 = lowest). If the client omits the priority,
  BACnet uses 16.
- A `WriteProperty(Present_Value, NULL, priority)` **relinquishes** that slot.
- The stack reports the value in the **highest-priority non-null slot** as the
  effective `Present_Value`; if every slot is null, it reports
  `Relinquish_Default`.

This lets several controllers command the same output at different priorities
(e.g. a manual override at priority 8 beats a normal command at 16) and cleanly
hand control back by writing NULL. The application just stores the array (see the
`Commandable` struct in `main.cpp`); the stack does the priority resolution.

## The device this example creates

```
Device 389021  "Rainbow"   (Vendor 389 - Chipkin Automation Systems)
    │
    ├── Analog Input  1       "Bronze"      Present_Value  21.5    (REAL, degrees Celsius; read-only)
    ├── Binary Input  1       "Emerald"     Present_Value  inactive  (0 = inactive / 1 = active; read-only)
    ├── Multi-State Input 1   "Hot Pink"    Present_Value  1       (state, 1..3; read-only)
    ├── Analog Output 1       "Chartreuse"  Present_Value  20.0    (REAL setpoint; WRITABLE, commandable)
    ├── Binary Output 1       "Fuchsia"     Present_Value  inactive  (0/1; WRITABLE, commandable)
    ├── Multi-State Output 1  "Indigo"      Present_Value  1       (state, 1..3; WRITABLE, commandable)
    └── Network Port 1        "Vermilion"   the BACnet/IP port     (required on every device)
```

The three **input** objects (Bronze, Emerald, Hot Pink) are the shared minimum
every example in this series carries. The three **output** objects (Chartreuse,
Fuchsia, Indigo) demonstrate DS-WP-B; their `Present_Value` starts at the
`Relinquish_Default` (no slots commanded yet). Object names follow this series'
colour-naming convention (Device is always "Rainbow").

## What this example supports

The example implements exactly the capabilities below - and nothing more, which
is the point of a profile example. These capabilities satisfy four of the five
BIBBs the **B-DAP (BACnet Device Address Proxy)** profile requires; DM-DAB-B is
not implemented (see [TODO.md](TODO.md)). Because DS-RP-B/DM-DDB-B/DM-DOB-B are
a superset of the **B-GENERAL** baseline, a conformant device necessarily
satisfies **B-GENERAL** too. That is subsumption, not a second claim: this
repository still claims exactly one profile.

### BIBBs (BACnet Interoperability Building Blocks)

| BIBB | Description | Supported |
|------|-------------|:---------:|
| DS-RP-B | Data Sharing - ReadProperty - B | ✅ |
| DS-WP-B | Data Sharing - WriteProperty - B | ✅ |
| DM-DDB-B | Device Management - Dynamic Device Binding - B | ✅ |
| DM-DOB-B | Device Management - Dynamic Object Binding - B | ✅ |
| DM-DAB-B | Device Management - Dynamic Address Binding - B | ⏳ not implemented — see [TODO.md](TODO.md) |

### Services (executed / B-side)

| Service | Notes |
|---------|-------|
| ReadProperty | Responds to property reads (DS-RP-B). |
| WriteProperty | Accepts writes to the commandable outputs' Present_Value (DS-WP-B). |
| Who-Is / I-Am | Answers Who-Is with I-Am, and broadcasts an I-Am on start-up (DM-DDB-B). |
| Who-Has / I-Have | Answers Who-Has with I-Have (DM-DOB-B). |

### Object types

| Object type | Instance | Name | Access |
|-------------|:--------:|------|--------|
| Device | 389021 | Rainbow | - |
| Analog Input | 1 | Bronze | read-only |
| Binary Input | 1 | Emerald | read-only |
| Multi-State Input | 1 | Hot Pink | read-only |
| Analog Output | 1 | Chartreuse | writable (commandable) |
| Binary Output | 1 | Fuchsia | writable (commandable) |
| Multi-State Output | 1 | Indigo | writable (commandable) |
| Network Port | 1 | Vermilion | - |

Every required property of every object, and who answers it, is in
[docs/PICS.md](docs/PICS.md).

## Requires the CAS BACnet Stack (licensed product)

This example **builds against the CAS BACnet Stack, which is a commercial Chipkin
product** - it is not free or open source, and there is no public/trial build.
The stack is referenced here as the **private** git submodule
`submodules/cas-bacnet-stack`; you can only fetch and build it once you have a CAS
BACnet Stack license and access to that repository.

**To get the CAS BACnet Stack (and access to build this example), contact
Chipkin:** <https://store.chipkin.com/services/stacks/bacnet-stack> or
sales@chipkin.com.

You do not need a stack licence to *read* this example, or to run a
[prebuilt release binary](https://github.com/chipkin/BACnetProfileExample-B-DAP-CPP/releases).
The licence is what lets you *build* it - that is the part the stack submodule
gates.

## What's in this repository

This is a **self-contained** project. It ships:

- `main.cpp` - the example device.
- `common/` - the shared helper (UDP, callbacks, CLI, keyboard) vendored in.
- `CMakeLists.txt` - the build, the same on Windows, Linux, and macOS.
- `docs/PICS.md` - the conformance statement.
- `submodules/cas-bacnet-stack/` - the **CAS BACnet Stack as a git submodule**
  (private; requires a license - see above). Its sources are compiled into the
  executable, so there is no library or DLL to build, ship, or install.

## Prerequisites

- A C++17 compiler (MSVC, GCC, or Clang).
- CMake >= 3.15.
- Git (to fetch the stack submodule).

### Windows

- **C++ compiler** - install
  [Visual Studio Community](https://visualstudio.microsoft.com/downloads/)
  (free) and select the **"Desktop development with C++"** workload.
- **CMake** - from <https://cmake.org/download/>, or `winget install Kitware.CMake`.

### Linux / macOS

- Debian/Ubuntu: `sudo apt install build-essential cmake git`
- macOS: `xcode-select --install` and `brew install cmake`

## Build

CMake only, and the same two commands on every platform:

```bash
git clone --recursive https://github.com/chipkin/BACnetProfileExample-B-DAP-CPP.git
cd BACnetProfileExample-B-DAP-CPP

cmake -B build -S .
cmake --build build --config Release
```

Already cloned without `--recursive`? Run `git submodule update --init --recursive`
first - the build needs the stack submodule.

> **The first build takes a few minutes** - it compiles the entire CAS BACnet
> Stack (~600 source files) into the executable. Rebuilds after that are
> incremental and take seconds.

If your CAS BACnet Stack lives somewhere other than the bundled submodule, point
CMake at it: `cmake -B build -S . -D CAS_STACK_DIR=/path/to/cas-bacnet-stack`.

## Run

```bash
# Linux / macOS
./build/BACnetExampleBDAP

# Windows
.\build\Release\BACnetExampleBDAP.exe
```

Expected output:

```
BACnet B-DAP (Device Address Proxy) Example - C++ v1.0.0
CAS BACnet Stack version: 6.0.21.0
Common helper (common/) version: 2.5.0
FYI: Listening for BACnet/IP on UDP port 47808 (Network Port 1).
TX 21 bytes to 192.168.3.255:47808 (broadcast) (Network Port 1)
FYI: Device 389021 ("Rainbow") ready. Vendor ID 389. Press 'h' for help.
```

The `TX` line is the start-up I-Am the device broadcasts to announce itself. It
goes to the **local subnet broadcast** address (here `192.168.3.255`, computed
from the Network Port's interface), not the global `255.255.255.255`. As clients
talk to the device you'll see `RX ... bytes from ...` and `TX ... bytes to ...`
lines showing the traffic; a WriteProperty to an output also prints a line such
as `WriteProperty: Analog Output 1 (Chartreuse) <- 42.50 @ priority 8`.

The device listens on UDP **47808** (BACnet/IP). Allow that port through your
firewall. To use a different port, pass `--port` (see below).

> **A wall of red `Error:` lines at start-up is expected and is not your bug** -
> it is the stack's own debug logging (the device hearing its own broadcast I-Am,
> and a one-time BACnet/SC UUID notice). [TUTORIAL.md](TUTORIAL.md#troubleshooting)
> explains both.

### Command-line options

| Option | Default | Meaning |
|--------|---------|---------|
| `--port <n>` | `47808` | UDP port to listen on (BACnet/IP). |
| `--deviceID <n>` | `389021` | The device's BACnet instance number (BACnet requires this to be configurable). |
| `--help`, `-h` | - | Show usage and exit. |
| `--version` | - | Print the example, stack, and `common/` helper versions, then exit. |

### Interactive commands

While the example runs, these keys are available:

| Key | Action |
|-----|--------|
| `h` | Show the version information and this command list. |
| `q` | Quit. |
| up arrow | Increase Analog Input 1 (`Bronze`) by 1.1. |
| down arrow | Decrease Analog Input 1 (`Bronze`) by 1.1. |

The up/down keys change the live `Present_Value` of the analog input, so a client
re-reading it sees the new value.

## Verify

Use a BACnet client such as the
[**CAS BACnet Explorer**](https://store.chipkin.com/products/tools/cas-bacnet-explorer):

1. **Discover** - send a **Who-Is**. The device replies with **I-Am** from
   instance **389021** (vendor **389**). It also broadcasts an I-Am at start-up.
2. **Browse the object model** - the device shows eight objects: the Device
   (`Rainbow`), three inputs, three outputs, and the Network Port (`Vermilion`).
   Reading the Device's `Object_List` returns all eight.
3. **Read the Device** - ReadProperty `389021` -> `Object_Name` returns
   `"Rainbow"`; `Protocol_Revision` returns `24`; `Description` returns the
   profile description string.
4. **Read a sensor** - ReadProperty Analog Input `1` -> `Present_Value` returns
   `21.5`; `Units` returns `degrees-Celsius`; `Object_Name` returns `"Bronze"`.
5. **Command an output (the DS-WP-B test)** - ReadProperty Analog Output `1`
   -> `Present_Value` returns `20.0` (its `Relinquish_Default`, nothing commanded
   yet). Now **WriteProperty** Analog Output `1` `Present_Value` = `42.5` at
   priority `8`. Re-read `Present_Value`: it returns `42.5`, and
   `Priority_Array[8]` shows `42.5`. **WriteProperty** `Present_Value` = `NULL`
   at priority `8` to relinquish; `Present_Value` returns to `20.0`. Repeat for
   Binary Output `1` (`"Fuchsia"`, active/inactive) and Multi-State Output `1`
   (`"Indigo"`, state 1..3).
6. **Confirm the profile boundary** - a **WriteProperty** to a read-only *input*
   (e.g. Analog Input 1) is rejected. That is correct: inputs are sensors.
7. **Discover by object (DM-DOB-B)** - send a **Who-Has** naming Analog Input 1
   (`"Bronze"`). The device replies with **I-Have**. `Protocol_Services_Supported`
   read from the Device confirms `who-has` and `i-have` are both enabled.
8. **DM-DAB-B is not exercised here** - this example does not proxy
   address-binding requests for other devices; see [TODO.md](TODO.md) for why.

For a property-by-property review against the conformance statement, see
[TUTORIAL.md](TUTORIAL.md).

## Footprint

Release-build sizes and start-up timing from release
[v1.0.0](https://github.com/chipkin/BACnetProfileExample-B-DAP-CPP/releases/tag/v1.0.0)
(`metrics-windows.json` / `metrics-linux.json`):

<!-- METRICS -->
| Platform | Binary | Size | SHA-256 (prefix) | Start-up to `ready` | Stack commit | Link mode | Compiler |
|---|---|---|---|---|---|---|---|
| Windows x64 (windows-2022) | `BACnetExampleBDAP.exe` | 3,250,688 bytes (~3.1 MiB) | `bea309274ab89868` | 191 ms | `abd4cee1` | STATIC | Visual Studio 17 2022 |
| Linux x64 (ubuntu-latest) | `BACnetExampleBDAP` | 44,528 bytes (~43 KiB) | `2f54ac276f689c16` | 109 ms | `abd4cee1` | STATIC | `/usr/bin/c++` |

v1.0.0 was published from a STATIC-linked build; the documented build is now
SOURCE-mode (see [Build](#build)), so the next release refreshes this table
from that build.

## The BACnet profile example series

<!-- PROFILE-TABLE:BEGIN (generated from cas-bacnet-stack-examples/docs/profile-table.md - do not edit here) -->
The CAS BACnet Stack supports every standardized device profile in ASHRAE 135-2024 Annex L, and there is one example repository per profile. Pick the profile your device claims, then the language you build in.

### Controllers (Annex L.4)

| Profile | C++ | Node.js |
|---|---|---|
| **B-SS** Smart Sensor | [B-SS-CPP](https://github.com/chipkin/BACnetProfileExample-B-SS-CPP) | — |
| **B-SA** Smart Actuator | [B-SA-CPP](https://github.com/chipkin/BACnetProfileExample-B-SA-CPP) | — |
| **B-ASC** Application Specific Controller | [B-ASC-CPP](https://github.com/chipkin/BACnetProfileExample-B-ASC-CPP) | [B-ASC-Node](https://github.com/chipkin/BACnetProfileExample-B-ASC-Node) |
| **B-AAC** Advanced Application Controller | [B-AAC-CPP](https://github.com/chipkin/BACnetProfileExample-B-AAC-CPP) | — |
| **B-BC** Building Controller | [B-BC-CPP](https://github.com/chipkin/BACnetProfileExample-B-BC-CPP) | — |

### Life safety controllers (Annex L.5)

| Profile | C++ | Node.js |
|---|---|---|
| **B-LSC** Life Safety Controller | [B-LSC-CPP](https://github.com/chipkin/BACnetProfileExample-B-LSC-CPP) 🚧 | — |
| **B-ALSC** Advanced Life Safety Controller | [B-ALSC-CPP](https://github.com/chipkin/BACnetProfileExample-B-ALSC-CPP) | — |

### Access control controllers (Annex L.6)

| Profile | C++ | Node.js |
|---|---|---|
| **B-ACC** Access Control Controller | [B-ACC-CPP](https://github.com/chipkin/BACnetProfileExample-B-ACC-CPP) | — |
| **B-AACC** Advanced Access Control Controller | [B-AACC-CPP](https://github.com/chipkin/BACnetProfileExample-B-AACC-CPP) | — |

### Lighting controllers (Annex L.11)

| Profile | C++ | Node.js |
|---|---|---|
| **B-LD** Lighting Device | [B-LD-CPP](https://github.com/chipkin/BACnetProfileExample-B-LD-CPP) | — |
| **B-LS** Lighting Supervisor | [B-LS-CPP](https://github.com/chipkin/BACnetProfileExample-B-LS-CPP) | — |

### Elevator controllers (Annex L.13)

| Profile | C++ | Node.js |
|---|---|---|
| **B-EM** Elevator Monitor | [B-EM-CPP](https://github.com/chipkin/BACnetProfileExample-B-EM-CPP) | — |
| **B-EC** Elevator Controller | [B-EC-CPP](https://github.com/chipkin/BACnetProfileExample-B-EC-CPP) | — |
| **B-AEC** Advanced Elevator Controller | [B-AEC-CPP](https://github.com/chipkin/BACnetProfileExample-B-AEC-CPP) | — |

### Authentication and authorization (Annex L.14)

| Profile | C++ | Node.js |
|---|---|---|
| **B-AS** Authorization Server | [B-AS-CPP](https://github.com/chipkin/BACnetProfileExample-B-AS-CPP) | — |

### Miscellaneous (Annex L.7, combinable with any one family)

| Profile | C++ | Node.js |
|---|---|---|
| **B-BBMD** Broadcast Management Device | [B-BBMD-CPP](https://github.com/chipkin/BACnetProfileExample-B-BBMD-CPP) | — |
| **B-ACDC** Access Control Door Controller | [B-ACDC-CPP](https://github.com/chipkin/BACnetProfileExample-B-ACDC-CPP) | — |
| **B-ACCR** Access Control Credential Reader | [B-ACCR-CPP](https://github.com/chipkin/BACnetProfileExample-B-ACCR-CPP) | — |
| **B-RTR** Router | [B-RTR-CPP](https://github.com/chipkin/BACnetProfileExample-B-RTR-CPP) | — |
| **B-GW** Gateway | [B-GW-CPP](https://github.com/chipkin/BACnetProfileExample-B-GW-CPP) | — |
| **B-DAP** Device Address Proxy | [B-DAP-CPP](https://github.com/chipkin/BACnetProfileExample-B-DAP-CPP) | — |
| **B-SCHUB** BACnet/SC Hub | [B-SCHUB-CPP](https://github.com/chipkin/BACnetProfileExample-B-SCHUB-CPP) | — |
| **B-GENERAL** General device (Annex L.8) | *(satisfied by every example above)* | — |

### Operator interfaces and workstations (Annex L.1–L.3, L.9–L.10, L.12)

Client-side profiles.

| Profile | C++ | Node.js |
|---|---|---|
| **B-OD** Operator Display | [B-OD-CPP](https://github.com/chipkin/BACnetProfileExample-B-OD-CPP) | — |
| **B-OWS** Operator Workstation | planned | — |
| **B-AWS** Advanced Operator Workstation | planned | — |
| **B-XAWS** Extended Advanced Operator Workstation | planned | — |
| **B-LSAP** Life Safety Annunciator Panel | planned | — |
| **B-LSWS** Life Safety Workstation | planned | — |
| **B-ALSWS** Advanced Life Safety Workstation | planned | — |
| **B-ACSD** Access Control Security Display | planned | — |
| **B-ACWS** Access Control Workstation | planned | — |
| **B-AACWS** Advanced Access Control Workstation | planned | — |
| **B-LOD** Lighting Operator Display | planned | — |
| **B-ALWS** Advanced Lighting Workstation | planned | — |
| **B-LCS** Lighting Control Station | planned | — |
| **B-ALCS** Advanced Lighting Control Station | planned | — |
| **B-ED** Elevator Display | planned | — |
| **B-EWS** Elevator Workstation | planned | — |
| **B-AEWS** Advanced Elevator Workstation | planned | — |

🚧 = in progress. Profile definitions: ANSI/ASHRAE 135-2024 Annex L. BIBB definitions: Annex K. Get the stack: <https://store.chipkin.com/services/stacks/bacnet-stack>.
<!-- PROFILE-TABLE:END -->

## References

- **ANSI/ASHRAE Standard 135** (BACnet) - the protocol standard. Object model
  (Clause 12), services (Clause 15), BACnet/IP (Annex J), device profiles
  (Annex L). Purchase / preview via the [ASHRAE store](https://www.ashrae.org/technical-resources/standards-and-guidelines).
- **What is BACnet?** - Chipkin's introduction:
  <https://docs.chipkin.com/protocols/bacnet/>.
- **CAS BACnet Stack** - product page and documentation:
  <https://store.chipkin.com/services/stacks/bacnet-stack>.
- **CAS BACnet Explorer** - client for testing this device:
  <https://store.chipkin.com/products/tools/cas-bacnet-explorer>.
- **B-SA (Smart Actuator) example** - the profile this one's object model and
  service wiring mirror: <https://github.com/chipkin/BACnetProfileExample-B-SA-CPP>.
- **Shared helper used by this example** - [`common/README.md`](common/README.md).

See also [TUTORIAL.md](TUTORIAL.md), [docs/PICS.md](docs/PICS.md),
[TODO.md](TODO.md), [CHANGELOG.md](CHANGELOG.md), and [AGENTS.md](AGENTS.md).
