# Guidance and mission programs

Back to: [Luminary 099 Documentation Wiki](index.md)

Luminary’s mission logic is organized largely as “programs” and associated subroutines (e.g., P20–P25, P40–P47, P63/P64/P65/P66/P67, etc.) with supporting math and navigation routines. The authoritative module index for Luminary 099 is the “Source Code Index” in `Luminary099/README.md` (derived from `Luminary099/MAIN.agc`) and the “TABLE OF SUBROUTINE LOG SECTIONS” in `Luminary099/ASSEMBLY_AND_OPERATION_INFORMATION.agc`.

This page focuses on how to locate and reason about guidance-related modules in the repository, without inventing behavior beyond what is present in the source index.

## Where guidance lives in the source tree

The include order in `Luminary099/MAIN.agc` and the index in `Luminary099/README.md` show several major guidance-related groupings:

1. Rendezvous and orbital maneuver programs, including:
   - `Luminary099/P20-P25.agc`
   - `Luminary099/P30_P37.agc`
   - `Luminary099/P32-P35_P72-P75.agc`
   - `Luminary099/P34-35_P74-75.agc`
   - `Luminary099/R30.agc`, `Luminary099/R31.agc`, and `Luminary099/R63.agc` (display/routine support in the index)

2. Powered flight and landing-related modules, including:
   - `Luminary099/BURN_BABY_BURN--MASTER_IGNITION_ROUTINE.agc`
   - `Luminary099/P40-P47.agc`
   - `Luminary099/THE_LUNAR_LANDING.agc`
   - `Luminary099/THROTTLE_CONTROL_ROUTINES.agc`
   - `Luminary099/LUNAR_LANDING_GUIDANCE_EQUATIONS.agc`
   - `Luminary099/P70-P71.agc`
   - `Luminary099/P12.agc`
   - `Luminary099/ASCENT_GUIDANCE.agc`

3. Shared math / propagation and geometry that guidance commonly calls into, including:
   - `Luminary099/CONIC_SUBROUTINES.agc`
   - `Luminary099/ORBITAL_INTEGRATION.agc`
   - `Luminary099/INTEGRATION_INITIALIZATION.agc`
   - `Luminary099/POWERED_FLIGHT_SUBROUTINES.agc`
   - `Luminary099/TIME_OF_FREE_FALL.agc`
   - `Luminary099/LEM_GEOMETRY.agc`

These modules are explicitly present in the Luminary index and are included in the monolithic build.

## Interfaces to runtime services

Guidance programs use runtime services rather than “owning” CPU time directly:

1. Time-based activities can be scheduled using Waitlist (`Luminary099/WAITLIST.agc: WAITLIST`, `TWIDDLE`, `VARDELAY`, `FIXDELAY`, `LONGCALL`).
2. Higher-level work can be requested through the Executive using job entry routines (`Luminary099/EXECUTIVE.agc: NOVAC`, `FINDVAC`), which is how interpretive jobs or other priority work can be introduced.
3. Periodic monitoring and device interaction may occur in interrupts, especially T4RUPT (`Luminary099/T4RUPT_PROGRAM.agc: T4RUPT` and its subroutines).

## Landing and ascent: visible evidence in erasable assignments

Even without reading every mission program module, `Luminary099/ERASABLE_ASSIGNMENTS.agc` gives strong evidence about which guidance phases exist and which shared state they require, because it defines dedicated erasables and overlays that are named for particular phases.

Examples grounded in erasable assignments:

1. “LUNAR LANDING STORAGE” includes `RLS` (landing site vector) in EBANK 4 (`Luminary099/ERASABLE_ASSIGNMENTS.agc`).
2. A large W-matrix region is defined in EBANK 5, and the file contains explicit “W-MATRIX PADLOADS” and numerous landing-phase parameters such as `TLAND`, `GAINBRAK`, `GAINAPPR`, etc. (`Luminary099/ERASABLE_ASSIGNMENTS.agc`, EBANK 5).
3. The file defines “SECOND DPS GUIDANCE (LUNAR LANDING)” overlays and display quantities such as `VHORIZ`, `TTF/8`, `DELTAH`, `FUNNYDSP`, and notes about not sharing during P63–P67 (`Luminary099/ERASABLE_ASSIGNMENTS.agc`, EBANK 7 overlay regions).
4. “ASCENT GUIDANCE ERASABLES” are explicitly called out in EBANK 7 overlay 5 (`Luminary099/ERASABLE_ASSIGNMENTS.agc`).

These names are not sufficient to document full algorithms, but they are strong navigational anchors for finding where those phases are implemented and what shared state they manipulate.

## See also

1. [Program flow maps](program-flow-maps.md) for how startup, Executive, Waitlist, and interrupts interact.
2. [Subsystems: Navigation](subsystems-navigation.md) for propagation and state vector storage that guidance depends on.
3. [Subsystems: Control](subsystems-control.md) for DAP/jet timing constraints (notably `TIME6` conventions in `Luminary099/T6-RUPT_PROGRAMS.agc`).
