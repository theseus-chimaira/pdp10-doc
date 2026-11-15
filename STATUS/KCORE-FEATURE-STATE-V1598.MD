# KCORE P1-C/P1-D v1598: physical high media loss, live MEMFS, missing DRM

Date: 2026-11-15. Production DAIMOS VERSION 0.5, original PDP-6
KCORE, MM and filesystem providers. No production kernel changes,
RAM growth, ABI edits or on-disk format changes. Strict 32K/size
optimization stays on hold, and P1-E is held for KA10 design.

## P1-D v1595: actual mounted-provider error versus empty slot PASS

Full physical 256K KCC PDP-6 selected TSFS root on two original
Type-551 reels DTC0/DTC1; DTC2 explicitly empty. Original KINIT
publishes **8** FSINFO slots. Four separately 72-bit-stamped native
DTFS secondary reels on DTC3..DTC6 occupy public mount IDs 2..5.
Actual physical DTC3 DETACH causes original FSINFO on the still
published mount to return **-1** rather than 0; the other three
mounted tapes remain valid including high public IDs 4 and 5.
An unrelated valid replacement reel does not satisfy the old
72-bit identity; the original reel returns and revalidates.
All four secondary mounts then retire without moving IDs.
The complete physical guest and independent five-media identity
oracle PASS; five deliberately invalid raw media/stage controls
rejected. No source, media or MRES modification.

## P1-C v1596: real DTC6 high-slot failure and live MEMFS PASS

Same full 256K PDP-6 TSFS/DTFS geometry plus independent writable
MEMFS mounted at `/MOUNT/RAM` public ID6. After authentic DTC3
failure/replacement/return, physically DETACH live DTC6 (high
public mount ID5). Original FSINFO refuses the unavailable volume,
but all other DTFS mounts remain published and available; MEMFS
performs a genuine file rewrite and verifies all three data words
by readback during the DTC6 outage. The identical original DTC6
physical reel returns, validates its complete media identity and
all mounts retire. Full real guest plus raw media/CTY oracle PASS,
eight unsafe negative controls rejected.

The original Type-551 simulator was itself defective on returning
a previously removed high reel. The old code restored tape file
data/position but retained stale motion/command/delay fields.
The original v1596 guest repeatedly failed DTC6 revalidation on
valid media; an isolated simulator build which resets the fields
made the **identical DAIMOS guest** pass without any kernel edit.
The tested simulator repair was pushed to the separate
`theseus-chimaira/sims` repository as commit `fe97912`, v1598.
The corrected binary is installed as `$PDP10_PREFIX/bin/pdp6-v1598`
and default `pdp6` link; original `pdp6-v1524` retained.

## P1-D v1597: physically unavailable Type-167 processor PASS

Real 128K KCC PDP-6 from four DSK270 D6FSR2 root disks.
The real Type-167 upstream I/O processor `DP` (hardware probe
device 0010) and all downstream Type-236 `DR` units are disabled
before KINIT MINIT, so no DRM236 services are published.
Original FSINFO capacity is exactly **2** (D6FS root provider6
ID1 + MEMFS provider4 ID2), versus 6 when DRM236 is present.
No phantom four unused drum slots; out-of-range indices fail.
Original XIP RUN/WAIT/EXEC, four real D6FSR2 physical members
and four unsafe negative tests PASS. The first test which disabled
DR but left upstream DP enabled failed correctly, exposing a
fixture assumption, not a KCORE defect.

## Open remaining feature work

P1-C: combined actual DSK/DRM BACKSTORE media faults and an
independently executing swapped owner, dirty/write-through provider
faults and repeated physical loaded-simulator timing. Preserve
critical fatal owned-media contract. P1-D: final current-source,
matched image and unusual root/service combinations, not a request
to fabricate 26 physical devices. P1-E on hold for KA10-aware VM
design. VERSION 0.5 and fixed KCORE low end `024364` octal.

Canonical source/testkit reports:
`DAIMOS/design/kcore-p1cd-highslot-provider-physical-v1598.md`,
`daimos-testkit/tests/validation/daimos-fsinfo-high-media-return-physical-v1595.md`,
`daimos-testkit/tests/validation/daimos-p1c-highslot-memfs-return-physical-v1596.md`,
`daimos-testkit/tests/validation/daimos-fsinfo-dsk-no-drm-physical-v1597.md`.
