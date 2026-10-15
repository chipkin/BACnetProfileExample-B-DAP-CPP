# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.2] - unreleased

### Changed - honesty pass (2026-10-14)

- **Stack Pin moved `abd4cee1` (6.0.21) -> `6.x` @ `e6de4ffd` (6.0.23)**, per
  docs/adr/0001 (the pin follows the latest `6.x` when an example is updated).
  Not the `6.x` tip `4b27bd56d`: that tip fails to link any customer
  (non-test-tool) build - `RecordTypedGetDecline` is defined only under
  `BACNET_STACK_TESTTOOL` (stack #3850, fixed on `issues/runbook` but not yet
  promoted to `6.x`). `e6de4ffd` is the newest `6.x` commit that links. Builds
  clean (STATIC) with no source change.
- **Protocol_Revision is 30, not 24** - the stack default at this pin. README,
  PICS, TUTORIAL, AGENTS.md and `main.cpp` comments said 24.
- README version banner and sample output, and PICS Application Software
  Version / Firmware Revision, now match what the program reports
  (`APP_VERSION`, the linked stack version, `common/` 3.0.0).
- Series profile table re-synced (B-ACCR and B-DAP marked as Blocked Examples).
- **Now a Blocked Example on stack #2032.** README banner: a B-DAP is a router
  that answers Who-Is for proxied devices (cl. 19.9); today this example behaves
  as a B-SA.
- The BIBB is **DM-DAP-B, Device Address Proxying** (Annex K.5.37), not
  "DM-DAB-B Dynamic Address Binding" (Annex L's table misprints it; Annex K has
  no such BIBB). Renamed throughout, including the Device `Description`.
- TODO.md rewritten: the stack has never implemented device address proxying
  (stack CHANGELOG #2346), so a customer export alone will not fix it; stale
  `abd4cee1` line references dropped.

### Changed

- **Device renamed from the series' colour placeholder "Rainbow" to "Chipkin
  Example B-DAP"** so devices from different examples in the series are
  distinguishable from each other on the same BACnet network - every example
  previously announced the identical Object_Name "Rainbow", which made two
  examples on one subnet indistinguishable by name. Sub-object names (Analog
  Input 1 "Bronze", etc.) are unchanged - only the Device object's name
  changed. `docs/colour-table.md` (series root) updated to match. APP_VERSION
  bumped 1.0.1 -> 1.0.2.

## [1.0.1] - 2026-09-22

### Fixed

- **`Application_Software_Version` (12) and `Firmware_Revision` (44) were
  hardcoded and stale** - both were separate `"1.0.0"` constants
  (`APPLICATION_SOFTWARE_VERSION`, `FIRMWARE_REVISION`) that never moved as
  the example's real version advanced, the same bug already found and fixed
  in the B-SCHUB-CPP example of this series. Fixed: `Application_Software_Version`
  now reads `APP_VERSION` directly (one source of truth, can't drift from
  `--version`'s own banner again). `Firmware_Revision` is now built at
  runtime from the CAS BACnet Stack's own `BACnetStack_GetAPIMajorVersion()`/
  `GetAPIMinorVersion()`/`GetAPIPatchVersion()`/`GetAPIBuildVersion()` (the
  same 4 calls `common/CASExampleHelper.cpp`'s `PrintVersion()` already uses
  for the startup banner), populated once right after `LoadBACnetFunctions()`
  succeeds. Verified with a clean Release build and a code read confirming
  the property-read switch now returns `APP_VERSION` / the runtime-built
  stack version string instead of the old hardcoded literals; a real
  ReadProperty against the running device was not performed this round (see
  commit for details).

### Changed

- Restructured documentation to match the series' new shape: `README.md` is
  now cut down to this example only (no series framing, no generic profile
  explanation, no "Before you ship" table, no "Objects and properties"
  section) with pointers to two new files, `TUTORIAL.md` (extending and
  reviewing the example, carried over verbatim from the old README's
  long-form material) and `docs/PICS.md` (the Protocol Implementation
  Conformance Statement, ANSI/ASHRAE 135 Annex A shape).
- `docs/objects.json` now includes a `Device` entry (previously the generated
  tables omitted it); `docs/PICS.md`'s generated block regenerates with zero
  ⚠ rows.
- Absorbed the README's per-field "Before you ship" guidance into comments
  next to the `CHANGE ALL OF THIS BEFORE YOU SHIP` block in `main.cpp`,
  including the `DEVICE_NAME` uniqueness warning.
- Switched the documented and CI build from a prebuilt **STATIC** library
  (`tools/build-stack-static.sh` + `-DCAS_BACNET_STACK_LINK=STATIC`) to the
  adapter's default **SOURCE** mode: `cmake -B build -S .` /
  `cmake --build build --config Release`, identical to every other command in
  the series. `.github/workflows/release.yml` no longer caches or builds a
  static library, no longer carries per-OS `lib:` matrix entries, asserts
  `CAS_BACNET_STACK_LINK=SOURCE`, records `"link_mode": "SOURCE"` in
  `metrics-*.json`, and packages `TUTORIAL.md` and `docs/PICS.md` alongside
  the binary. The `common/` helper bundled here is v2.5.0 (previously the
  README under-reported it as v2.3.0); the Footprint table still reflects the
  v1.0.0 STATIC-linked release and is noted as due for a refresh under the
  new documented build.
- Corrected `README.md`'s "Expected output" block to match what the example
  actually prints.

## [1.0.0] - 2026-09-15

### Added

- Initial B-DAP (Device Address Proxy) example, implemented against the CAS
  BACnet Stack pinned at `6.x` @ `abd4cee1` (reports 6.0.21), **linked as a
  prebuilt STATIC library** (`-DCAS_BACNET_STACK_LINK=STATIC`, built by
  `tools/build-stack-static.sh`). No DLL is shipped or documented.
- `common/` vendored at **v2.3.0** (see `common/CHANGELOG.md`), byte-identical
  with the rest of the series.
- Implements **DS-RP-B** (ReadProperty), **DS-WP-B** (WriteProperty to three
  commandable outputs via a 16-slot Priority_Array + Relinquish_Default),
  **DM-DDB-B** (Who-Is/I-Am) and **DM-DOB-B** (Who-Has/I-Have).
- Base series object set: Analog Input 1 "Bronze", Binary Input 1 "Emerald",
  Multi-State Input 1 "Hot Pink" (read-only sensors), Analog Output 1
  "Chartreuse", Binary Output 1 "Fuchsia", Multi-State Output 1 "Indigo"
  (commandable outputs), Network Port 1 "Vermilion". Device instance 389021
  ("Rainbow").
- `docs/objects.json`-driven "Objects and properties" reference block, the
  series-wide profile table block, and a `## Footprint` placeholder table in
  the README.
- `.github/workflows/release.yml`: the series' proven template
  (windows-2022 + ubuntu-latest, static-library caching keyed on the stack
  commit, `metrics-*.json` publishing on a version tag).

### Not implemented

- **DM-DAB-B** (Device Management - Dynamic Address Binding - B) - the fifth
  BIBB the B-DAP profile requires. The pinned stack has no customer-facing
  export for device-address-proxy configuration. See `TODO.md` and
  [cas-bacnet-stack#2032](https://github.com/chipkin/cas-bacnet-stack/issues/2032).
