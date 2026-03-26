# IMU / ISS interfaces

Back to: [Luminary 099 Documentation Wiki](index.md)

Luminary interfaces with the Inertial Measurement Unit / Inertial Subsystem (often referred to as ISS in the source comments) through dedicated channel bits and interrupt-driven monitoring and initialization sequences.

The strongest grounded description of IMU/ISS handling in the codebase is within the T4RUPT program, particularly `IMUMON` and `TNONTEST` in `Luminary099/T4RUPT_PROGRAM.agc`.

## Channel bits used for IMU/ISS

`Luminary099/INPUT_OUTPUT_CHANNEL_BIT_DESCRIPTIONS.agc` lists the IMU/ISS-related inbits on channel 30, including:

1. Bit 15: Temperature within limits
2. Bit 14: ISS turn-on requested
3. Bit 13: IMU fail
4. Bit 12: IMU CDU fail
5. Bit 11: IMU cage command
6. Bit 9: IMU operate with no malfunction

Channel 12 outbits include IMU control and CDU zeroing:

1. Bit 4: Coarse align enable
2. Bit 5: Zero IMU CDUs
3. Bit 6: Enable IMU error counter and CDU error counter
4. Bit 15: ISS turn on delay complete (signal)

These definitions are used directly by T4RUPT’s code paths (`WOR/WAND CHAN12`, `RAND CHAN12`, etc.) in `Luminary099/T4RUPT_PROGRAM.agc`.

## IMUMON: status monitoring loop

`Luminary099/T4RUPT_PROGRAM.agc: IMUMON` is entered periodically (“every 480 ms” per its comment block). It:

1. Compares relevant bits of channel 30 to the previously sampled state in `IMODES30` (`RXOR CHAN30`, `MASK 30RDMSK`).
2. Saves which bits changed and updates `IMODES30`.
3. Scans and dispatches bit-change handlers via a jump table `IFAILJMP` (`Luminary099/T4RUPT_PROGRAM.agc` near `IFAILJMP`).

The comment block above `IMUMON` explicitly lists bit functions and the subroutines called, including `TLIM`, `ITURNON`, `SETISSW` (for failures), `IMUCAGE`, and `IMUOP`.

## TNONTEST: turn-on and initialization sequencing

`Luminary099/T4RUPT_PROGRAM.agc: TNONTEST` is described as honoring requests for ISS initialization. Its comment block describes three forms of initialization:

1. ISS turn-on with caging for 90 seconds and ICDU zeroing.
2. ICDU initialization when only ISS operate is present.
3. Restart with a restartable program using IMU (no initialization).

The implementation uses bits 7 and 8 of `IMODES30` as a two-sample latch mechanism (first sample sets bit 8, second sample triggers action) and schedules a 90-second delay via Waitlist (`TC WAITLIST`, `2CADR ENDTNON`) (`Luminary099/T4RUPT_PROGRAM.agc: PROCTNON` and `ENDTNON`).

The code also issues program alarm `00213` if ISS turn-on is requested without ISS operate (`Luminary099/T4RUPT_PROGRAM.agc: PROCTNON`), consistent with the alarm table in `Luminary099/ASSEMBLY_AND_OPERATION_INFORMATION.agc` which lists `00213` as “IMU not operating with turn-on request” set by `T4RUPT`.

## C33TEST and PIPA fail interaction

`C33TEST` monitors flip-flop inbits on channel 33 and dispatches handling routines such as `PIPFAIL` (`Luminary099/T4RUPT_PROGRAM.agc: C33TEST` and `C33JMP` jump table). `PIPFAIL` can issue program alarm `00212` under a described condition when PIPA fail is present (`Luminary099/T4RUPT_PROGRAM.agc: PIPFAIL`), matching the alarm table in `Luminary099/ASSEMBLY_AND_OPERATION_INFORMATION.agc` which lists `00212` as “PIPA fail but PIPA is not being used.”

## See also

1. [Interrupts and I/O](interrupts-io.md) for how T4RUPT fits into the interrupt model and how channels are used.
2. [DSKY and crew interaction](subsystems-dsky.md) for shared use of DSKY lamp control bits (e.g., ISS warning lamp is driven via `DSALMOUT` bit 1 as described in `INPUT_OUTPUT_CHANNEL_BIT_DESCRIPTIONS.agc` and used by `SETISSW` in T4RUPT).
