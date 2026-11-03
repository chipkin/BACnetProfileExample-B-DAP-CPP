# TODO

## Blocked: DM-DAP-B (Device Management - Device Address Proxying - B)

This is a **Blocked Example**, blocked on
[chipkin/cas-bacnet-stack#2032](https://github.com/chipkin/cas-bacnet-stack/issues/2032).
Today it behaves as a B-SA: a single, non-routing BACnet/IP device. It does
not perform the B-DAP Profile Function.

### What B-DAP needs

- **The profile.** Annex L.7.7: "A B-DAP is a BACnet network router that
  provides device address proxying for one or more directly connected
  networks."
- **The BIBB is DM-DAP-B, Device Address Proxying** (Annex K.5.37), not
  "Dynamic Address Binding". (The Annex L profile table in 135-2024 prints the
  entry as "DM-DAB-B"; Annex K defines no BIBB by that name, and K.5.37
  DM-DAP-B is the one that matches the profile's description. Earlier versions
  of this file used the misprint.) K.5.37: "The B device implements proxying
  functionality and provides I-Am messages on behalf of proxied devices." It
  also requires the `Device_Address_Proxy_Enable` property of the Network
  Port, the `Device_Address_Proxy_Table` property of the Device (at least 128
  entries per proxied network), execution of ReadRange, and NM-RC-B.
- **The behaviour** (clause 19.9.1.1): when `Device_Address_Proxy_Enable` is
  TRUE, the router "shall respond with a unicast I-Am unconfirmed request for
  each device on the proxied network that is in the Device object's
  Device_Address_Proxy_Table and matches the device range parameters of a
  Who-Is request", as if the proxied device had sent it.

### What the stack does and does not do yet (at v1.1.1, Stack Pin `6.x` @ `1e7dffe2b`)

At the previous release's Stack Pin (`6.x` @ `69b6249f`) the CAS BACnet Stack
did not perform device address proxying. The Stack Pin of this release does
([chipkin/cas-bacnet-stack#4560](https://github.com/chipkin/cas-bacnet-stack/issues/4560),
closed): a device whose own router is enabled (two or more router ports)
performs the clause 19.9.1 proxying. Its Device gains
`Device_Address_Proxy_Table` and `Max_Proxied_I_Ams_Per_Second`, each Network
Port gains `Device_Address_Proxy_Enable` (default FALSE) and
`Device_Address_Proxy_Timeout`, and a port proxies once its own
`Device_Address_Proxy_Enable` is TRUE.

What is still missing before this example can perform B-DAP:

- **This example is not a router.** It has one BACnet/IP port and does not
  claim NM-RC-B, so the stack's proxying never switches on here. Reworking it
  into a router with a proxied network is the next step in this repository.
- **Keeping the proxy table over a restart** needs the customer export asked
  for in #2032, which is still open.

### Status (2026-10-15)

The stack owner ruled on #2032 in the #2159 session-D pass (stack PR #3937,
commit `aeaa1a37e`, in this Stack Pin): #2032 is **split into its own work
package** and stays open. #3937 only removed stale header text that named the
long-gone `SetDABHeartbeatBroadcast`; it added no export and no proxying. The
package proposed in
[#2032's comments](https://github.com/chipkin/cas-bacnet-stack/issues/2032) is a
device-address-proxy engine in the network layer plus customer exports to
configure it and restore its table, and it covers everything K.5.37 asks for:
`Device_Address_Proxy_Enable` / `_Timeout` on every Network Port type,
`Device_Address_Proxy_Table` and `Max_Proxied_I_Ams_Per_Second` on the Device,
and the clause 19.9.1 behaviour. This example stays blocked on #2032 until that
package lands; no separate child issue has been filed.

### Status (2026-11-02, v1.1.1, Stack Pin `6.x` @ `1e7dffe2b`)

Still blocked on #2032. The router behaviour of clause 19.9.1 (#4560) is in
this Stack Pin, but it applies only to a device that routes, and this example
does not. Because the example is not a router, its Device and Network Port are
expected to list none of the proxy properties and a Who-Is for a range other
than the device's own to get no answer, as they did at `69b6249f`; that was
not re-read from the running example at this pin.

### What this example does instead

Implements DS-RP-B, DS-WP-B, DM-DDB-B and DM-DOB-B (see README "Verify"),
and leaves DM-DAP-B unimplemented rather than fake it. It does not route and
does not claim NM-RC-B. No `BACNET_STACK_TESTTOOL` code path is used anywhere
in this repository.

### Known gaps in this example (ref: `chipkin/cas-bacnet-stack#2032`)

- DM-DAP-B, DM-DDB-A, DM-LM-B and NM-RC-B are not claimed; the device does not
  route. The stack's router proxying
  (chipkin/cas-bacnet-stack#4560, closed) is in this Stack Pin; reworking the
  example into a router that uses it is not done yet.
- The BTL run has no FAIL. Rows 10.2.9 to 10.2.12 are skipped (fixtures and
  `servicesExecuted` gates a customer project cannot meet) and 10.2.13 / 10.2.14
  have no script in the BTL test set.
