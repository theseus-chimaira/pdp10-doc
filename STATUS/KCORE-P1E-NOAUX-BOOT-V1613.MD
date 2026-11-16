# PDP-6 TSFS physical PURE boot, v1613

Date: 2026-11-16.
Feature status: experimental, OFF by default.

Actual AUXST1 discovery returns -1 when no usable
backing descriptor exists. The prior PURE MINIT
branch returned early before installing its module.
KINIT now handles both absent AUXST1 and validated
zero BACKSTORE as no-backing cases. An authenticated
nonzero BACKSTORE still selects the existing disk
MRES, with no changes to storage formats or process
state.

An executed, source-linked C89 KINIT test covers
the three discovery outcomes, D6FS root and
default-OFF behavior, and rejects an unsafe
early-return mutant.

The native SIMH PDP-6 with 64K words, a three-tape
TSFS root and no auxiliary disk/drum attachments
reports MEM 64 K, DTC OK, PURE LOADED and INIT V1.
It stays alive until deliberate test termination.
The transient KINIT diagnostic is emitted after
actual PURE installation and both resident service
trampoline patches. This is a real physical boot
test, not merely a successful KINIT link.

GCC fixed KCORE is byte-identical to v1612:
low end 024254 octal; optional PURE MRES remains
1758 installed words, BSS zero, no new
per-process words.

The historical DISK-style pressure guest was also
adapted experimentally, but both tested 64K
variants failed at their third large child RUN
before proving RAW prefix reclaim or restoration.
These are non-accepting diagnostic failures, not
production scheduler acceptance or proof of a
kernel bug. MEMINFO's resident-process count is
not exact physical occupancy in PACKED mode,
because it excludes the retained suffix MM extent.

Next required tests: establish smaller sleeping
owners first, request controlled BRK/physical
pressure, confirm exact MM RAW suffix retention,
perform slot-0 restore and complete PIPE/WAIT;
measure real physical net savings and temporary
high-water. Also close immutable executable source
lifetime, unmount and failure paths. No compression
or new CPU backend is enabled.

Canonical source and test evidence:
DAIMOS `design/kcore-p1e-noaux-boot-v1613.md`.
