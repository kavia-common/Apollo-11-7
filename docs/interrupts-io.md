# Interrupts and I/O

Back to: [Luminary 099 Documentation Wiki](index.md)

Luminary 099 is heavily interrupt-driven. The interrupt model is visible in two layers:

1. Fixed-memory **interrupt lead-ins**, which save minimal context and vector into the correct bank and EBANK.
2. Banked **interrupt handlers**, which implement periodic housekeeping, time-based task dispatch, telemetry downlink handling, and LM DAP timing conventions.

I/O is mediated through AGC channels. Channel numbering and bit assignments are documented in `Luminary099/INPUT_OUTPUT_CHANNEL_BIT_DESCRIPTIONS.agc`, and channel names are also assigned symbolically in `Luminary099/ERASABLE_ASSIGNMENTS.agc`.

## Interrupt vectors (lead-ins)

The fixed “lead-in” entries are in `Luminary099/INTERRUPT_LEAD_INS.agc`. The file begins with `SETLOC 4000`, placing the lead-ins at fixed addresses. Each lead-in pattern:

1. Saves `ARUPT` via `DXCH ARUPT`.
2. Sets `BBANK` (and thus bank context) to a pre-defined `BBCON` constant such as `T4RPTBB` or `T3RPTBB`.
3. Transfers control to the handler label (`TCF T4RUPT`, `TCF T3RUPT`, `DTCB` to `DOT6RUPT`, etc.).

Examples grounded directly in the lead-in file:

1. The **T3** lead-in sets `BBANK` to `T3RPTBB` and branches to `T3RUPT` (`Luminary099/INTERRUPT_LEAD_INS.agc` “T3RUPT” lead-in).
2. The **T4** lead-in sets `BBANK` to `T4RPTBB` and branches to `T4RUPT` (`Luminary099/INTERRUPT_LEAD_INS.agc` “T4RUPT” lead-in).
3. The **DOWNRUPT** lead-in sets `BBANK` to `DWNRPTBB` and branches to `DODOWNTM` (`Luminary099/INTERRUPT_LEAD_INS.agc` “DOWNRUPT” lead-in).
4. The **T6** lead-in uses `DCA T6ADR` followed by `DTCB`, with `T6ADR` defined as `2CADR DOT6RUPT` in the same file (`Luminary099/INTERRUPT_LEAD_INS.agc: T6ADR` and `Luminary099/T6-RUPT_PROGRAMS.agc: DOT6RUPT`).

## Key interrupts in Luminary 099

### T3RUPT: Waitlist dispatch

Waitlist task dispatch is implemented by `Luminary099/WAITLIST.agc: T3RUPT`. The handler:

1. Saves `SUPERBNK` state to `BANKRUPT` and saves `QRUPT`.
2. Updates `LST1` and `TIME3`.
3. Shifts `LST2` and dispatches the next due task by writing `SUPERBNK` and executing `DTCB`.
4. Returns through `TASKOVER` and the `RESUME` logic in the same file (`Luminary099/WAITLIST.agc: TASKOVER`, `RESUME`).

This makes the Waitlist a first-class time-based scheduling mechanism driven by an interrupt.

### T4RUPT: periodic housekeeping and I/O monitoring

The T4 interrupt handler is implemented by `Luminary099/T4RUPT_PROGRAM.agc: T4RUPT`. The module header and comments make clear that it contains periodic routines and monitors.

Several key features are explicit:

1. DSKY display relay output is driven through `OUT0` using `DSPOUT` (`Luminary099/T4RUPT_PROGRAM.agc: DSPOUT`, writing `OUT0`).
2. The Proceed pushbutton is monitored “every 120 milliseconds via channel 32 bit 14 inbit” in the `PROCEEDE` code (`Luminary099/T4RUPT_PROGRAM.agc: PROCEEDE`, using `CHAN32` and `BIT14`).
3. IMU/ISS status bits from channel 30 are monitored and processed by `IMUMON` (`Luminary099/T4RUPT_PROGRAM.agc: IMUMON`), with a table listing bit meanings and associated subroutines (e.g., `ITURNON`, `IMUFAIL`, `ICDUFAIL`, `IMUCAGE`, `IMUOP`).
4. Flip-flop inbits on channel 33 are monitored by `C33TEST` with a bit-to-subroutine mapping, including PIPA fail and downlink/uplink too fast (`Luminary099/T4RUPT_PROGRAM.agc: C33TEST`, and subsequent routines `PIPFAIL`, `DNTMFAST`, `UPTMFAST`).
5. Gimbal lock monitoring is performed by `GLOCKMON` based on `CDUZ` (`Luminary099/T4RUPT_PROGRAM.agc: GLOCKMON`), with explicit thresholds and behavior described in comments.

