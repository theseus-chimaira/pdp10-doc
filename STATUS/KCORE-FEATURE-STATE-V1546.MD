# KCORE v1546: shared ROOT/BACKSTORE native ordering

Date: 2026-11-13. KCORE feature acceptance, not optimization.

The selected D6FS root and active/owned BACKSTORE may share a
Type-270 disk unit, Type-236 drum unit, or a multi-member VOLUME.
Their logical failure boundaries remain distinct: validated D6FS
I/O that fails after normal driver recovery uses the v1538
provider/mount classifier and root HALT; failed required BACKSTORE
I/O uses the v1541 enabled-or-owned classifier and swap HALT.
The first failed requested service determines which HALT is reached;
the controller alone cannot decide which filesystem is critical.

Actual native PDP-6 testkit v1546 assembles the D6FS reader,
`fs_backing` adapter, fixed BACKSTORE DEVICE/VOLUME gateway,
both v1515 reason-specific HALT leaves and their classifiers
unchanged from production source. The common controller completion,
optional mapper and cache outcome are test-controlled. Twelve
cases PASS: DSK/DRM, root-first and swap-first, VOLUME mapping,
mapper failure, success preceding the other owner's failure,
disabled-but-owned swap and inactive unowned probes. Five unsafe
actual-source mutations are rejected. Exact HALT PCs, unmodified
owned-block count and suppression of second service after the first
fatal failure are asserted. No fixed RAM, ABI or on-media change.

**P0-A still open:** physical full-guest shared ROOT/BACKSTORE
DEVICE/VOLUME fault acceptance with genuine mapper installation,
in-flight transfer arbitration and live swapped-process ownership.
Native test-controlled physical completions do not prove that
full-guest fault case. Previous independent physical v1542/v1545
tests remain accepted. DAIMOS
`design/kcore-shared-root-backstore-v1546.md`; testkit
`tests/validation/daimos-shared-root-backstore-v1546.md`.
VERSION 0.5; strict 32K/optimization HOLD.
