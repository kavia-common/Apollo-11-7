# Diagrams (Mermaid)

Back to: [Luminary 099 Documentation Wiki](index.md)

This page collects Mermaid diagrams for Luminary 099. The diagrams are grounded in the repository’s source modules, especially:

1. Executive and Waitlist scheduling (`Luminary099/EXECUTIVE.agc`, `Luminary099/WAITLIST.agc`).
2. Interrupt lead-ins and interrupt programs (`Luminary099/INTERRUPT_LEAD_INS.agc`, `Luminary099/T4RUPT_PROGRAM.agc`, `Luminary099/T6-RUPT_PROGRAMS.agc`, `Luminary099/DOWN_TELEMETRY_PROGRAM.agc`).
3. Interpreter execution (`Luminary099/INTERPRETER.agc`).
4. I/O and channel semantics (`Luminary099/INPUT_OUTPUT_CHANNEL_BIT_DESCRIPTIONS.agc`, `Luminary099/ERASABLE_ASSIGNMENTS.agc`).
5. Downlink list categories (`Luminary099/DOWNLINK_LISTS.agc`).

## A. System architecture diagram

```mermaid
flowchart LR
  A["Luminary 099 (monolithic build via MAIN.agc)"] --> K["Runtime kernel services"]
  A --> S["Mission and subsystem modules"]
  A --> IO["I/O and device interfaces"]
  A --> INT["Interrupt system"]

  K --> EX["Executive (job scheduler)<br/>EXECUTIVE.agc: NOVAC, FINDVAC, CHANJOB"]
  K --> WL["Waitlist (delayed tasks)<br/>WAITLIST.agc: WAITLIST, T3RUPT, TASKOVER"]
  K --> IP["Interpreter (interpretive engine)<br/>INTERPRETER.agc: INTPRET, DANZIG"]
  K --> BC["Inter-bank calls and superbank<br/>INTER-BANK_COMMUNICATION.agc: BANKCALL, IBNKCALL, SUPERSW"]

  INT --> LEAD["Interrupt lead-ins (fixed entries)<br/>INTERRUPT_LEAD_INS.agc: lead-ins at SETLOC 4000"]
  INT --> T3["T3RUPT task dispatch<br/>WAITLIST.agc: T3RUPT"]
  INT --> T4["T4RUPT periodic services<br/>T4RUPT_PROGRAM.agc: T4RUPT"]
  INT --> T6["T6RUPT for LM DAP timing<br/>T6-RUPT_PROGRAMS.agc: DOT6RUPT, T6JOBCHK"]
  INT --> DN["DOWNRUPT telemetry end pulse<br/>DOWN_TELEMETRY_PROGRAM.agc: DODOWNTM"]

  IO --> CH["Channel definitions and bits<br/>INPUT_OUTPUT_CHANNEL_BIT_DESCRIPTIONS.agc"]
  IO --> DSKY["Display/DSKY outputs<br/>T4RUPT_PROGRAM.agc: DSPOUT writes OUT0"]
  IO --> IMU["IMU/ISS monitoring and init<br/>T4RUPT_PROGRAM.agc: IMUMON, TNONTEST"]
  IO --> TLM["Telemetry outputs (channels 34/35)<br/>DOWN_TELEMETRY_PROGRAM.agc: DNTMEXIT"]

  S --> GNC["Guidance/Navigation/Control modules<br/>Indexed in Luminary099/README.md"]
  S --> DAP["DAP/autopilot modules<br/>Indexed in Luminary099/README.md"]
```

## B. Task scheduling / Executive flow

This diagram focuses on the job/task scheduling services that are explicit in the source.

```mermaid
flowchart TD
  START["A routine requests work"] --> JREQ["Job request via Executive<br/>EXECUTIVE.agc: NOVAC or FINDVAC"]
  JREQ --> ALLOC["Allocate core set (and VAC area if needed)<br/>EXECUTIVE.agc: FINDVAC2, NOVAC2"]
  ALLOC --> RUN["Job becomes runnable (priority bookkeeping)<br/>EXECUTIVE.agc: SETLOC / EJSCAN logic"]
  RUN --> SWITCH{"Is higher priority runnable?"}
  SWITCH -->|Yes| CHG["Change job / swap core sets<br/>EXECUTIVE.agc: CHANJOB"]
  SWITCH -->|No| CONT["Continue current job"]

  CONT --> SLEEPQ["Optional: job sleeps waiting for event<br/>EXECUTIVE.agc: JOBSLEEP"]
  SLEEPQ --> WAKE["Wake on event<br/>EXECUTIVE.agc: JOBWAKE/JOBWAKE2"]
  WAKE --> RUN

  START --> TSK["Schedule delayed task (centiseconds)<br/>WAITLIST.agc: WAITLIST or TWIDDLE"]
  TSK --> T3["Dispatch from T3 interrupt<br/>WAITLIST.agc: T3RUPT"]
  T3 --> TASKRUN["Task runs"]
  TASKRUN --> TEND["Task ends via TASKOVER<br/>WAITLIST.agc: TASKOVER"]
  TEND --> CONT
```

