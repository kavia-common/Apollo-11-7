# Navigation and state propagation

Back to: [Luminary 099 Documentation Wiki](index.md)

Navigation in Luminary 099 comprises state representation, propagation (integration), and measurement incorporation. This wiki page focuses on the concrete artifacts visible in the source tree: state vectors and their storage, and the modules that the Luminary index identifies as integration/navigation-related.

## State vectors and permanent navigation storage

`Luminary099/ERASABLE_ASSIGNMENTS.agc` defines a set of state vectors that are explicitly permanent “whole mission” quantities and additional vectors used for downlink.

Grounded examples include:

1. Permanent state vectors `RN` and `VN` (“PERM STATE VECTORS FOR BOOST AND DOWNLINK — WHOLE MISSION”) (`Luminary099/ERASABLE_ASSIGNMENTS.agc`).
2. Permanent CSM and LM state vectors (`RRECTCSM`, `VRECTCSM`, `RRECTLEM`, `VRECTLEM`) and associated times, deltas, and Kepler terms (`TETCSM`, `DELTACSM`, `NUVCSM`, etc.) (`Luminary099/ERASABLE_ASSIGNMENTS.agc`).
3. Explicit “STATE VECTORS FOR DOWNLINK” such as `R-OTHER` and `V-OTHER` (`Luminary099/ERASABLE_ASSIGNMENTS.agc`).

These definitions are important because they act as stable anchors for navigating integration and navigation logic. Many mission programs will refer to these symbols rather than raw numeric addresses.

## Navigation-related modules (by source index)

The Luminary source index in `Luminary099/README.md` and the include list in `Luminary099/MAIN.agc` identify several navigation-related modules:

1. `Luminary099/INTEGRATION_INITIALIZATION.agc`
2. `Luminary099/ORBITAL_INTEGRATION.agc`
3. `Luminary099/MEASUREMENT_INCORPORATION.agc`
4. `Luminary099/CONIC_SUBROUTINES.agc`
5. `Luminary099/LATITUDE_LONGITUDE_SUBROUTINES.agc`
6. `Luminary099/PLANETARY_INERTIAL_ORIENTATION.agc`

Because these are discrete modules included in the monolithic build, the most reliable way to locate specific navigation behavior is to search within these modules for named entry points and comments, and then correlate referenced erasables back to `ERASABLE_ASSIGNMENTS.agc`.

## Interactions with interrupts and timing

Navigation computations are often timed or triggered by runtime services:

1. Waitlist can schedule tasks in centiseconds (`Luminary099/WAITLIST.agc: WAITLIST`) and dispatch them via `T3RUPT` (`Luminary099/WAITLIST.agc: T3RUPT`), which is a common pattern for time-based updates.
2. T4RUPT runs periodic housekeeping and status monitoring (`Luminary099/T4RUPT_PROGRAM.agc: T4RUPT`). While the T4RUPT module excerpted here focuses on display, IMU status processing, and radar monitoring, it demonstrates how periodic entry points are implemented.

The exact timing of navigation propagation loops is not asserted here because it depends on specific program modules; this page instead provides the grounded hooks that allow you to trace timing through Waitlist calls and interrupt entry points.

## See also

1. [Memory model](memory-model.md) for how these state vectors are laid out and banked.
2. [Telemetry and downlink](telemetry-downlink.md) for how state vectors are packaged into downlink lists (`Luminary099/DOWNLINK_LISTS.agc` includes lists that reference `RN`, `VN`, `R-OTHER`, `V-OTHER`, and other navigation-related erasables).
