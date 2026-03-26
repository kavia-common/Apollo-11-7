# Luminary 099 Documentation Wiki

This wiki documents the Apollo 11 Lunar Module (LM) Apollo Guidance Computer (AGC) flight software **Luminary 099** (“LMY99”) as archived in this repository. The intent is to help software engineers, historians, and real-time/embedded programmers understand how Luminary is organized, how it executes, how it uses memory, how it services interrupts and I/O, and where to find key routines in the source.

The Luminary source is split into modules which are included into a single monolithic build using `MAIN.agc`. This matches the historical “single assembly” build style described in `Luminary099/README.md` and `Luminary099/ASSEMBLY_AND_OPERATION_INFORMATION.agc`.

## Start here

If you are new to AGC software or to this repository, start with the overview page and then follow the execution/interrupt pages before diving into subsystems.

## Table of contents

### Overview and orientation

Read these first to learn how to navigate the code.

1. [Overview](overview.md)
2. [Directory and file index](directory-file-index.md)
3. [Glossary](glossary.md)

### Architecture and execution

These pages explain the system decomposition, runtime model, and the most important “kernel-like” services.

1. [Architecture](architecture.md)
2. [Memory model](memory-model.md)
3. [Execution model (Executive + Waitlist)](execution-model.md)
4. [Interrupts and I/O](interrupts-io.md)
5. [Interpreter (interpretive instruction engine)](interpreter.md)
6. [Telemetry and downlink](telemetry-downlink.md)

### Build and tooling

1. [Build and tooling](build-tooling.md)

### Key subsystems (deep dives)

These pages provide subsystem-level context and point you to key modules and entry points.

1. [Guidance and mission programs](subsystems-guidance.md)
2. [Navigation and state propagation](subsystems-navigation.md)
3. [Control (LM DAP / RCS / engine-related control)](subsystems-control.md)
4. [DSKY and crew interaction](subsystems-dsky.md)
5. [IMU / ISS interfaces](subsystems-imu.md)

### Program flow maps and diagrams

1. [Program flow maps](program-flow-maps.md)
2. [Diagrams (Mermaid)](diagrams.md)

### Routine reference

1. [Key routines reference](routine-reference.md)

### Contributing to this wiki

1. [How to extend this documentation](how-to-extend.md)

## Source grounding conventions

Throughout this wiki, claims are grounded using references of the form:

- `Luminary099/<file>.agc: <label>` for a routine/entry label, or
- `Luminary099/<file>.agc: <commented section name>` when the source uses prose rather than a single label.

For example, the Executive’s “find a VAC area” routine is referenced as `Luminary099/EXECUTIVE.agc: FINDVAC2`, and the Waitlist’s task dispatch interrupt is referenced as `Luminary099/WAITLIST.agc: T3RUPT`.

## See also

If you need the exact include order for the full build, see `Luminary099/MAIN.agc`. For the module index and page mapping back to the scanned listing, see `Luminary099/README.md`.