## C. Interrupt handling sequence

This is a generic sequence grounded in how lead-ins save `ARUPT` and vector into handlers (e.g., `INTERRUPT_LEAD_INS.agc` lead-ins) and how several handlers save `QRUPT` and return to resume (`WAITLIST.agc`, `T6-RUPT_PROGRAMS.agc`, `DOWN_TELEMETRY_PROGRAM.agc`).

```mermaid
sequenceDiagram
  participant HW as "Hardware event"
  participant LI as "Interrupt lead-in (fixed)<br/>INTERRUPT_LEAD_INS.agc"
  participant H as "Banked handler (example)"
  participant R as "Resume path"

  HW->>LI: "Interrupt occurs"
  LI->>LI: "Save context (e.g., DXCH ARUPT)"
  LI->>LI: "Set bank (XCH BBANK) and branch"
  LI->>H: "TCF handler (e.g., T4RUPT / T3RUPT / DODOWNTM / DOT6RUPT)"
  H->>H: "Save additional state (often Q via QXCH QRUPT)"
  H->>H: "Do interrupt-specific work"
  H->>R: "Return via RESUME or equivalent"
  R-->>HW: "Resume interrupted execution"
```

## D. Data flow (sensor inputs → navigation → guidance → control outputs)

This diagram is intentionally high level. Its grounding is in channel semantics (input channels 30–33, controller channels 31/32, output channels 5/6/12/13/14, downlink channels 34/35) and in the presence of navigation/guidance/control module groupings in the Luminary index.

```mermaid
flowchart LR
  IN30["Status inbits<br/>CHAN30..CHAN33<br/>INPUT_OUTPUT_CHANNEL_BIT_DESCRIPTIONS.agc"] --> MON["Periodic monitoring<br/>T4RUPT_PROGRAM.agc: IMUMON, C33TEST"]
  IN31["Controller inputs<br/>CHAN31/CHAN32<br/>INPUT_OUTPUT_CHANNEL_BIT_DESCRIPTIONS.agc"] --> DAP["Control laws / DAP<br/>Time6 conventions<br/>T6-RUPT_PROGRAMS.agc"]
  MON --> NAV["Navigation state (erasable state vectors)<br/>ERASABLE_ASSIGNMENTS.agc: RN, VN, etc."]
  NAV --> GUID["Guidance programs and subroutines<br/>Indexed in Luminary099/README.md"]
  GUID --> DAP
  DAP --> OUTJ["Jet outputs<br/>CHAN5/CHAN6<br/>INPUT_OUTPUT_CHANNEL_BIT_DESCRIPTIONS.agc"]
  DAP --> OUTHW["Hardware outputs<br/>CHAN12..CHAN14<br/>INPUT_OUTPUT_CHANNEL_BIT_DESCRIPTIONS.agc"]
  NAV --> TLM["Downlink selection and output<br/>DOWN_TELEMETRY_PROGRAM.agc: DODOWNTM"]
  TLM --> OUTDN["Downlink channels 34/35<br/>DOWN_TELEMETRY_PROGRAM.agc: DNTMEXIT"]
```

## E. Key mission-mode flow (grounded by downlink list categories)

This flow is grounded in the existence and naming of downlink lists in `Luminary099/DOWNLINK_LISTS.agc`. It does not claim the exact mission program sequencing; it provides a source-grounded “phase vocabulary” that Luminary supports for telemetry contexts.

```mermaid
flowchart TD
  P0["Operational context"] --> COAST["Coast and alignment downlist<br/>DOWNLINK_LISTS.agc: LMCSTADL"]
  P0 --> ORB["Orbital maneuvers downlist<br/>DOWNLINK_LISTS.agc: LMORBMDL"]
  P0 --> REND["Rendezvous and pre-thrust downlist<br/>DOWNLINK_LISTS.agc: LMRENDDL"]
  P0 --> DSAS["Descent and ascent downlist<br/>DOWNLINK_LISTS.agc: LMDSASDL"]
  P0 --> SURF["Lunar surface align downlist<br/>DOWNLINK_LISTS.agc: LMLSALDL"]
  P0 --> AGS["AGS initialization/update downlist<br/>DOWNLINK_LISTS.agc: LMAGSIDL"]

  COAST --> DNPGM["Downlink program uses list pointers<br/>DOWN_TELEMETRY_PROGRAM.agc: DODOWNTM"]
  ORB --> DNPGM
  REND --> DNPGM
  DSAS --> DNPGM
  SURF --> DNPGM
  AGS --> DNPGM
```

## See also

1. [Architecture](architecture.md) for textual decomposition matching Diagram A.
2. [Execution model](execution-model.md) for Executive/Waitlist details matching Diagram B.
3. [Interrupts and I/O](interrupts-io.md) for interrupt lead-ins and channel usage matching Diagram C/D.
4. [Telemetry and downlink](telemetry-downlink.md) for list formats and handlers matching Diagram E.
