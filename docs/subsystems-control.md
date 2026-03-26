# Control (LM DAP / RCS / engine-related control)

Back to: [Luminary 099 Documentation Wiki](index.md)

Luminary contains extensive control logic for the Lunar Module, including the Digital Autopilot (DAP) and its interfaces to reaction control jets and other actuators. This page focuses on control mechanisms that are explicitly described in the modules read for this wiki: Time6 interrupt conventions and key channel interactions.

## Time6 and the LM DAP: a hard contract

`Luminary099/T6-RUPT_PROGRAMS.agc` is explicit that Time6 is reserved for the LM DAP to control RCS jet thrust times. The module header lists conventions that “must not be tampered with”, including:

1. “No number is ever placed into TIME6 except by LM DAP.”
2. “No program other than LM DAP enables the TIME6 counter.”
3. A prescribed sequence for using TIME6 (store positive number, enable clock, interrogate TIME6, and interpret special values including POSMAX and negative zero).
4. Programs that operate in interrupt mode or with interrupts inhibited must call `T6JOBCHK` every 5 ms to process a waiting T6RUPT (`Luminary099/T6-RUPT_PROGRAMS.agc` header and `T6JOBCHK`).

This is one of the clearest control-related architectural constraints in the codebase and should be treated as a “kernel-level” interface contract.

The Time6 interrupt handler is `DOT6RUPT` (`Luminary099/T6-RUPT_PROGRAMS.agc: DOT6RUPT`), which saves `BANKRUPT` and `QRUPT`, calls `T6JOBCHK`, and returns to `RESUME`.

## Control outputs via channels

Control logic interacts with hardware through channels. The channel bit descriptions file provides grounded semantics:

1. Channel 5 and 6 are jet control outputs (“PITCH RCS JET CONTROL” and “ROLL RCS JET CONTROL”) (`Luminary099/INPUT_OUTPUT_CHANNEL_BIT_DESCRIPTIONS.agc` channel 5 and 6).
2. Channel 12 includes gimbal trim commands and radar/IMU controls (bits 9–12 include pitch/roll gimbal trim, and bits 4–6 relate to IMU control) (`Luminary099/INPUT_OUTPUT_CHANNEL_BIT_DESCRIPTIONS.agc` channel 12).
3. Channel 13 includes “ENABLE T6 RUPT” on bit 15 (`Luminary099/INPUT_OUTPUT_CHANNEL_BIT_DESCRIPTIONS.agc` channel 13 bit 15), consistent with Time6 enable behavior described in the Time6 program (`T6-RUPT_PROGRAMS.agc` uses `WOR CHAN13` with `BIT15` at `ENABLET6`).
4. Channel 31 and 32 provide inputs related to controllers and thruster disable bits, and are “used by RCS DAP” per the channel descriptions (`Luminary099/INPUT_OUTPUT_CHANNEL_BIT_DESCRIPTIONS.agc` channel 31 and 32).

## Where DAP state is stored (erasable evidence)

Even without detailing the full DAP algorithm, `Luminary099/ERASABLE_ASSIGNMENTS.agc` contains a clearly named “PERMANENT LEM DAP STORAGE” region and DAP-related erasables, including:

1. `CH5MASK`, `CH6MASK` and `RCSFLAGS` for jet masking and flags.
2. `T5ADR` as a “GENADR of next LM DAP T5RUPT” (a pointer-like structure for DAP timing).
3. Many DAP-specific variables in EBANK 6 and beyond, including desired rates, error terms, and torque reconstruction variables.

These assignments provide navigational anchors for tracing control logic in DAP-related modules such as `DAPIDLER_PROGRAM.agc`, `P-AXIS_RCS_AUTOPILOT.agc`, and `Q_R-AXIS_RCS_AUTOPILOT.agc` (all present in the Luminary index in `Luminary099/README.md`).

## See also

1. [Interrupts and I/O](interrupts-io.md) for how `DOT6RUPT` fits into the interrupt model and how channels are used.
2. [Memory model](memory-model.md) for the placement of DAP storage and Time6 (`TIME6` is assigned in `Luminary099/ERASABLE_ASSIGNMENTS.agc`).
