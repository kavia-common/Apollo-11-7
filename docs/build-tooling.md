# Build and tooling

Luminary 099 in this repository is an archival transcription targeted at the `yaYUL` assembler (part of the Virtual AGC toolchain). The repository itself is primarily a source archive and does not include a complete build pipeline; instead, it provides the source in a `yaYUL`-compatible format and relies on external tooling for assembly.

This page documents what can be grounded directly in the repository sources.

## Build structure (monolithic include)

The Luminary source is assembled as a single monolithic listing through inclusion, not by assembling independent files and linking them. This is stated explicitly in `Luminary099/README.md`, which explains that the original development team did not use a linker and therefore the build was monolithic.

In this repository, `Luminary099/MAIN.agc` is the include driver that re-joins the split modules. Each line of `MAIN.agc` includes a module and often includes page number ranges corresponding to the original hardcopy listing. The module list in `Luminary099/README.md` is “Derived from MAIN.agc” and provides the same index.

## Assembler and format

The Luminary listing is adapted for `yaYUL`, which “accepts a slightly different format” than the original YUL/GAP assemblers. This is also stated in `Luminary099/README.md`.

The source modules include header comment blocks that indicate:

1. `Assembler: yaYUL` (for example `Luminary099/EXECUTIVE.agc`, `Luminary099/WAITLIST.agc`, `Luminary099/INTERPRETER.agc`).
2. Original program identity and assembly provenance (for example `Luminary099/ASSEMBLY_AND_OPERATION_INFORMATION.agc` describing “Assemble revision 001 of AGC program LMY99 by NASA 2021112-061”).

## Compiling outside this repository

The top-level repository README indicates that users interested in compiling should check out “Virtual AGC” (`Apollo-11-7/README.md`).

Because this repository is an archive, this wiki does not claim a specific command line or CI process for building Luminary. If you intend to assemble it:

1. Use the Virtual AGC / yaYUL toolchain.
2. Assemble using `Luminary099/MAIN.agc` as the top-level source (because it includes the full module list in assembly order).
3. Expect the source to be organized as include modules rather than independent compilation units.

## See also

1. [Directory and file index](directory-file-index.md) for the module catalog and which modules are “kernel-like”.
2. [Memory model](memory-model.md) for how banking and erasable assignments shape the code organization.
