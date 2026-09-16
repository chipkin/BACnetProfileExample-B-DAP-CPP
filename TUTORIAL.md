# Tutorial - extending and reviewing the B-DAP example

[README.md](README.md) says what this example *is*. This document is the *how*:
how to extend it into your own device, who serves which property, how to review
the result for conformance, and what goes wrong when you get it subtly right.

Read this once before you start changing `main.cpp`. The most expensive mistake
in this example is silent, and the section it lives in is
[Add a second analog input](#add-a-second-analog-input).

- [Extending the example](#extending-the-example)
- [What each object type needs you to serve](#what-each-object-type-needs-you-to-serve)
- [Who serves what: the application or the stack?](#who-serves-what-the-application-or-the-stack)
- [Reviewing your device](#reviewing-your-device)
- [Troubleshooting](#troubleshooting)

## Extending the example

The example is intentionally small so it's easy to change.

**Change a value or name** - edit the constants / callbacks in `main.cpp` (e.g.
the `Relinquish_Default` of an output in its `Commandable` initializer, or the
`"Chartreuse"` string in `GetPropertyCharString`).

**Change the device identity before you ship** - vendor ID, vendor name, model
name, description, firmware revision and device name are all in the
`CHANGE ALL OF THIS BEFORE YOU SHIP` block at the top of `main.cpp`, with a
per-field note on each saying what to change it to. That block is the
authoritative checklist; it is in the source rather than here so it cannot be
skipped by someone who only reads the code.

### Add a second analog input

Read this whole recipe before starting — the last step is the one that is easy
to miss and the one BTL will fail you for.

> **Why there are four edits, not three — and why skipping one is SILENT.**
> Most of the `GetProperty*` callbacks match on **both** object type *and*
> instance (`objectInstance == ANALOG_INPUT_INSTANCE`), so a new instance falls
> through every one of them. `GetPropertyBool` is the exception: it matches on
> type only, so `Out_Of_Service` works for a new instance for free.
>
> Here is the part that matters, and it is the opposite of what most people
> assume: falling through a callback does **not** reliably produce an
> error. The stack errors only for the few properties it refuses to invent —
> `Present_Value`, `Number_Of_States`, `Relinquish_Default`, `Local_Date`,
> `Local_Time`, and a Network Port's `APDU_Length`. For everything else it
> **silently substitutes a default**:
>
> | Property | If you forget to serve it | Loud? |
> |---|---|:--:|
> | `Present_Value` | Error (`read-access-denied`) | yes |
> | `Object_Name` | reads back as the string **`"undefined"`** | **no** |
> | `Units` | reads back as **`no-units` (95)** | **no** |
> | `Out_Of_Service` | served on type alone — works by accident | n/a |
>
> **Doesn't the `errorCode` out-parameter fix this?** Only if you use it, and
> only where it is right to. Each `GetProperty*` callback ends with a
> `uint32_t* errorCode` that the stack presets to `success` and reads only when
> you return `false`, so you *can* turn any decline into a chosen BACnet error.
> But ending every callback with `*errorCode = unknown-property` breaks the
> device: the stack's decline-and-fabricate path is what answers required
> properties an application is not expected to serve — the Device's
> `Max_APDU_Length_Accepted`, `APDU_Timeout` and `Number_Of_APDU_Retries` among
> them. Name an error on the catch-all and those start failing instead of
> answering. This example never has a case where it knows a read is wrong (no
> out-of-range array index), so every `Get*` callback declines with
> `errorCode` left alone throughout - but the `Set*` callbacks DO set it, for
> an out-of-range WriteProperty value (see
> [Add a second commandable output](#add-a-second-commandable-output)).
>
> It is worse than "wrong value": the object's `Property_List` **still advertises
> `Units` (117)**. So the object actively claims to have the property, and then
> answers with a default. Nothing on the wire says you forgot anything.
>
> So a half-added object does not look broken; it looks **healthy**. Add two of
> them and both report `Object_Name "undefined"` — duplicate object names inside
> one device, which is a spec violation and a hard BTL failure that every scan
> tool will render as a perfectly good object. **"It scanned OK" is exactly the
> failure mode, not evidence against it.**

```cpp
// 1) a new instance number (in section 1).
//    Naming: a second object of a type is "<Colour> 2" - so Analog Input 2 is
//    "Bronze 2", NOT a new colour. Each object TYPE owns one colour series-wide.
static const uint32_t ANALOG_INPUT_2_INSTANCE = 2;   // "Bronze 2"
static float g_analogInput2Value = 23.1f;            // its live value

// 2) add the object (in main, next to the other BACnetStack_AddObject calls).
//    Check the return, like every other stack call in this file.
if (!BACnetStack_AddObject(g_deviceInstance, OBJECT_TYPE_ANALOG_INPUT, ANALOG_INPUT_2_INSTANCE)) {
    printf("Error: Failed to add Analog Input 2 (Bronze 2).\n");
    return 1;
}

// 3) serve its Present_Value + Object_Name:
//    GetPropertyReal:        AI/2 + Present_Value -> *value = g_analogInput2Value;
//    GetPropertyCharString:  AI/2 + Object_Name   -> "Bronze 2"

// 4) DO NOT SKIP: serve its Units, in GetPropertyEnumerated.
//    Units is a REQUIRED property of an Analog Input. The existing check reads
//    `objectInstance == ANALOG_INPUT_INSTANCE`, which is instance 1 - so without
//    this, reading Analog Input 2's Units returns no-units and the object is
//    NON-CONFORMANT. It will still appear in the Object_List and its
//    Present_Value will read back perfectly, so the device looks healthy right
//    up until BTL certification.
//    GetPropertyEnumerated:  AI/2 + Units -> *value = ENGINEERING_UNITS_DEGREES_CELSIUS;
```

Then re-run the README's Verify steps **against Analog Input 2**, not just Analog
Input 1 — read every required property and **diff it against Analog Input 1**.
Any property that comes back `"undefined"`, `no-units`, or `0` where object 1
returns something real is a step you missed. Because the failure is silent (see
the table above), this diff is the only thing that catches it.

### Add a second commandable output

The output path is the parallel of the input recipe above, and it has one
extra step because outputs are commandable. To add a second Analog Output:

1. A new instance constant and a `Commandable` for it (mirror
   `g_analogOutput`).
2. Teach `GetCommandable()` about the new type+instance pair - the existing
   check matches instance 1 only, so a second output falls through it exactly
   like an unhandled input falls through the `Get*` callbacks above, with the
   same silent consequence: its `Priority_Array`/`Relinquish_Default` reads
   back at the datatype default instead of erroring.
3. `BACnetStack_AddObject` it in `main`, next to the other outputs.
4. Add it to the `outputs[]` table in `main` so the commandable-setup loop
   enables its `Priority_Array`, `Relinquish_Default`, and writable
   `Present_Value`. Skip this and WriteProperty to the new output is rejected
   outright - the one case in this file where a missed step IS loud, because
   `IsPropertyCommandable()` (see the loop's comment in `main.cpp`) requires
   both properties enabled for an optionally-commandable type. (Analog/Binary/
   Multi-State Output are commandable unconditionally in the stack, so this
   step is actually redundant for THIS type - but omit it on a Value type you
   add later and the write is silently rejected instead.)
5. Give its `Set*` callback the same out-of-range validation as the existing
   one (`ERROR_CODE_VALUE_OUT_OF_RANGE`) - accepting an out-of-band write
   silently is its own non-conformance.

Then read back every required property of the new output and **diff it
against the existing one of that type**, exactly as for the input recipe - the
same silent-default trap applies.

Going beyond read/write (COV, alarms, scheduling) means implementing a richer
profile - see the series table in [README.md](README.md).

## What each object type needs you to serve

The application must serve every REQUIRED property the stack does not generate.
It differs per type — this is the checklist, so you do not have to infer it:

| Object type | You must serve | Plus |
|---|---|---|
| Analog Input | `Present_Value` (Real), `Object_Name`, `Units` | — |
| Binary Input | `Present_Value` (Enumerated), `Object_Name` | `Polarity` |
| Multi-State Input | `Present_Value` (Unsigned), `Object_Name` | `Number_Of_States` |
| Analog Output | `Object_Name`, `Units`, `Relinquish_Default`, the `Commandable` slots | `Priority_Array` (stack-computed from the slots) |
| Binary Output | `Object_Name`, `Polarity`, `Relinquish_Default`, the `Commandable` slots | `Priority_Array` (stack-computed) |
| Multi-State Output | `Object_Name`, `Number_Of_States`, `Relinquish_Default`, the `Commandable` slots | `Priority_Array` (stack-computed) |

An **output**'s `Present_Value` is *not* served directly — the stack computes
it from the `Priority_Array` slots your `GetPropertyBool`/typed getters return
(see the `Commandable` struct in `main.cpp`). It is also the one place the
stack, not the app, serves a *writable* property (see the next section).

## Who serves what: the application or the stack?

The single most common question when reading this file is "who answers this
property?" For Analog Input 1 (read-only):

| Property | Served by | How |
|---|---|---|
| `Object_Identifier` | **stack** | generated from the object you added |
| `Object_Type` | **stack** | generated |
| `Object_List` | **stack** | generated (Device object) |
| `Property_List` | **stack** | generated |
| `Status_Flags` | **stack** | generated |
| `Event_State` | **stack**, sort of | no intrinsic alarming here, so nothing serves it — it reads `normal` only because `normal` is the enumeration's zero value and the stack substitutes a datatype default. Correct by coincidence, not design. |
| `Out_Of_Service` | **you** | `GetPropertyBool` — matched on object **type only** |
| `Present_Value` | **you** | `GetPropertyReal` |
| `Object_Name` | **you** | `GetPropertyCharString` |
| `Units` | **you** | `GetPropertyEnumerated` |

For Analog Output 1 (commandable, writable), the picture changes for two rows:

| Property | Served by | How |
|---|---|---|
| `Present_Value` | **stack** | computed from the `Priority_Array` slots below - **writable** |
| `Priority_Array` | **stack** | the stack asks your `GetPropertyBool`/typed getters, per slot, whether each of the 16 slots is set and what it holds; it assembles the array itself |
| `Relinquish_Default` | **you** | `GetPropertyReal` |
| `Units` | **you** | `GetPropertyEnumerated` — see the comment in `main.cpp`: serve it for BOTH the Analog Input and the Analog Output, or the output silently reports `no-units` |

Every object, not just these two, is in [docs/PICS.md](docs/PICS.md).

Going beyond read/write (COV, alarms, scheduling) means implementing a richer
profile — see the series table in [README.md](README.md).

## Reviewing your device

After you have changed anything, review it against the conformance statement
rather than against "it looked fine in the explorer":

1. Regenerate [docs/PICS.md](docs/PICS.md) after editing `docs/objects.json`
   (see [Keeping the PICS honest](#keeping-the-pics-honest) below). A ⚠ row is a
   required property nothing serves.
2. Read **every** property listed for **every** object with a BACnet client, and
   compare the value against the PICS. `"undefined"`, `no-units` and `0` are the
   three shapes a missed callback takes.
3. Diff a new object of a type against the existing one of that type. Anything
   that differs and shouldn't is a callback that matched on instance.
4. For each commandable output: WriteProperty its `Present_Value` at a
   priority, re-read it and `Priority_Array[priority]`, then WriteProperty
   `NULL` at that priority to relinquish and confirm it falls back to
   `Relinquish_Default`. Confirm an out-of-range write (e.g. Binary Output
   value `2`) is rejected with `value-out-of-range`, not silently clamped.
5. Confirm a WriteProperty to a read-only **input** (e.g. Analog Input 1) is
   still rejected.
6. Confirm DM-DAB-B is genuinely absent from what the device claims:
   `Protocol_Services_Supported` should not claim any address-proxy service
   this device does not implement. Do not add code that makes the device
   *look* like it proxies address bindings without the stack actually doing
   so - see [Troubleshooting](#troubleshooting) and `TODO.md`.

### Keeping the PICS honest

`docs/PICS.md` is partly generated. `docs/objects.json` describes each object and
who serves which property; the series tool regenerates the object tables from it
plus the stack's own `docs/property-profile-reference.md` at the pinned commit:

```bash
python tools/gen-objects-properties.py BACnetProfileExample-B-DAP-CPP            # rewrite
python tools/gen-objects-properties.py BACnetProfileExample-B-DAP-CPP --check    # fail if stale
```

(That tool lives in the example-series repository, not in this one. If you only
have this repository, edit the generated block by hand and keep it matching the
callbacks in `main.cpp`.)

When you add an object or a property to `main.cpp`, update `docs/objects.json`
in the same change and regenerate. The `app` list is what the callbacks serve;
`accepted` is for a required property you deliberately leave to the stack's
default, and each one needs a justification. Anything required, not in `app` and
not in `accepted`, comes out as a ⚠ row - that is a defect, not a feature.

## Troubleshooting

| Symptom | Cause / fix |
|---------|-------------|
| On start-up the app prints a wall of red `Error:` lines but the device works | **Expected — this is not your bug.** Two benign sources, both from the stack's own debug logging: (1) the device receives its **own** broadcast I-Am and logs a decode cascade (*"Services is not supported service=[0]"* … *"Failed to process the incoming NPDU"*) — any BACnet/IP device that listens for broadcasts hears itself; (2) a one-time *"UUID has not been set. A UUID must be set for the BACnetSC device to start."* — the stack starts a BACnet/SC datalink these IP-only examples never configure. It appears once and does not spam. On a healthy start-up roughly half the output is these lines. |
| CMake error: *"CAS BACnet Stack adapter not found under: ..."* | Submodules not initialized. Run `git submodule update --init --recursive` (or pass `-D CAS_STACK_DIR=...`). |
| `CASBACnetStackDLL.h: No such file or directory` | Same - submodules not checked out. |
| Windows: *"No CMAKE_CXX_COMPILER could be found"* | Install Visual Studio with the "Desktop development with C++" workload, then re-run from a fresh terminal. |
| First build seems stuck for minutes | Normal - it's compiling ~600 stack files. Only the first build is slow. |
| App prints *"Failed to bind UDP port 47808"* | Another BACnet program is already using 47808. Stop it, or run with `--port <n>`. |
| WriteProperty to an output is rejected | Write to the **output** objects (Analog/Binary/Multi-State Output), not the inputs. Inputs are read-only sensors by design. |
| WriteProperty to an output returns `value-out-of-range` | Working as intended — the `Set*` callbacks validate the written value (a Binary Output only accepts 0/1, a Multi-State Output only accepts `1..Number_Of_States`). Write an in-range value. |
| Client sends Who-Is but sees no I-Am | Firewall is blocking UDP 47808, or the client and device are on different subnets (Who-Is is a broadcast). Allow the port; test on the same subnet first. |
| Replies show an unexpected device instance or vendor | Another BACnet device is already answering on this host/port. On Linux/macOS two processes can share the port and both reply; on Windows the example asks for `SO_EXCLUSIVEADDRUSE` (`common/SimpleUDP.cpp`) so this shows up as a bind failure instead. Stop the other device, or use `--port`. |
| Looking for the device-address-proxy (DM-DAB-B) behavior | **It is not implemented in this example.** The pinned CAS BACnet Stack has no customer-facing export for device-address-proxy configuration - see `TODO.md` and [cas-bacnet-stack#2032](https://github.com/chipkin/cas-bacnet-stack/issues/2032). Do not treat DM-DDB-B/DM-DOB-B (which this example does implement) as a substitute; they are different BIBBs. |
