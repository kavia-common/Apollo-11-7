# Glossary

Back to: [Luminary 099 Documentation Wiki](index.md)

This glossary defines AGC- and Luminary-specific terms used throughout this wiki. Where the source explicitly documents a concept, it is grounded with a file reference.

## Core concepts

### AGC

Apollo Guidance Computer, the onboard guidance computer for Apollo spacecraft. Luminary 099 is the LM AGC flight software archive in `Luminary099/`.

### Bank, bank switching

A method of selecting which region of fixed memory is currently addressable. Luminary uses `FBANK` and `BBANK` erasable registers for bank selection (`Luminary099/ERASABLE_ASSIGNMENTS.agc` defines `FBANK`, `BBANK`) and uses channel 7 (`SUPERBNK`) for superbank selection (`Luminary099/INPUT_OUTPUT_CHANNEL_BIT_DESCRIPTIONS.agc` channel 7; `Luminary099/INTER-BANK_COMMUNICATION.agc` documents superbank settings and `SUPERSW`).

### Superbank

An extension mechanism to select high fixed banks when `FBANK` is 30 octal or more (`Luminary099/INPUT_OUTPUT_CHANNEL_BIT_DESCRIPTIONS.agc` channel 7). Luminary provides `SUPERSW` to write superbank bits (`Luminary099/INTER-BANK_COMMUNICATION.agc: SUPERSW`) and the Executive writes `SUPERBNK` during job changes (`Luminary099/EXECUTIVE.agc: CHANJOB`).

### Erasable

AGC RAM. Luminary’s erasable map is defined in `Luminary099/ERASABLE_ASSIGNMENTS.agc`, including special registers, interrupt storage, time counters, and major subsystem state.

### Fixed memory

AGC rope memory (read-only). Luminary modules are stored in fixed memory banks and assembled monolithically via `Luminary099/MAIN.agc`.

### Channel

An AGC I/O channel. Luminary assigns symbolic names to channels (e.g., `OUT0`, `DSALMOUT`, `CHAN30`) in `Luminary099/ERASABLE_ASSIGNMENTS.agc` and documents bit meanings in `Luminary099/INPUT_OUTPUT_CHANNEL_BIT_DESCRIPTIONS.agc`.

## Runtime services

### Executive

The priority-based job scheduler and dispatcher implemented in `Luminary099/EXECUTIVE.agc` (see labels such as `NOVAC`, `FINDVAC`, `CHANJOB`, `JOBSLEEP`, and `JOBWAKE`).

### Job

A runnable unit managed by the Executive. Jobs have per-job “core sets” (including `MPAC`, `LOC`, `BANKSET`, `PUSHLOC`, `PRIORITY`) defined in `Luminary099/ERASABLE_ASSIGNMENTS.agc` under “DYNAMICALLY ALLOCATED CORE SETS FOR JOBS”. The Executive swaps core sets in `Luminary099/EXECUTIVE.agc: CHANJOB`.

### VAC area

A work area allocated for certain jobs, especially interpretive jobs. VAC areas and their “use” flags (`VAC1USE`..`VAC5USE`) are defined in `Luminary099/ERASABLE_ASSIGNMENTS.agc`. The Executive searches for VAC availability in `Luminary099/EXECUTIVE.agc: FINDVAC2`.

### Waitlist

The delayed task scheduler implemented in `Luminary099/WAITLIST.agc`. It schedules tasks in centiseconds and dispatches them from `T3RUPT`.

### Task

A time-based callback scheduled by the Waitlist. The Waitlist module header specifies that tasks end by `TC TASKOVER` and are dispatched with interrupts inhibited (`Luminary099/WAITLIST.agc` header; `TASKOVER` label).

### Interpreter

The interpretive instruction engine implemented in `Luminary099/INTERPRETER.agc`, entered via `INTPRET` and driven by the main loop `DANZIG`/`NEWOPS`. Interpretive programs use work areas (`FIXLOC`) and a pushdown list (`PUSHLOC`).

### MPAC

Multi-Purpose Accumulator storage used heavily by the interpreter and other routines. `MPAC` is part of the Executive core set and is defined in `Luminary099/ERASABLE_ASSIGNMENTS.agc`.

### LOC / BANKSET

Interpreter and Executive control registers. `LOC` is the current interpretive instruction pointer-like location; `BANKSET` captures bank context. Both are in the core set defined in `Luminary099/ERASABLE_ASSIGNMENTS.agc` and used in `Luminary099/INTERPRETER.agc`.

## Interrupts

### Lead-in

Fixed-memory entry code that saves minimal context and branches to a banked interrupt handler. Lead-ins are defined in `Luminary099/INTERRUPT_LEAD_INS.agc`.

### T3RUPT

The interrupt that dispatches Waitlist tasks, implemented at `Luminary099/WAITLIST.agc: T3RUPT`.

### T4RUPT

A periodic interrupt handler used for housekeeping, DSKY output, and device monitoring. Implemented in `Luminary099/T4RUPT_PROGRAM.agc: T4RUPT`.

### T6RUPT / TIME6

The Time6 interrupt and counter, reserved for LM DAP jet timing per conventions in `Luminary099/T6-RUPT_PROGRAMS.agc`. The T6 handler is `DOT6RUPT`.

### DOWNRUPT

The downlink telemetry end-pulse interrupt. The handler is `Luminary099/DOWN_TELEMETRY_PROGRAM.agc: DODOWNTM`, which runs every 20 ms per its documentation.

## See also

1. [Memory model](memory-model.md) for a grounded overview of erasable and bank switching.
2. [Execution model](execution-model.md) for job/task interactions.
3. [Interrupts and I/O](interrupts-io.md) for interrupt vectors and channel usage.
