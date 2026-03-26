# Directory and file index

This page provides a navigational index for the **Luminary099** source modules in this repository, with a focus on “what to read first” and how the modules relate to the runtime architecture.

The authoritative assembly include order is `Luminary099/MAIN.agc`. The canonical module list with page references is in `Luminary099/README.md` (“Source Code Index”, derived from `MAIN.agc`). The original listing’s log-section grouping is documented in `Luminary099/ASSEMBLY_AND_OPERATION_INFORMATION.agc` (“TABLE OF SUBROUTINE LOG SECTIONS”).

## Repository layout relevant to Luminary

The Luminary code and documentation lives under:

1. `Luminary099/` — the LM flight software sources (modules included by `MAIN.agc`).
2. `docs/` — this wiki.

Other top-level directories (e.g., `Comanche055/`) contain the Apollo 11 Command Module software and are out of scope for this Luminary-specific wiki.

## “Read first” kernel modules

These modules define the execution, interrupt, and memory model used by the rest of Luminary:

### Top-level include driver

1. `Luminary099/MAIN.agc`  
   This is the monolithic include list (the effective “linker script”). It defines the build order and is the best map of the whole program.

### Memory and I/O definitions

1. `Luminary099/ERASABLE_ASSIGNMENTS.agc`  
   Central definition of erasable addresses, interrupt storage, counters (`TIME1`..`TIME6`), VAC areas (`VAC1`..`VAC5`), Waitlist lists (`LST1`, `LST2`), Executive core sets (`MPAC` block), and channel naming.

2. `Luminary099/INPUT_OUTPUT_CHANNEL_BIT_DESCRIPTIONS.agc`  
   Canonical description of channels and bit meanings, including `OUT0`, `DSALMOUT`, `CHAN12`..`CHAN14`, `CHAN30`..`CHAN33`, and downlink channels `DNTM1`/`DNTM2`.

### Interrupt entry and major interrupt programs

1. `Luminary099/INTERRUPT_LEAD_INS.agc`  
   Fixed-memory vectors at `SETLOC 4000` that save context (`DXCH ARUPT`) and branch to banked handlers (e.g., `T3RUPT`, `T4RUPT`, `DODOWNTM`, and `DOT6RUPT` via `T6ADR`).

2. `Luminary099/WAITLIST.agc`  
   Contains Waitlist scheduling (`WAITLIST`, `TWIDDLE`) and the T3 interrupt entry (`T3RUPT`) that dispatches delayed tasks.

3. `Luminary099/T4RUPT_PROGRAM.agc`  
   The T4 periodic interrupt program (`T4RUPT`) including DSKY output (`DSPOUT`), Proceed key monitoring (`PROCEEDE`), IMU/ISS monitoring (`IMUMON` / `TNONTEST`), and channel 33 flip-flop monitoring (`C33TEST`).

4. `Luminary099/T6-RUPT_PROGRAMS.agc`  
   Time6 / LM DAP interrupt handling (`DOT6RUPT`) and conventions (`T6JOBCHK`) that define exclusive use of `TIME6`.

5. `Luminary099/DOWN_TELEMETRY_PROGRAM.agc`  
   Downrupt-driven telemetry handler (`DODOWNTM`) that runs every 20 ms and writes to channels 34/35 via `DNTMEXIT`.

6. `Luminary099/DOWNLINK_LISTS.agc`  
   Downlink list format (`1DNADR`..`6DNADR`, `DNPTR`, `DNCHAN`) and concrete downlists such as `LMCSTADL`, `LMORBMDL`, `LMRENDDL`, `LMDSASDL`, `LMLSALDL`, plus the `DNTABLE`.

### Executive, interpreter, and bank-calling primitives

1. `Luminary099/EXECUTIVE.agc`  
   The priority-based job scheduler (`NOVAC`, `FINDVAC`, `CHANJOB`, `JOBSLEEP`, `JOBWAKE`, `DUMMYJOB`).

2. `Luminary099/INTERPRETER.agc`  
   Interpretive instruction engine (`INTPRET`, dispatch loop `DANZIG`/`NEWOPS`, store codes, addressing rules, and many operations).

3. `Luminary099/INTERPRETIVE_CONSTANT.agc`  
   Fixed-location interpretive constants (`DP1/4TH`, unit vectors, special constants).

4. `Luminary099/INTER-BANK_COMMUNICATION.agc`  
   Cross-bank call and jump primitives (`BANKCALL`, `SWCALL`, `POSTJUMP`, `BANKJUMP`), interrupt-safe variants (`IBNKCALL`), and superbank setting (`SUPERSW`).

## Mission and subsystem modules (by index)

The Luminary index in `Luminary099/README.md` enumerates many mission-program and subsystem modules (e.g., rendezvous programs, landing guidance, ascent guidance, DAP/autopilot modules). This wiki does not reproduce the entire index verbatim; instead it recommends using `Luminary099/README.md` for the complete list and using this wiki’s subsystem pages to choose where to start.

For example:

1. Guidance and mission program modules are indexed (e.g., `P20-P25.agc`, `P40-P47.agc`, `THE_LUNAR_LANDING.agc`, `ASCENT_GUIDANCE.agc`) in `Luminary099/README.md` and described at a log-section level in `Luminary099/ASSEMBLY_AND_OPERATION_INFORMATION.agc`.
2. DAP-related modules (e.g., `DAP_INTERFACE_SUBROUTINES.agc`, `DAPIDLER_PROGRAM.agc`, `P-AXIS_RCS_AUTOPILOT.agc`, `Q_R-AXIS_RCS_AUTOPILOT.agc`, `TJET_LAW.agc`) are also listed in `Luminary099/README.md`.

## See also

1. [Architecture](architecture.md) for component-level decomposition.
2. [Execution model](execution-model.md) for how the Executive and Waitlist work together.
3. [Interrupts and I/O](interrupts-io.md) for interrupt vectors and channel usage.
4. `Luminary099/README.md` and `Luminary099/MAIN.agc` for the full module list and assembly order.
