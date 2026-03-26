# Telemetry and downlink

Back to: [Luminary 099 Documentation Wiki](index.md)

Luminary 099 contains a downlink telemetry subsystem that is explicitly interrupt-driven and list-based. Its structure is visible in two modules:

1. `Luminary099/DOWN_TELEMETRY_PROGRAM.agc`, which implements the 20 ms handler that selects and transmits data.
2. `Luminary099/DOWNLINK_LISTS.agc`, which defines the downlink list format and specific lists used for different mission phases.

## Downrupt-driven execution (20 ms)

The downlink telemetry program states that it is “initiated by telemetry end pulse from the downlink telemetry converter” and that the pulse occurs “50 times per sec (every 20 ms)”, so `DODOWNTM` is executed at that rate (`Luminary099/DOWN_TELEMETRY_PROGRAM.agc` header).

The interrupt lead-in for DOWNRUPT is in `Luminary099/INTERRUPT_LEAD_INS.agc`, which saves `ARUPT`, sets `BBANK`, and transfers to `DODOWNTM` (“DOWNRUPT” lead-in).

The handler writes two AGC words per interrupt to channels 34 and 35:

- These channels are described as `DNTM1` and `DNTM2` in `Luminary099/ERASABLE_ASSIGNMENTS.agc` (channel assignments).
- The channel bit descriptions file describes channel 34 and 35 as downlink serialization words (`Luminary099/INPUT_OUTPUT_CHANNEL_BIT_DESCRIPTIONS.agc`).
- The program’s `DNTMEXIT` writes `A` to `DNTM1` and `L` to `DNTM2` (`Luminary099/DOWN_TELEMETRY_PROGRAM.agc: DNTMEXIT`).

## Downlink list format (DNADR / DNPTR / DNCHAN)

The downlink list system is described in both the list file and in the down telemetry program comments.

`Luminary099/DOWNLINK_LISTS.agc` defines “special downlink op codes” such as:

1. `1DNADR`, `2DNADR`, … `6DNADR` which indicate sending 2, 4, … 12 AGC words from erasable starting at an ECADR-like address.
2. `DNCHAN` which indicates downlinking channel pairs.
3. `DNPTR` which points to a sublist.

The program in `DOWN_TELEMETRY_PROGRAM.agc` explains several constraints and rules, including:

1. Lists consist of a control list and sublists.
2. Snapshot sublists are homogeneous and must be stored in a buffer during one downrupt.
3. Snapshot sublists can contain only `1DNADR` entries and cannot refer to the first location in any EBANK (these are repeated as “rules unique to the snapshot portion”) (`Luminary099/DOWN_TELEMETRY_PROGRAM.agc`, snapshot notes).

## Where downlink list selection state is stored

The down telemetry program uses several erasable control words and notes restart behavior:

1. `DNTMGOTO` is used as a “goto appropriate phase of program” pointer (`Luminary099/DOWN_TELEMETRY_PROGRAM.agc: DODOWNTM` and usage of `TC DNTMGOTO`).
2. The program states that `STARTSUB` (fresh start/restart logic elsewhere) initializes the downlist pointer so that after a restart downlink resumes at the beginning of the current downlist (`Luminary099/DOWN_TELEMETRY_PROGRAM.agc` header, “RESTART PROTECTION” section). The specific “startsub” routine is not in this file, so this wiki does not claim its exact label, but it does ground the restart behavior statement to the down telemetry program’s own documentation.
3. `DNLSTCOD` and list tables are used to select a list; `DOWNLINK_LISTS.agc` defines `DNTABLE` and list roots such as `LMCSTADL`, `LMORBMDL`, `LMRENDDL`, `LMDSASDL`, and `LMLSALDL` (`Luminary099/DOWNLINK_LISTS.agc`).

Erasable assignments define downlink-related storage in the “DOWNLINK STORAGE” section, including `DNLSTCOD`, `LDATALST`, `DNTMGOTO`, `TMINDEX`, `DUMPLOC`, `DNQ`, and `DNTMBUFF` (`Luminary099/ERASABLE_ASSIGNMENTS.agc`, “DOWNLINK STORAGE”).

## Examples: lists by mission context

The list file defines concrete downlists for different activity types. The following are examples grounded in `Luminary099/DOWNLINK_LISTS.agc`:

1. `LMCSTADL` is labeled “LM COAST AND ALIGNMENT DOWNLIST”.
2. `LMORBMDL` is labeled “LM ORBITAL MANEUVERS LIST”.
3. `LMRENDDL` is labeled “LM RENDEZVOUS AND PRE-THRUST DOWNLIST”.
4. `LMDSASDL` is labeled “LM DESCENT AND ASCENT DOWNLIST”.
5. `LMLSALDL` is labeled “LM LUNAR SURFACE ALIGN DOWNLIST”.

Each list uses control-list entries such as `DNPTR` (sublist pointers) and `*DNADR` address descriptors to assemble a 2-second “downlist” (the down telemetry program states it is coded for a 2-second downlist).

## See also

1. [Interrupts and I/O](interrupts-io.md) for the DOWNRUPT lead-in and the relationship between downrupt and channel 34/35 writes.
2. [Memory model](memory-model.md) for how erasable locations like `DNTMBUFF` and pointers like `DNTMGOTO` are assigned and used.
