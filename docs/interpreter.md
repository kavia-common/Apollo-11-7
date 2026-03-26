# Interpreter (interpretive instruction engine)

Back to: [Luminary 099 Documentation Wiki](index.md)

Luminary includes an interpretive instruction engine that executes compact “interpretive programs” stored in fixed memory. The interpreter is implemented in `Luminary099/INTERPRETER.agc` and uses interpretive constants defined in `Luminary099/INTERPRETIVE_CONSTANT.agc`.

This page documents the interpreter’s execution model and interfaces as they appear in the source.

## Entry, resume, and dispatch loop

### Entry point: INTPRET

The interpreter entry point is `Luminary099/INTERPRETER.agc: INTPRET`. The source comments describe the setup performed at entry:

1. `LOC` is set to the first interpretive instruction (the word following the `TC` that entered the interpreter).
2. `BANKSET` is set to the `BBANK` of the interpretive program.
3. `INTBIT15` captures bit 15 content of `FBANK`, which is used to enforce a restriction: programs in high banks do not refer to low banks and vice versa, and the interpreter does not switch superbanks.

The interpreter ensures `EDOP` (edited opcode pair state) is cleared and then enters `NEWOPS` to fetch the next opcode pair.

### Resume: INTRSM

Interpretive jobs can be suspended and later resumed by `INTRSM` (`Luminary099/INTERPRETER.agc: INTRSM`). This is relevant because the Executive can switch away from an interpretive job when a higher priority job is runnable, and then later resume it. The Executive explicitly branches to `INTRSM` after interpretive job change epilogue (`Luminary099/EXECUTIVE.agc: ENDPRCHG`).

### The main loop: DANZIG and NEWOPS

Most interpretive operations ultimately return to `DANZIG` (`Luminary099/INTERPRETER.agc: DANZIG`). `DANZIG`:

1. Restores `BBANK` from `BANKSET`.
2. Checks whether there is a leftover operation code in `EDOP` and dispatches it if present.
3. Checks `NEWJOB` to see whether the Executive indicates a higher-priority job is runnable; if so, it transfers to `CHANG2` to change jobs (`Luminary099/INTERPRETER.agc: DANZIG` and `CHANG2` referenced in Executive).
4. Advances `LOC` and fetches the next opcode pair via `NEWOPS`.

This explicitly couples interpretive execution to the Executive’s job scheduling mechanism.

## Addressing model and bank switching

The interpreter supports multiple operand addressing modes.

### Direct addresses and work area vs general erasable

Direct address decoding routes through `ADDRESS` and `DIRADRES` (`Luminary099/INTERPRETER.agc: ADDRESS`, `DIRADRES`).

A key rule is visible in the comments and code paths:

1. Addresses less than `45D` are treated as relative to the work area base stored in `FIXLOC` (for example `NETZERO` adds `FIXLOC`).
2. General erasable addressing triggers E-bank switching via `EBANK` updates (see `GEADDR` which sets `EBANK` and constructs a subaddress).
3. Fixed memory addressing triggers F-bank switching via `FBANK` and the low-10-bit subaddress scheme (`IERASTST` fixed bank path).

### Indexed addresses

The `INDEX` routine (`Luminary099/INTERPRETER.agc: INDEX`) processes indexed addresses using an interpretive index register (e.g., `X1`, `X2`) and handles address classes (work area, erasable, fixed) after applying the index.

### Pushdown list (“push up”)

When no explicit operand address is given, `PUSHUP` logic obtains operands from the pushdown list based on `PUSHLOC` and the current `MODE` (`Luminary099/INTERPRETER.agc: PUSHUP`, `REGUP`). This makes the interpreter somewhat stack-machine-like, and it means that interpretive programs can be compact by omitting explicit operand addresses.

## Store codes and calling

The interpreter supports “store codes” as a primary output mechanism. The store code dispatch is implemented in `DOSTORE` and `STORJUMP` (`Luminary099/INTERPRETER.agc: DOSTORE`, `STORJUMP`).

The documentation in `INTERPRETER.agc` describes:

1. `STORE` (store MPAC)
2. `STODL` and `STOVL` (store then reload, potentially from pushdown list)
3. `STCALL` (store and call)

The control-transfer primitives include:

1. `CALL` which sets up `QPRET` and transfers to a subroutine (`Luminary099/INTERPRETER.agc: CALL`).
2. `GOTO` and `CGOTO` which change the interpretive sequence (`Luminary099/INTERPRETER.agc: GOTO`, `CGOTO`).
3. `RTB` which returns to basic language at a given address (`Luminary099/INTERPRETER.agc: RTB/BHIZ`).

## Interpretive constants

The file `Luminary099/INTERPRETIVE_CONSTANT.agc` defines constants used by interpretive programs in specific fixed locations (`SETLOC INTPRET1` and `SETLOC INTPRET2`). Examples include:

1. `DP1/4TH` and `DPHALF` (via `UNITX`).
2. Unit vectors (e.g., `UNITX`, `UNITY`, `UNITZ` and the alternate `XUNIT`, `YUNIT`, `ZUNIT`).
3. Special constants such as `LODPMAX` and a sequence of `-0, -6, -12` that are required to remain in order (`Luminary099/INTERPRETIVE_CONSTANT.agc` comments near `DFC-6` and `DFC-12`).

Because these are fixed-located tables, many interpretive routines assume their placement.

## See also

1. [Execution model](execution-model.md) for how interpretive jobs are scheduled and swapped by the Executive.
2. [Inter-bank communication](architecture.md) for `BANKCALL`/`IBNKCALL` and superbank setting, which interpretive programs and interrupt code rely on (`Luminary099/INTER-BANK_COMMUNICATION.agc`).
