# Execution model (Executive + Waitlist)

Back to: [Luminary 099 Documentation Wiki](index.md)

Luminary’s runtime model is explicitly built around two cooperating services:

1. The **Executive**, which schedules and dispatches **jobs** (priority-based runnable units with per-job state).
2. The **Waitlist**, which schedules time-based **tasks** and dispatches them from the **T3 interrupt**.

Both services are implemented as first-class modules and are documented in their headers and code.

## Jobs (Executive)

### What is a “job” in Luminary

Within Luminary’s Executive implementation, a job is represented by:

1. A **core set** (a bundle of erasable registers including `MPAC`, `MODE`, `LOC`, `BANKSET`, `PUSHLOC`, and `PRIORITY`) defined in `Luminary099/ERASABLE_ASSIGNMENTS.agc` under “DYNAMICALLY ALLOCATED CORE SETS FOR JOBS”.
2. A **priority word** that indicates active vs dormant state and also packs work-area info (see `Luminary099/EXECUTIVE.agc: CORFOUND` and the masking of `LOW9`/`PRIORITY`).
3. Bank settings, including `BBANK`, `FBANK`, and `SUPERBNK` behavior, which are swapped during job changes (`Luminary099/EXECUTIVE.agc: CHANJOB`).

The Executive code also distinguishes between **basic** jobs and **interpretive** jobs. The `CHANG2` entry uses a negative `LOC` to indicate interpretive jobs (`Luminary099/EXECUTIVE.agc: CHANG2`), and job change dispatch uses `DTCB` for basic jobs and an interpretive resume path for interpretive jobs (`Luminary099/EXECUTIVE.agc: ENDPRCHG`).

### Entering jobs: NOVAC vs FINDVAC

The Executive supports two entry paths:

1. `NOVAC` enters a job request requiring no VAC area (`Luminary099/EXECUTIVE.agc: NOVAC`). It computes a new priority (`NEWPRIO`) and transfers to `NOVAC2` to allocate a core set.
2. `FINDVAC` enters a job request requiring a VAC area, “e.g., all (partially) interpretive jobs” (`Luminary099/EXECUTIVE.agc: FINDVAC`). It stores `NEWPRIO` and the job 2CADR in `NEWLOC`, switches to the Executive bank, and continues at `FINDVAC2`.

VAC areas are reserved using `VAC1USE`..`VAC5USE` (`Luminary099/EXECUTIVE.agc: FINDVAC2` checks them, and `Luminary099/ERASABLE_ASSIGNMENTS.agc` defines the VAC blocks). If no VAC area is available, the Executive bails out with alarm code `1201` (“NO VAC AREAS”) (`Luminary099/EXECUTIVE.agc: FINDVAC2`).

Core sets are allocated by scanning `PRIORITY` registers; the code comments state that an available core set has a priority of “-0” and that dormant jobs have negative priority (`Luminary099/EXECUTIVE.agc: NOVAC3` and surrounding comments). If no core set is available, the Executive bails out with alarm code `1202` (`Luminary099/EXECUTIVE.agc: NEXTCORE`).

### Job switching and dispatch

The Executive’s central “swap and dispatch” routine is `CHANJOB` (`Luminary099/EXECUTIVE.agc: CHANJOB`). It:

1. Inhibits interrupts.
2. Saves and restores superbank state using `WRITE SUPERBNK`.
3. Swaps the `MPAC` blocks and other per-job state.
4. Updates `FIXLOC` based on the masked low bits of the job’s priority word (`Luminary099/EXECUTIVE.agc: CHANJOB`, using `LOW9` and `PRIORITY`).

When no runnable job exists, the Executive enters an idling loop implemented by `DUMMYJOB` and `ADVAN` (`Luminary099/EXECUTIVE.agc: DUMMYJOB`, `ADVAN`).

### Sleeping and waking jobs

The Executive provides:

1. `JOBSLEEP` to voluntarily suspend a job until a later event (`Luminary099/EXECUTIVE.agc: JOBSLEEP` and `JOBSLP1`/`JOBSLP2`).
2. `JOBWAKE` to awaken a job put to sleep by `JOBSLEEP` (`Luminary099/EXECUTIVE.agc: JOBWAKE` and scanning logic in `JOBWAKE2`).

`JOBWAKE2` explicitly describes its algorithm: it scans core sets looking for jobs with negative priority (sleeping) whose `LOC` matches the caller-supplied `NEWLOC`, then re-complements priority to make the job active and reconstructs the `NEWLOC` 2CADR using saved bank info (`Luminary099/EXECUTIVE.agc: JOBWAKE2`).