The `T4JUMP` section shows that T4RUPT dispatches several “once-per-second (0.96 sec actually)” activities based on `RUPTREG1` (`Luminary099/T4RUPT_PROGRAM.agc: T4JUMP`), though those specific targets are defined elsewhere in the module.

### DOWNRUPT: downlink telemetry end pulse (20 ms)

The downlink telemetry interrupt handler is `Luminary099/DOWN_TELEMETRY_PROGRAM.agc: DODOWNTM`. The header states that it is “initiated by telemetry end pulse” and that the pulse occurs “50 times per sec (every 20 ms)”, so `DODOWNTM` is executed at that rate.

The handler selects appropriate data and writes it to output channels 34 and 35 (named `DNTM1` and `DNTM2` in erasable assignments: `Luminary099/ERASABLE_ASSIGNMENTS.agc`). In the program, the `DNTMEXIT` path uses `WRITE DNTM1` and `WRITE DNTM2` (`Luminary099/DOWN_TELEMETRY_PROGRAM.agc: DNTMEXIT`).

The downlink list format itself is documented in `Luminary099/DOWNLINK_LISTS.agc` and further explained in comments in `DOWN_TELEMETRY_PROGRAM.agc`.

### T6RUPT: Time6 interrupt reserved for LM DAP

`Luminary099/T6-RUPT_PROGRAMS.agc` documents strict conventions: “NO NUMBER IS EVER PLACED INTO TIME6 EXCEPT BY LM DAP” and “NO PROGRAM OTHER THAN LM DAP ENABLES THE TIME6 COUNTER” (module header). It also states that programs operating in interrupt mode or with interrupts inhibited must call `T6JOBCHK` every 5 ms to process a possible waiting T6RUPT before hardware honors it (`Luminary099/T6-RUPT_PROGRAMS.agc: T6JOBCHK` and header).

The interrupt handler itself is `DOT6RUPT`, which saves context and calls `T6JOBCHK` before returning to `RESUME` (`Luminary099/T6-RUPT_PROGRAMS.agc: DOT6RUPT`).

## I/O channels used by these interrupts

`Luminary099/INPUT_OUTPUT_CHANNEL_BIT_DESCRIPTIONS.agc` provides the canonical bit-level descriptions. The following are particularly prominent in the interrupt code read in this wiki:

1. `OUT0` (channel 10) is used to transmit relay driving information for display (`INPUT_OUTPUT_CHANNEL_BIT_DESCRIPTIONS.agc` channel 10; `T4RUPT_PROGRAM.agc` writes `OUT0`).
2. `DSALMOUT` (channel 11) drives lamps and engine on/off bits (`INPUT_OUTPUT_CHANNEL_BIT_DESCRIPTIONS.agc` channel 11). The Executive also uses it for activity light control (`Luminary099/EXECUTIVE.agc: DUMMYJOB` uses `WAND/WOR DSALMOUT`).
3. `CHAN12`, `CHAN13`, `CHAN14` carry navigation and hardware control outbits, including IMU control and enabling T6 RUPT (`INPUT_OUTPUT_CHANNEL_BIT_DESCRIPTIONS.agc` channels 12–14). T4RUPT uses these for IMU/ISS initialization sequences and DAP enabling/disabling (`Luminary099/T4RUPT_PROGRAM.agc`, e.g., `ENDTNON2`, `ISSUP`, `CAGESUB`).
4. `CHAN30`..`CHAN33` are input channels with inverted semantics (zero means signal present) per the note in the channel description file. T4RUPT uses `CHAN30`, `CHAN32`, and `CHAN33` directly (`Luminary099/T4RUPT_PROGRAM.agc: IMUMON`, `PROCEEDE`, `C33TEST`).
5. `DNTM1` and `DNTM2` (channels 34 and 35) are downlink word serialization outputs (`INPUT_OUTPUT_CHANNEL_BIT_DESCRIPTIONS.agc` channels 34 and 35; `DOWN_TELEMETRY_PROGRAM.agc: DNTMEXIT` writes them).

## See also

1. [Execution model](execution-model.md) for how `T3RUPT` interacts with Waitlist tasks.
2. [Telemetry and downlink](telemetry-downlink.md) for downlink list structure and data path.
3. [Diagrams](diagrams.md) for sequence diagrams of interrupt handling and data flow.
