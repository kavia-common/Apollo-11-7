# Architecture

Back to: [Luminary 099 Documentation Wiki](index.md)

Luminary 099 is organized as a set of assembly modules which are “linked” by inclusion into a single assembly via `Luminary099/MAIN.agc`. The architecture is therefore best understood as a set of cooperating runtime services plus mission and device subsystems, rather than as separately-linked binaries.

This page describes the major architectural components that are explicit in the source and points you to the modules and labels that implement them.

## Top-level decomposition

At a high level, Luminary 099 consists of:

1. A **runtime kernel** that schedules and dispatches work (Executive + Waitlist), supports an interpretive instruction engine (Interpreter), and provides bank-switch and call primitives.
2. A set of **interrupt-driven services** that react to time and hardware events, including periodic housekeeping and I/O monitoring.
3. Mission- and vehicle-specific **subsystems** for guidance, navigation, control, rendezvous and landing logic, DSKY interaction, and telemetry.

These components are visible as dedicated modules in `Luminary099/MAIN.agc` and are described by the “TABLE OF SUBROUTINE LOG SECTIONS” in `Luminary099/ASSEMBLY_AND_OPERATION_INFORMATION.agc`.

## Runtime kernel services

### Executive (job scheduling and dispatch)

The Executive is the job scheduler and dispatcher. It handles:

1. Entering a job request without allocating a VAC work area (`Luminary099/EXECUTIVE.agc: NOVAC`).
2. Entering a job request that requires a VAC work area (typically interpretive jobs) (`Luminary099/EXECUTIVE.agc: FINDVAC` and the VAC search logic in `FINDVAC2`).
3. Suspending and swapping between jobs (basic and interpretive) (`Luminary099/EXECUTIVE.agc: CHANG1`, `CHANG2`, and the core-swap routine `CHANJOB`).
4. Putting a job to sleep and waking it based on a matching address (`Luminary099/EXECUTIVE.agc: JOBSLEEP`, `JOBWAKE`, and `JOBWAKE2`).
5. Idling when no job is runnable (`Luminary099/EXECUTIVE.agc: DUMMYJOB` and `ADVAN`).

The Executive is also responsible for coordinating bank settings when dispatching work. For example, it uses `SUPERBNK` writes in `CHANJOB` and in the fixed-fixed `SUPDXCHZ` helper (`Luminary099/EXECUTIVE.agc: SUPDXCHZ`).

### Waitlist (delayed tasks and T3 dispatch)

The Waitlist provides delayed scheduling of “tasks” based on centisecond time. It maintains two lists (`LST1` and `LST2`) and an associated time register (`TIME3`) as described in the module header in `Luminary099/WAITLIST.agc`. Tasks are dispatched from the **T3 interrupt** entry point `Luminary099/WAITLIST.agc: T3RUPT`, and each dispatched task is expected to end with `TC TASKOVER` (see “Warnings” in `Luminary099/WAITLIST.agc`).

The Waitlist also includes `LONGCALL`, which supports longer delay times by cycling through multiple Waitlist entries (`Luminary099/WAITLIST.agc: LONGCALL`, `LONGCYCL`, and `LASTTIME`).

### Interpreter (interpretive instruction engine)

The Interpreter executes “interpretive programs” stored in fixed memory. The entry point is `Luminary099/INTERPRETER.agc: INTPRET`, which sets up `LOC`, `BANKSET`, and `INTBIT15` and then enters the main dispatch loop (`DANZIG`, `NEWOPS`). The interpreter defines:

1. Operand addressing rules (direct, indexed, and pushdown list) (`Luminary099/INTERPRETER.agc: ADDRESS`, `DIRADRES`, `INDEX`, and `PUSHUP`).
2. Control transfer and calls (e.g., `CALL`, `GOTO`, `CGOTO`) and return-to-basic (`RTB`) (`Luminary099/INTERPRETER.agc: CALL`, `GOTO`, `RTB/BHIZ`).
3. A large set of interpretive operations (load/store, arithmetic, vector ops, trig, shifts) dispatched via jump tables (`INDJUMP`, `MISCJUMP`, `UNAJUMP`).

