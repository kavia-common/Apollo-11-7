# DSKY and crew interaction

Back to: [Luminary 099 Documentation Wiki](index.md)

The DSKY (Display and Keyboard) is Luminary’s primary crew interface. In the source modules read for this wiki, two aspects are especially explicit:

1. The downlink and channel documentation that defines which bits drive indicator lamps and how key input arrives.
2. The T4RUPT display output path that drives relay rows via `OUT0` and monitors the Proceed key.

This page focuses on those grounded aspects.

## DSKY output channels and lamp bits

`Luminary099/INPUT_OUTPUT_CHANNEL_BIT_DESCRIPTIONS.agc` describes several channels directly relevant to DSKY output:

1. `OUT0` (channel 10) is used to transmit “latching-relay driving information for the display system,” with row selection in bits 15–12 and relay settings in bits 11–1.
2. `DSALMOUT` (channel 11) drives individual indicators and engine on/off control. The file lists bit meanings including:
   - Bit 1: ISS warning
   - Bit 2: Computer activity lamp
   - Bit 4: Temp caution lamp
   - Bit 5: Keyboard release lamp
   - Bit 6: Flash verb and noun lamps
   - Bit 7: Operator error lamp
   - Bits 13/14: Engine on/off

These definitions are used by runtime code. For example, the Executive idling logic turns the activity light off and on using `DSALMOUT` (`Luminary099/EXECUTIVE.agc: DUMMYJOB`, `NUDIRECT`). T4RUPT also uses `DSALMOUT` for lamp control (for example the temp lamp logic in `Luminary099/T4RUPT_PROGRAM.agc: TLIM`).

## Keyboard input channels and interrupts

The channel description file defines:

1. `MNKEYIN` (channel 15) as key code input “sensed by program when program interrupt #5 is received” (bits 5–1).
2. `NAVKEYIN` (channel 16) as optics mark information and navigation panel / thrust control “sensed by program when program interrupt #6 is received” (bits 3–7).

The interrupt lead-ins include explicit entries for keyboard-related interrupts:

1. `KEYRUPT1` lead-in branches to `KEYRUPT1` with `BBANK` set to `KEYRPTBB` (`Luminary099/INTERRUPT_LEAD_INS.agc`).
2. `KEYRUPT2` lead-in branches to `MARKRUPT` with `BBANK` set to `MKRUPTBB` (`Luminary099/INTERRUPT_LEAD_INS.agc`).

The detailed keyboard handling routines are in `Luminary099/KEYRUPT_UPRUPT.agc` and the “pinball” modules (as seen in the Luminary index in `Luminary099/README.md`), but this page does not assert specifics beyond the grounded interrupt entry points and channel assignments.

## Display output path in T4RUPT

The T4RUPT handler contains a display output mechanism that writes `OUT0` based on a table and “relay row” encoding:

1. The code defines a packed relay table `RELTAB` and writes `OUT0` after combining relay row bits and data bits (`Luminary099/T4RUPT_PROGRAM.agc`, `RELTAB` and the `DSPLAY` logic).
2. The `DSPOUT` routine checks whether a “DSKY flag” is on and whether there are pending display requests (`Luminary099/T4RUPT_PROGRAM.agc: DSPOUT`, referencing `FLAGWRD5` and `NOUT`).

Because `OUT0` is described as the relay driving channel in `INPUT_OUTPUT_CHANNEL_BIT_DESCRIPTIONS.agc`, this forms a concrete source-grounded pipeline: T4RUPT → DSPOUT/DSPLAY → `WRITE OUT0` → display relays.

## Proceed key monitoring

T4RUPT includes a dedicated `PROCEEDE` routine that monitors the Proceed pushbutton via channel 32 bit 14:

- The comment block states the status is monitored every 120 ms and processed based on transitions.
- The code uses `RXOR CHAN32` and masks `BIT14` and updates `IMODES33` accordingly, potentially requesting `PROCKEY` via the Executive (`TC NOVAC`, `2CADR PROCKEY`) (`Luminary099/T4RUPT_PROGRAM.agc: PROCEEDE`).

This is a clear example of crew input (Proceed) being sampled in an interrupt and then converted into scheduled work via the Executive.

## See also

1. [Interrupts and I/O](interrupts-io.md) for where keyboard interrupts and T4RUPT fit in the interrupt model.
2. [Execution model](execution-model.md) for how `NOVAC` job requests are entered (as used by `PROCEEDE`).
3. [IMU / ISS interfaces](subsystems-imu.md) for the IMU/ISS status bits that drive warnings and for the T4RUPT monitoring paths that interact with display and alarms.
4. [Telemetry and downlink](telemetry-downlink.md) for the downrupt-driven telemetry path that often gets analyzed alongside crew-visible status and monitoring.
