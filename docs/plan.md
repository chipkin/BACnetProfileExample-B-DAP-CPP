# Plan (STUB): B-DAP (Device Address Proxy) — C++ example

> **STATUS: STUB.** Seed facts below. Expand from
> [`bacnet-profile-plan-template.md`](../../bacnet-profile-plan-template.md) after the
> sample plans ([B-LD](../../BACnetProfileExample-B-LD-CPP/docs/plan.md),
> [B-BC](../../BACnetProfileExample-B-BC-CPP/docs/plan.md)) are reviewed.

**Profile:** B-DAP · **Family:** Annex L.7 (Miscellaneous) · **Role:** B ·
**Archetype:** Infrastructure · **Difficulty:** 3/5 · **Phase:** B (deferred — infrastructure)

**Thesis:** a device-address proxy — answers Who-Is/Who-Has **on behalf of** other
devices on a network it fronts (e.g. an MS/TP or SC segment), so remote clients can
find them. The interesting code is the proxy/advertise behaviour, not objects.

## Required BIBBs (profiles.md L.7)
`DS-RP-B, DS-WP-B; DM-DDB-B, DM-DOB-B, DM-DAB-B`.

## Services to enable
- ReadProperty (1), WriteProperty (15), baseline discovery, + the proxy function.

## Objects (baseline + )
- Possibly a second Network Port representing the directly-connected network the
  proxy fronts. Minimal application object model beyond the baseline.

## Shared features
- **DEFINE:** F-PROXY (DM-DAB-B device-address-proxy advertise).
- **REUSE:** F-OUTPUTS (B-SA writes).

## Known stack gaps
- Confirm the standard DLL's DM-DAB-B (= DM-DAP-B in Annex-K K.5.37) API and how to
  model "one or more directly connected networks." profiles.md: ✅ S74 ("SC proxy
  implementation satisfies the directly-connected-networks criterion").

## Notes / open questions
- Infrastructure archetype — see master plan §5 and §7 risk 6 (multi-network-port).
  Decide how to represent the proxied network in a single-host example (a virtual
  network? a second port?). Spike the API before writing the code section.
