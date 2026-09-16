# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

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
