# Memory model

Back to: [Luminary 099 Documentation Wiki](index.md)

Luminary 099 runs on the Apollo Guidance Computer (AGC), which has a banked fixed (rope) memory and a banked erasable (RAM) memory. Within Luminary’s source, the memory model is made concrete through:

1. Symbolic assignments for erasable locations and special registers (`Luminary099/ERASABLE_ASSIGNMENTS.agc`).
2. Interpreter rules for addressing erasable and fixed memory and for switching banks (`Luminary099/INTERPRETER.agc`).
3. Inter-bank calling primitives (`Luminary099/INTER-BANK_COMMUNICATION.agc`).
4. Channel assignments and I/O semantics (`Luminary099/INPUT_OUTPUT_CHANNEL_BIT_DESCRIPTIONS.agc`).

This page focuses on what Luminary explicitly documents in its source modules.

## Erasable vs fixed memory (as used by Luminary)

### Erasable assignments are centralized

`Luminary099/ERASABLE_ASSIGNMENTS.agc` is the authoritative map of erasable symbols to numeric addresses. It begins by describing notation and conventions, including the difference between “sharing” (`EQUALS`) and logical equivalence (`=`), and the mobility/permanence annotations used in comments.

It then defines special registers and interrupt save areas. Examples that show how Luminary structures erasable use include:

1. Special registers (`A`, `L`, `Q`, `EBANK`, `FBANK`, `BBANK`) are assigned to numeric locations (`Luminary099/ERASABLE_ASSIGNMENTS.agc`, “SPECIAL REGISTERS” section).
2. Interrupt storage (`ARUPT`, `LRUPT`, `QRUPT`, `BANKRUPT`, etc.) is defined explicitly (`Luminary099/ERASABLE_ASSIGNMENTS.agc`).
3. Time counters (`TIME1`..`TIME6`) are assigned (`Luminary099/ERASABLE_ASSIGNMENTS.agc`), and those counters are referenced by runtime services and interrupts (for example `TIME3` is central to Waitlist in `Luminary099/WAITLIST.agc`, and `TIME6` is governed by DAP conventions in `Luminary099/T6-RUPT_PROGRAMS.agc`).

### Work areas and “VAC” areas

The Executive distinguishes between jobs that require a “VAC area” and those that do not. This is explicit in `Luminary099/EXECUTIVE.agc`:

1. `NOVAC` is described as “enter a job request requiring no VAC area”.
2. `FINDVAC` is described as “enter a job request requiring a VAC area”, for example interpretive jobs.

The erasable allocation of VAC areas is defined in `Luminary099/ERASABLE_ASSIGNMENTS.agc` under “VAC AREAS”, with `VAC1USE` and `VAC1` blocks through `VAC5USE` and `VAC5`. The Executive’s VAC search logic checks `VAC1USE`..`VAC5USE` in `Luminary099/EXECUTIVE.agc: FINDVAC2`.

### Interpreter work area and pushdown list

The interpreter uses a job’s work area base stored in `FIXLOC` and uses a pushdown list pointer stored in `PUSHLOC` (`Luminary099/ERASABLE_ASSIGNMENTS.agc` defines both). The interpreter explicitly states that E-bank switching occurs when “general erasable (100-3777) is addressed” (`Luminary099/INTERPRETER.agc` header comment in “SECTION 1: DISPATCHER”).

The interpreter also uses a set of MPAC registers (multi-purpose accumulator) in a “core set” that is swapped by the Executive (`Luminary099/EXECUTIVE.agc: CHANJOB` swaps `MPAC` blocks; `Luminary099/ERASABLE_ASSIGNMENTS.agc` defines `MPAC`, `MODE`, `LOC`, `BANKSET`, `PUSHLOC`, `PRIORITY` as the job core set).

## Fixed memory banking and superbanking

Luminary uses bank switching via `FBANK` and the “superbank” channel `SUPERBNK` (channel 7). Several concrete grounding points:

1. `SUPERBNK` is assigned as channel 7 (`Luminary099/ERASABLE_ASSIGNMENTS.agc` “INPUT/OUTPUT CHANNELS”).
2. `INPUT_OUTPUT_CHANNEL_BIT_DESCRIPTIONS.agc` describes channel 7 as “SUPERBNK; OUTPUT CHANNEL; NOT RESET BY RESTART; fixed extension bits used to select the appropriate fixed memory bank if FBANK is 30 octal or more” (channel list in `Luminary099/INPUT_OUTPUT_CHANNEL_BIT_DESCRIPTIONS.agc`).
3. `Luminary099/INTER-BANK_COMMUNICATION.agc` documents the possible superbank settings and provides `SUPERSW` for writing superbank bits to channel 7.

The Executive explicitly writes `SUPERBNK` when changing jobs (`Luminary099/EXECUTIVE.agc: CHANJOB` contains `WRITE SUPERBNK`) and when dispatching jobs via `SUPDXCHZ` (`Luminary099/EXECUTIVE.agc: SUPDXCHZ`).

## Interrupt context storage

Erasable assignments define dedicated interrupt storage such as `ARUPT`, `LRUPT`, `QRUPT`, and `BANKRUPT` (`Luminary099/ERASABLE_ASSIGNMENTS.agc`). The interrupt lead-ins in `Luminary099/INTERRUPT_LEAD_INS.agc` begin by saving `ARUPT` (`DXCH ARUPT`) for each interrupt and then setting `BBANK` and branching to the appropriate handler (for example the `T3RUPT` lead-in sets `BBANK` to `T3RPTBB` and branches to `T3RUPT`).

Several runtime routines also assume that interrupts do not save superbank state, which is why some routines inhibit interrupts around `SUPERBNK` updates. For example `SUPDACAL` explicitly says “INHINT because RUPT does not save SUPERBANK” (`Luminary099/INTER-BANK_COMMUNICATION.agc: SUPDACAL`).

## See also

1. [Execution model](execution-model.md) for how the Executive uses core sets and VAC areas.
2. [Interpreter](interpreter.md) for interpretive addressing rules and pushdown list behavior.
3. [Interrupts and I/O](interrupts-io.md) for interrupt lead-ins and how interrupt handlers save/restore context.
