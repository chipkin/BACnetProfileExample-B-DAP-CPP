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
own CHANGELOG records this under #2346 (in the pinned stack, `6.x` @
`e6de4ffd`): the BACnet/SC Network Port stopped listing
`Device_Address_Proxy_Enable` / `Device_Address_Proxy_Timeout` because "The
stack has never done that proxying: it never answers Who-Is from a
Device_Address_Proxy_Table and never changes routing."

So a customer-facing export alone (a `BACnetStack_*` function to enable the
proxy or seed the table) would **not** close this: there is no proxying
behaviour behind it to switch on. #2032 needs the proxying itself - the
Who-Is handling, the `Device_Address_Proxy_Table`, routing - before this
example can be reworked into a router that proxies.

### What this example does instead

Implements DS-RP-B, DS-WP-B, DM-DDB-B and DM-DOB-B (see README "Verify"),
and leaves DM-DAP-B unimplemented rather than fake it. It does not route and
does not claim NM-RC-B. No `BACNET_STACK_TESTTOOL` code path is used anywhere
in this repository.