## Tasks (Waitlist) and time-based scheduling

### What the Waitlist provides

The Waitlist is described in detail in the header of `Luminary099/WAITLIST.agc`. It schedules a “task” to begin after a specified delta time in centiseconds. The Waitlist maintains:

1. `TIME3` which is described as `16384 -(T1-T)` in centiseconds (where `T` is present time and `T1` is the time for task 1) (`Luminary099/WAITLIST.agc`, header).
2. `LST1` which stores relative time deltas between tasks (`LST1`, `LST1+1`, …) (`Luminary099/WAITLIST.agc`, header).
3. `LST2` which stores the 2CADR of each task (`LST2`, `LST2+2`, …) (`Luminary099/WAITLIST.agc`, header).

The primary entry points are:

1. `WAITLIST` for inserting a task with a 2CADR following the `TC WAITLIST` call (`Luminary099/WAITLIST.agc: WAITLIST`).
2. `TWIDDLE`, a variant that saves a word when the task is in the same EBANK/FBANK as the caller (`Luminary099/WAITLIST.agc: TWIDDLE`).
3. `T3RUPT` which dispatches the next due task on the T3 interrupt (`Luminary099/WAITLIST.agc: T3RUPT`).
4. `TASKOVER` which is the expected termination path for tasks and is also used by `FIXDELAY`/`VARDELAY` and `LONGCALL` to return control appropriately (`Luminary099/WAITLIST.agc: TASKOVER`).

### Constraints and failure behavior

The Waitlist header states several “warnings” that define the runtime contract:

1. Delay time must be within a bounded range (`1 <= C(A) <= 16250D` centiseconds).
2. At most nine tasks may be scheduled.
3. “Tasks called under interrupt inhibited.”
4. “Tasks end by TC TASKOVER.”

Overflow conditions are explicit. If the task list is full, `FILLED` triggers a bailout with abort code `01203` (“WAITLIST OVERFLOW - TOO MANY TASKS”) (`Luminary099/WAITLIST.agc: FILLED`).

The Waitlist also issues a “poo-doo” abort `01204` when delay services are called with invalid delta times (`Luminary099/WAITLIST.agc: WAITPOOH` and `LONGPOOH` both call `POODOO1` with `01204`).

### T3RUPT task dispatch

`T3RUPT` is the interrupt-driven dispatcher for Waitlist tasks (`Luminary099/WAITLIST.agc: T3RUPT`). It:

1. Saves `SUPERBNK` state in `BANKRUPT` and saves `QRUPT`.
2. Updates `LST1` and `TIME3` to prepare for the next interval.
3. Shifts the task list (`LST2`) and dispatches the next task using `DTCB` after writing `SUPERBNK` from the task’s `BBCON`.
4. Returns via the resume logic in `TASKOVER` and `RESUME` (`Luminary099/WAITLIST.agc: TASKOVER`, `RESUME`).

Because `T3RUPT` is the delivery mechanism, Waitlist tasks are effectively “interrupt-dispatched callbacks” which then run as normal code and end by returning to `TASKOVER` or to the caller depending on the scheduling primitive used.

## Relationship between jobs, tasks, and interrupts

A useful way to summarize Luminary’s execution model, grounded in code structure, is:

1. Interrupt lead-ins save minimal context and vector to handlers (`Luminary099/INTERRUPT_LEAD_INS.agc`).
2. Some interrupts do substantial work directly (for example, `DODOWNTM` runs every 20 ms per telemetry end pulse as documented in `Luminary099/DOWN_TELEMETRY_PROGRAM.agc`).
3. The Waitlist uses `T3RUPT` to dispatch delayed tasks (`Luminary099/WAITLIST.agc: T3RUPT`).
4. The Executive runs jobs based on priority and can be invoked to enter new jobs (`NOVAC`, `FINDVAC`) or to switch away when a higher-priority job is ready (`Luminary099/EXECUTIVE.agc: PRIOCHNG` and the `NEWJOB` flag checked in `Luminary099/INTERPRETER.agc: DANZIG`).

## See also

1. [Interrupts and I/O](interrupts-io.md) for the interrupt vectoring model and the specific responsibilities of T4/T3/T6/DOWNRUPT.
2. [Interpreter](interpreter.md) for how interpretive jobs resume via `INTRSM` and how the Executive changes interpretive jobs (`Luminary099/EXECUTIVE.agc: ENDPRCHG` and `Luminary099/INTERPRETER.agc: INTRSM`).
