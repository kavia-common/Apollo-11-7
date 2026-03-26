# Overview (Luminary 099)

Back to: [Luminary 099 Documentation Wiki](index.md)

Luminary 099 (“LMY99”) is the Apollo 11 Lunar Module (LM) flight software for the Apollo Guidance Computer (AGC). In this repository, the original monolithic source listing has been split into smaller modules and then reassembled through a single include file, `Luminary099/MAIN.agc`, to preserve the historical “assemble as one deck” workflow. This approach is described explicitly in `Luminary099/README.md` and the “Table of subroutine log sections” in `Luminary099/ASSEMBLY_AND_OPERATION_INFORMATION.agc`.

## What is in the Luminary099 directory

`Luminary099/MAIN.agc` is the top-level include list. It enumerates the full build in the order the assembler sees it, with page references back to the scanned listing. The directory contains both “system services” (e.g., Executive, Waitlist, Interpreter, interrupt programs) and mission/program logic (e.g., landing guidance, ascent guidance, rendezvous programs, DAP/autopilot modules).

You can treat each `.agc` file as a “module” in the absence of a linker. The boundaries largely correspond to the original listing’s log sections. See `Luminary099/ASSEMBLY_AND_OPERATION_INFORMATION.agc` for the canonical “TABLE OF SUBROUTINE LOG SECTIONS”.

## How to read the source

The code is AGC assembly formatted for the modern `yaYUL` assembler (see `Luminary099/README.md`). A few practical navigation tips grounded in the Luminary sources:

The AGC runtime is built around a small set of services that everything else depends on.

1. The **Executive** schedules and dispatches “jobs”, supports sleeping and waking jobs, and manages per-job work areas. This is implemented in `Luminary099/EXECUTIVE.agc` (for example `NOVAC`, `FINDVAC`, `CHANJOB`, `JOBSLEEP`, and `JOBWAKE`).
2. The **Waitlist** schedules delayed “tasks” (time-based callbacks) and dispatches them from the **T3** interrupt. This is implemented in `Luminary099/WAITLIST.agc` (for example `WAITLIST`, `TWIDDLE`, and the interrupt entry `T3RUPT`).
3. The **Interpreter** is an interpretive instruction engine that executes compact “interpretive programs” stored in fixed memory while using erasable “work areas” for scratch and a pushdown list. Entry is via `Luminary099/INTERPRETER.agc: INTPRET`, and interpretive constants live in `Luminary099/INTERPRETIVE_CONSTANT.agc`.
4. **Interrupt lead-ins** define the fixed-memory entry points that save context and vector into banked interrupt handlers. These lead-ins are in `Luminary099/INTERRUPT_LEAD_INS.agc`. Several major interrupt programs are in `Luminary099/T4RUPT_PROGRAM.agc`, `Luminary099/WAITLIST.agc` (T3), and `Luminary099/T6-RUPT_PROGRAMS.agc` (Time6 / LM DAP convention).
5. Memory naming and “where variables live” is defined centrally by `Luminary099/ERASABLE_ASSIGNMENTS.agc`. It defines fixed register numbers (e.g., `A`, `L`, `Q`, `EBANK`, `FBANK`) and assigns symbolic names for scratch and shared state (for example `TIME3`, `TIME4`, `TIME6`, `ARUPT`, `QRUPT`, `NEWJOB`, `MPAC`, `LST1`, `LST2`, `IMODES30`, `IMODES33`).

## A minimal “module map” of the runtime

The include order in `Luminary099/MAIN.agc` provides a reliable map of the overall build and a natural reading order. Several modules are particularly central:

| Area | Why it matters | Primary source(s) |
|---|---|---|
| Interrupt vectors / lead-ins | Defines fixed entry points and bank setup for each interrupt | `Luminary099/INTERRUPT_LEAD_INS.agc` |
| Executive | Job scheduling, priority management, core set + VAC area allocation, job sleep/wake | `Luminary099/EXECUTIVE.agc` (`NOVAC`, `FINDVAC2`, `CHANJOB`, `JOBSLEEP`, `JOBWAKE`) |
| Waitlist | Delayed tasks, T3 interrupt dispatch, LONGCALL | `Luminary099/WAITLIST.agc` (`WAITLIST`, `T3RUPT`, `TASKOVER`, `LONGCALL`) |
| Interpreter | Interpretive program execution and support routines | `Luminary099/INTERPRETER.agc` (`INTPRET`, `DANZIG`, `NEWOPS`) |
| Inter-bank calling | Banked subroutine call primitives for normal and interrupt context | `Luminary099/INTER-BANK_COMMUNICATION.agc` (`BANKCALL`, `SWCALL`, `IBNKCALL`, `SUPERSW`) |
| I/O channel meanings | Channel numbers and bit-level semantics | `Luminary099/INPUT_OUTPUT_CHANNEL_BIT_DESCRIPTIONS.agc` |
| T4RUPT | Periodic activities (including DSKY output and monitoring of important status bits) | `Luminary099/T4RUPT_PROGRAM.agc` (`T4RUPT`, `DSPOUT`, `IMUMON`, `C33TEST`, `GLOCKMON`) |
| Downlink lists + program | Downlink list format and 20 ms downrupt processing | `Luminary099/DOWNLINK_LISTS.agc`, `Luminary099/DOWN_TELEMETRY_PROGRAM.agc` (`DODOWNTM`) |

## See also

If you want a “quick visual” for how the Executive, Waitlist, interrupts, interpreter, and telemetry relate, start with [Diagrams](diagrams.md) and then read [Execution model](execution-model.md) and [Interrupts and I/O](interrupts-io.md).
