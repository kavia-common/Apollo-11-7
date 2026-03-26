# How to extend this documentation

This wiki is intended to stay tightly coupled to the Luminary 099 source modules in this repository. The most important rule is: **do not invent behavior**. If something cannot be grounded in the code/comments, mark it explicitly as a hypothesis and clearly separate it from sourced facts.

## Recommended workflow

### 1. Start from the include order

Treat `Luminary099/MAIN.agc` as the authoritative “table of contents” for the software, because it is the monolithic include list and therefore the effective build structure.

Use `Luminary099/README.md` as a human-friendly mirror of this include order (it is explicitly “Derived from MAIN.agc”).

### 2. Ground claims with file + label

When documenting behavior, prefer references like:

- `Luminary099/EXECUTIVE.agc: NOVAC`
- `Luminary099/WAITLIST.agc: T3RUPT`
- `Luminary099/DOWN_TELEMETRY_PROGRAM.agc: DODOWNTM`

If the source only contains prose or a long comment block without a stable label, reference the file and the section title (for example `Luminary099/WAITLIST.agc` header documentation, or `Luminary099/INPUT_OUTPUT_CHANNEL_BIT_DESCRIPTIONS.agc` channel tables).

### 3. Use erasable assignments as your data dictionary

When a routine references a symbol, find its definition in `Luminary099/ERASABLE_ASSIGNMENTS.agc` and document:

1. Whether it is a special register, interrupt storage, a counter (`TIME*`), a job core-set field (`MPAC`, `LOC`, `BANKSET`, `PUSHLOC`, `PRIORITY`), a Waitlist structure (`LST1`, `LST2`), or a subsystem state variable.
2. Any comments about permanence, overlays, or ordering constraints.

This keeps subsystem documentation consistent and prevents incorrect assumptions about memory layout.

### 4. Use channel bit descriptions for I/O documentation

Do not infer channel semantics from code alone if the repository already contains a canonical channel table. Ground I/O claims using `Luminary099/INPUT_OUTPUT_CHANNEL_BIT_DESCRIPTIONS.agc` and then cite the code that reads/writes those channels.

For example, it is safe to state that T4RUPT monitors Proceed via `CHAN32` bit 14 because:

1. The channel/bit meaning is documented in `INPUT_OUTPUT_CHANNEL_BIT_DESCRIPTIONS.agc`.
2. The code explicitly masks `BIT14` against `CHAN32` in `Luminary099/T4RUPT_PROGRAM.agc: PROCEEDE`.

### 5. Expand subsystem deep dives incrementally

For a new subsystem page:

1. Identify the relevant modules from `Luminary099/README.md`.
2. Choose a small number of entry points (labels) and document:
   - Responsibilities (from comments and naming)
   - Key routines & entry points (labels)
   - Shared memory symbols used (from `ERASABLE_ASSIGNMENTS.agc`)
   - Timing/priority considerations (if explicitly documented)
3. Add links to:
   - [Routine reference](routine-reference.md)
   - [Interrupts and I/O](interrupts-io.md) if interrupts are involved
   - [Memory model](memory-model.md) if data layout is important

### 6. Keep diagrams grounded

When adding a new diagram, ensure every box/arrow is supported by at least one source reference. If you must show a relationship that is not explicitly stated, label it as a hypothesis in the surrounding text rather than embedding uncertain claims inside the diagram.

## Suggested next documentation additions

The current wiki focuses on kernel services (Executive, Waitlist, Interpreter, interrupts, telemetry). Natural next expansions that can be grounded by reading additional modules include:

1. Detailed DSKY keyboard paths (e.g., `Luminary099/KEYRUPT_UPRUPT.agc` and pinball modules listed in `Luminary099/README.md`).
2. DAP/autopilot deep dive (modules indexed near `T6-RUPT_PROGRAMS.agc` in `Luminary099/README.md`).
3. Landing guidance phase transitions (modules `THE_LUNAR_LANDING.agc`, `THROTTLE_CONTROL_ROUTINES.agc`, `LUNAR_LANDING_GUIDANCE_EQUATIONS.agc`).

## See also

1. [Directory and file index](directory-file-index.md) for the module map.
2. [Source grounding conventions](index.md) for how this wiki references code.
