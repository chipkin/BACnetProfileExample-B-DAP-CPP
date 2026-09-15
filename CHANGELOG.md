# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - unreleased

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
