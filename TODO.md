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

### Why the stack cannot do it yet

The CAS BACnet Stack has **never implemented device address proxying**. Its
own CHANGELOG records this under #2346 (`6.x` @ `64b2cfac5`, and still so
at the current Stack Pin `69b6249f`): the BACnet/SC Network Port stopped listing
`Device_Address_Proxy_Enable` / `Device_Address_Proxy_Timeout` because "The
stack has never done that proxying: it never answers Who-Is from a
Device_Address_Proxy_Table and never changes routing."

So a customer-facing export alone (a `BACnetStack_*` function to enable the
proxy or seed the table) would **not** close this: there is no proxying
behaviour behind it to switch on. #2032 needs the proxying itself - the
Who-Is handling, the `Device_Address_Proxy_Table`, routing - before this
example can be reworked into a router that proxies.

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

### Status (2026-10-26, Stack Pin `6.x` @ `69b6249f`)

Still blocked. The stack at this pin carries the data half of a device address
proxy engine (the proxy table, learning devices from their I-Ams, revalidation
and the I-Am budget), but nothing in the stack uses it yet. Read from the
running example at this pin: Device `Device_Address_Proxy_Table` and
`Max_Proxied_I_Ams_Per_Second`, and Network Port `Device_Address_Proxy_Enable`
and `Device_Address_Proxy_Timeout`, all answer Error property /
unknown-property and none is in a `Property_List`; a Who-Is for a range other
than the device's own gets no answer. The router behaviour of clause 19.9.1 is
being built in the stack's own work (stack issue chipkin/cas-bacnet-stack#4560) and has not reached
`6.x` yet. Keeping the table over a restart still needs #2032's export. When
both are on `6.x`, this example still has to become a router (NM-RC-B) before
it can perform B-DAP.

### What this example does instead

Implements DS-RP-B, DS-WP-B, DM-DDB-B and DM-DOB-B (see README "Verify"),
and leaves DM-DAP-B unimplemented rather than fake it. It does not route and
does not claim NM-RC-B. No `BACNET_STACK_TESTTOOL` code path is used anywhere
in this repository.

### Known gaps in this example (ref: `chipkin/cas-bacnet-stack#2032`)

- DM-DAP-B, DM-DDB-A, DM-LM-B and NM-RC-B are not claimed; the device does not
  route. Reworking it into a router needs the stack's proxy engine on `6.x`
  (chipkin/cas-bacnet-stack#4560).
- The BTL run has no FAIL. Rows 10.2.9 to 10.2.12 are skipped (fixtures and
  `servicesExecuted` gates a customer project cannot meet) and 10.2.13 / 10.2.14
  have no script (ref: `chipkin/chipkin-bacnet-testing-tool#735`).