Interpretive constants used by interpretive routines are defined in `Luminary099/INTERPRETIVE_CONSTANT.agc` (for example `DP1/4TH`, `UNITX`, and the “other half-memory” constants in `INTPRET2`).

### Inter-bank communication and calling conventions

Because Luminary is banked, it includes explicit routines for calling or jumping across banks:

1. `BANKCALL` and `SWCALL` for calling a subroutine in another bank (`Luminary099/INTER-BANK_COMMUNICATION.agc: BANKCALL`, `SWCALL`, and return via `SWRETURN`).
2. `POSTJUMP` and `BANKJUMP` for unilateral jumps with preserved registers (`Luminary099/INTER-BANK_COMMUNICATION.agc: POSTJUMP`, `BANKJUMP`).
3. `IBNKCALL` and `ISWCALLL` variants for use in interrupt context (`Luminary099/INTER-BANK_COMMUNICATION.agc: IBNKCALL`).
4. `SUPERSW` for setting the superbank bits written to channel 7 (`Luminary099/INTER-BANK_COMMUNICATION.agc: SUPERSW` and the superbank table in the same file).

## Interrupt-driven services

Interrupt entry lead-ins are defined in fixed memory by `Luminary099/INTERRUPT_LEAD_INS.agc`. The lead-ins save context (for example storing `ARUPT`) and then branch to banked handlers such as:

1. `T3RUPT` for Waitlist task dispatch (`Luminary099/INTERRUPT_LEAD_INS.agc` lead-in that sets `BBANK` and `TCF T3RUPT`, and `Luminary099/WAITLIST.agc: T3RUPT`).
2. `T4RUPT` for periodic housekeeping and I/O monitoring (`Luminary099/INTERRUPT_LEAD_INS.agc` lead-in for `T4RUPT`, and `Luminary099/T4RUPT_PROGRAM.agc: T4RUPT`).
3. `DOT6RUPT` for the Time6 interrupt used by the LM DAP conventions (`Luminary099/INTERRUPT_LEAD_INS.agc: T6ADR` pointing to `DOT6RUPT`, implemented in `Luminary099/T6-RUPT_PROGRAMS.agc: DOT6RUPT`).
4. `DODOWNTM` for downlink telemetry end-pulse interrupt (“DOWNRUPT”) (`Luminary099/INTERRUPT_LEAD_INS.agc` lead-in that `TCF DODOWNTM`, and `Luminary099/DOWN_TELEMETRY_PROGRAM.agc: DODOWNTM`).

The T4RUPT program includes explicit monitoring and processing of several hardware status channels, including ISS/IMU bits on channel 30 (`IMUMON`) and flip-flop bits on channel 33 (`C33TEST`) (`Luminary099/T4RUPT_PROGRAM.agc: IMUMON`, `C33TEST`).

## Mission and device subsystems

Luminary contains many mission programs and subsystem modules. This wiki provides separate deep dives, but at the architectural level it is useful to note:

1. Guidance, navigation, control, landing, and rendezvous logic are organized largely by program modules (e.g., `P20-P25.agc`, `P40-P47.agc`, `THE_LUNAR_LANDING.agc`, `ASCENT_GUIDANCE.agc`) as indexed in `Luminary099/README.md`.
2. Control/autopilot logic interacts with interrupts and dedicated counters. For example, Time6 is reserved for LM DAP jet timing, and non-DAP code is explicitly warned not to tamper with it (`Luminary099/T6-RUPT_PROGRAMS.agc` header conventions; see also `T6JOBCHK`).

## See also

For the dynamic aspects of architecture, read:

1. [Execution model](execution-model.md) for the job/task model implemented by Executive and Waitlist.
2. [Interrupts and I/O](interrupts-io.md) for the interrupt vectors, periodic loops, and I/O channel semantics.
3. [Diagrams](diagrams.md) for a visual representation of these components.
