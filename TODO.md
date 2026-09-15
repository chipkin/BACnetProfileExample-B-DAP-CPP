# TODO

## DM-DAB-B (Device Management - Dynamic Address Binding - B) not implemented

The B-DAP (Device Address Proxy) profile requires DS-RP-B, DS-WP-B, DM-DDB-B,
DM-DOB-B, and DM-DAB-B. This example implements the first four; **DM-DAB-B is
not implemented**, because the pinned CAS BACnet Stack (`6.x` @ `abd4cee1`,
reports 6.0.21) has no customer-facing export for it.

### What was checked

- `grep -in "proxy\|DAB\|address.binding\|ForeignDevice"
  submodules/cas-bacnet-stack/source/CASBACnetStackDLL.h` at the pin finds only:
  - Device_Address_Binding (DAB) heartbeat tunables
    (`BACnetStackSetting_DABHeartbeatIntervalSeconds`,
    `BACnetStackSetting_DABRangeClusterTolerance`,
    `BACnetStackSetting_DABMaxMissesBeforeStale`) - these configure the stack's
    *own outbound* address resolution (it resolving addresses for devices it
    wants to talk to as a client), not proxying address-binding on behalf of
    other devices.
  - BACnet/SC device-address-proxy property identifiers
    (`BACnetPropertyIdentifier_deviceAddressProxyEnable/Table/Timeout`,
    `maxProxiedIAmsPerSecond`) with **no matching `DllExport`**.
- The actual get/set logic for those proxy properties lives only on the
  internal C++ class `BACnetStackNetworkPortSC`
  (`source/BACnetStackNetworkPortSC.cpp`: `SetDeviceAddressProxyEnable`,
  `SetDeviceAddressProxyTimeout`, `SetSCDeviceAddressProxyEntries`) and
  `BACnetDeviceAddressProxyTableEntry`
  (`source/BACnetDeviceAddressProxyTableEntry.cpp`) - neither is exported in
  `source/CASBACnetStackDLL.h`. `BACnetStack_AddNetworkPortObject`
  (`CASBACnetStackDLL.h:742`) is the only network-port creation export and has
  no SC-proxy-specific variant or parameter for host code to use.
- A comment at `CASBACnetStackDLL.h:6031` references `SetDABHeartbeatBroadcast`
  as the way to configure the heartbeat broadcast destination, but that
  function was removed in Sprint 82 and does not exist anywhere in the header
  (confirmed stale by the stack's own
  `docs/reviews/2026-09-04-public-interface-function-review.md:201,2730`).
- `docs/profiles.md:111` in this examples repo records DM-DAB-B as "flipped"
  for B-DAP via an internal 143-test panel - that is a spec-literal/internal
  test-panel determination made against the stack's own code, not evidence
  that the capability is reachable from the customer DLL surface. It is not.

### What's needed to close this

A customer-facing export equivalent to
`BACnetStack_AddDeviceAddressProxyEntry(...)` /
`BACnetStack_SetDeviceAddressProxyEnabled(networkPortInstance, bool)`, so host
code can enable the proxy behavior and seed/inspect proxy table entries.

Filed: <https://github.com/chipkin/cas-bacnet-stack/issues/2032>

### What this example does instead

Implements DS-RP-B, DS-WP-B, DM-DDB-B, and DM-DOB-B in full (see README
"Verify"), and leaves DM-DAB-B unimplemented rather than fake it. No
`BACNET_STACK_TESTTOOL` code path is used anywhere in this repository.
