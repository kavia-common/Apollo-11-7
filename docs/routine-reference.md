# Key routines reference

This page is an index of particularly important routines and labels in Luminary 099 that define the runtime model, bank calling, interrupts, and telemetry. It is meant as a “jump table” for readers who want to go straight to the most influential code.

All routines referenced here are grounded in the repository source by **file + label**.

## Executive (jobs)

### Job entry and allocation

1. `Luminary099/EXECUTIVE.agc: NOVAC`  
   Enters a job request requiring no VAC area.

2. `Luminary099/EXECUTIVE.agc: FINDVAC` and `Luminary099/EXECUTIVE.agc: FINDVAC2`  
   Enters a job request requiring a VAC area and locates an available VAC (`VAC1USE`..`VAC5USE`), bailing out with `1201` if none.

3. `Luminary099/EXECUTIVE.agc: NOVAC2` / `NOVAC3` / `NEXTCORE`  
   Core-set allocation logic; bails out with `1202` if no core sets are available.

### Job switching, sleep, and wake

1. `Luminary099/EXECUTIVE.agc: CHANG1`  
   Suspend a basic job so a higher priority job may be serviced.

2. `Luminary099/EXECUTIVE.agc: CHANG2`  
   Suspend an interpretive job (negative `LOC` indicates interpretive).

3. `Luminary099/EXECUTIVE.agc: CHANJOB`  
   Swaps core set 0 with another core set (`NEWJOB`) and dispatches.

4. `Luminary099/EXECUTIVE.agc: JOBSLEEP` / `JOBSLP1`  
   Voluntarily suspend a job until an anticipated event (I/O event, etc.).

5. `Luminary099/EXECUTIVE.agc: JOBWAKE` / `JOBWAKE2`  
   Wake a sleeping job by matching its `LOC` to the caller’s `NEWLOC`.

### Idle path

1. `Luminary099/EXECUTIVE.agc: DUMMYJOB` and `Luminary099/EXECUTIVE.agc: ADVAN`  
   Idling and activity light maintenance when no job is runnable.

## Waitlist (tasks)

1. `Luminary099/WAITLIST.agc: WAITLIST`  
   Inserts a delayed task specified by a following 2CADR, using delta time in centiseconds.

2. `Luminary099/WAITLIST.agc: TWIDDLE`  
   A WAITLIST variant that saves a word when task bank matches caller bank.

3. `Luminary099/WAITLIST.agc: T3RUPT`  
   The T3 interrupt handler that dispatches due waitlisted tasks.

4. `Luminary099/WAITLIST.agc: TASKOVER` and `Luminary099/WAITLIST.agc: RESUME`  
   Task termination and resume sequence used by tasks and delay utilities.

5. `Luminary099/WAITLIST.agc: LONGCALL` / `LNGCALL2` / `LONGCYCL` / `LASTTIME`  
   Long delay support by cycling through multiple shorter delays before calling WAITLIST.

## Interpreter (interpretive engine)

1. `Luminary099/INTERPRETER.agc: INTPRET`  
   Entry to interpretive execution; sets up `LOC`, `BANKSET`, and `INTBIT15`.

2. `Luminary099/INTERPRETER.agc: INTRSM`  
   Resume a suspended interpretive job.

3. `Luminary099/INTERPRETER.agc: DANZIG` and `Luminary099/INTERPRETER.agc: NEWOPS`  
   Main dispatch loop; fetches opcode pairs and checks for job switches (`NEWJOB`).

4. `Luminary099/INTERPRETER.agc: INDJUMP` / `MISCJUMP` / `UNAJUMP`  
   Jump tables for interpretive operations.

## Inter-bank calls and superbank control

1. `Luminary099/INTER-BANK_COMMUNICATION.agc: BANKCALL`  
   Call a subroutine in another bank; CADR follows `TC BANKCALL`.

2. `Luminary099/INTER-BANK_COMMUNICATION.agc: SWCALL` and `SWRETURN`  
   SWCALL is like BANKCALL but CADR arrives in `A`. SWRETURN returns to caller.

3. `Luminary099/INTER-BANK_COMMUNICATION.agc: POSTJUMP` and `BANKJUMP`  
   Unilateral jump variants with preserved registers.

4. `Luminary099/INTER-BANK_COMMUNICATION.agc: IBNKCALL`  
   Interrupt-safe bank call variant (uses `RUPTREG3`/`RUPTREG4` as return storage).

5. `Luminary099/INTER-BANK_COMMUNICATION.agc: SUPERSW`  
   Writes superbank bits to channel 7 (`SUPERBNK`) and returns.

## Interrupt lead-ins and major interrupt handlers

### Lead-ins (fixed)

1. `Luminary099/INTERRUPT_LEAD_INS.agc` (fixed lead-ins at `SETLOC 4000`)  
   Contains the fixed entry points that save `ARUPT` and branch to handlers.

### T4RUPT

1. `Luminary099/T4RUPT_PROGRAM.agc: T4RUPT`  
   Main periodic interrupt handler.

2. `Luminary099/T4RUPT_PROGRAM.agc: DSPOUT` / `DSPOUTSB` / `DSPLAY`  
   Display output pipeline that writes `OUT0`.

3. `Luminary099/T4RUPT_PROGRAM.agc: PROCEEDE`  
   Proceed key monitoring via `CHAN32` bit 14; may request `PROCKEY` via `NOVAC`.

4. `Luminary099/T4RUPT_PROGRAM.agc: IMUMON` / `TNONTEST` / `C33TEST` / `GLOCKMON`  
   IMU/ISS monitoring and initialization logic, channel 33 flip-flop monitoring, and gimbal lock monitoring.

### T6RUPT (LM DAP timing)

1. `Luminary099/T6-RUPT_PROGRAMS.agc: T6JOBCHK`  
   Checks `TIME6` for waiting T6RUPT and processes it.

2. `Luminary099/T6-RUPT_PROGRAMS.agc: DOT6RUPT`  
   T6 interrupt handler; calls `T6JOBCHK` and returns to `RESUME`.

### Downrupt (telemetry)

1. `Luminary099/DOWN_TELEMETRY_PROGRAM.agc: DODOWNTM`  
   Down telemetry program entered on telemetry end pulse interrupt (20 ms rate).

2. `Luminary099/DOWN_TELEMETRY_PROGRAM.agc: DNTMEXIT`  
   Writes the selected pair of words to channels 34 and 35 (`DNTM1`, `DNTM2`).

3. `Luminary099/DOWNLINK_LISTS.agc: DNTABLE`  
   Table of downlink list roots.

## See also

1. [Directory and file index](directory-file-index.md) for where these routines live in the module map.
2. [Execution model](execution-model.md) for deeper explanation of jobs and tasks.
3. [Interrupts and I/O](interrupts-io.md) for how these handlers are vectored and which channels they use.
