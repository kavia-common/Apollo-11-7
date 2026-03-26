# Program flow maps

Back to: [Luminary 099 Documentation Wiki](index.md)

This page provides source-grounded flow maps for several critical runtime paths in Luminary 099. The goal is to help you trace “what happens when” at the system level, even before you understand every mission program.

Because Luminary is a monolithic include build (`Luminary099/MAIN.agc`) and because many phase-specific program entry points live in mission modules not fully expanded in this wiki, these flow maps focus on flows that are clearly visible in the kernel and interrupt modules.

## Flow 1: Interrupt lead-in → handler → resume

The interrupt lead-ins in `Luminary099/INTERRUPT_LEAD_INS.agc` are the fixed entry points. Each lead-in saves context (e.g., `DXCH ARUPT`) and vectors into a handler in the correct bank.

A representative grounded flow is the T4 path:

1. Fixed lead-in: `Luminary099/INTERRUPT_LEAD_INS.agc` “T4RUPT” lead-in sets `BBANK` (`T4RPTBB`) and branches to `T4RUPT`.
2. Handler: `Luminary099/T4RUPT_PROGRAM.agc: T4RUPT` saves `BANKRUPT`, saves `QRUPT`, and then runs periodic logic.
3. Return: Many T4 paths return through `RESUME` (the resume primitive is shown explicitly in `Luminary099/T6-RUPT_PROGRAMS.agc: DOT6RUPT` returning to `RESUME`, and `WAITLIST.agc` includes a resume sequence at `RESUME` used by `T3RUPT`).

## Flow 2: Waitlist scheduling → T3 dispatch → task termination

Waitlist behavior is extensively documented in the header of `Luminary099/WAITLIST.agc` and implemented by `WAITLIST`, `T3RUPT`, and `TASKOVER`.

A grounded flow is:

1. A program calls `WAITLIST` with delta time in centiseconds and a following 2CADR (`Luminary099/WAITLIST.agc` calling sequence; `Luminary099/WAITLIST.agc: WAITLIST`).
2. The Waitlist inserts the task into `LST1`/`LST2` and returns to the caller (via `LVWTLIST`).
3. On T3RUPT, the handler shifts `LST1`/`LST2`, writes `SUPERBNK` from the task’s `BBCON`, and dispatches the task via `DTCB` (`Luminary099/WAITLIST.agc: T3RUPT`).
4. The task ends via `TC TASKOVER` per Waitlist warning #4 (`Luminary099/WAITLIST.agc` header; `TASKOVER` label).
5. `TASKOVER` either dispatches additional due tasks or restores context and resumes the interrupted code (`Luminary099/WAITLIST.agc: TASKOVER`, `RESUME`).

## Flow 3: Executive job entry and possible job switching

The Executive’s `NOVAC` and `FINDVAC` are entry points for inserting new jobs (`Luminary099/EXECUTIVE.agc: NOVAC`, `FINDVAC`). A job switch is performed by `CHANJOB` (`Luminary099/EXECUTIVE.agc: CHANJOB`).

A particularly visible coupling is in the Interpreter: the `DANZIG` loop checks `NEWJOB` and, if set, transfers to `CHANG2` to switch interpretive jobs (`Luminary099/INTERPRETER.agc: DANZIG`, `CHANG2` referenced by label in `EXECUTIVE.agc`).

## Flow 4: Telemetry end pulse (downrupt) → list processing → channel write

The down telemetry program explicitly documents that `DODOWNTM` runs every 20 ms and loads the selected data words into output channels 34 and 35 (`Luminary099/DOWN_TELEMETRY_PROGRAM.agc` header; `DODOWNTM` and `DNTMEXIT`).

A grounded flow is:

1. Interrupt lead-in: `Luminary099/INTERRUPT_LEAD_INS.agc` “DOWNRUPT” lead-in branches to `DODOWNTM`.
2. Handler: `Luminary099/DOWN_TELEMETRY_PROGRAM.agc: DODOWNTM` selects data using list pointers and list formats (`1DNADR`..`6DNADR`, `DNPTR`, `DNCHAN`) defined in `Luminary099/DOWNLINK_LISTS.agc`.
3. Output: `Luminary099/DOWN_TELEMETRY_PROGRAM.agc: DNTMEXIT` writes to channels `DNTM1` and `DNTM2` (channels 34 and 35) using `WRITE`.
4. Return: the handler exits via `TCF RESUME` (`TMRESUME` path).

## Flow 5: “Major mode / mission phase” (source-grounded identification)

The downlink lists provide a source-grounded view of several mission/phase categories because each downlist is named for a context:

1. Coast and alignment: `Luminary099/DOWNLINK_LISTS.agc: LMCSTADL`
2. Orbital maneuvers: `Luminary099/DOWNLINK_LISTS.agc: LMORBMDL`
3. Rendezvous and pre-thrust: `Luminary099/DOWNLINK_LISTS.agc: LMRENDDL`
4. Descent and ascent: `Luminary099/DOWNLINK_LISTS.agc: LMDSASDL`
5. Lunar surface align: `Luminary099/DOWNLINK_LISTS.agc: LMLSALDL`
6. AGS initialization/update: `Luminary099/DOWNLINK_LISTS.agc: LMAGSIDL`

These names are not, by themselves, a complete mission sequence, but they are concrete anchors for identifying major operational contexts that the flight software expects to enter.

### Hypothesis (clearly marked)

It is likely (but not proven solely by these list names) that the software selects downlists based on program/major-mode transitions, because `DOWN_TELEMETRY_PROGRAM.agc` explicitly states that downlink list selection is influenced by “V37EXXE” and program types, and because it maintains list pointers like `DNLSTCOD` and `DNTABLE` (see `Luminary099/DOWN_TELEMETRY_PROGRAM.agc` header “DOWNLINK LIST SELECTION” and `Luminary099/DOWNLINK_LISTS.agc: DNTABLE`). To confirm the exact selection logic, trace where `DNLSTCOD` and `DNLSTADR` are updated in mode/program modules.

## See also

1. [Diagrams](diagrams.md) for Mermaid flowcharts and sequences corresponding to these flows.
2. [Execution model](execution-model.md) for the detailed job/task model.
3. [Telemetry and downlink](telemetry-downlink.md) for the list format and the 20 ms handler.
