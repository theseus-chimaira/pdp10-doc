# DAIMOS regression row 023 acceptance — v565, 2026-10-28

The exact ten matrix targets in row 023 of
`daimos-testkit/tests/validation/daimos-regression-matrix-v1.tsv` were
executed in order against current DAIMOS/testkit source. **All passed.**

The first target, `test-daimos-monitorfs-devices-online`, initially crashed
SIMH with exit 139 and SDL2 window-event diagnostics, before any MonitorFS
guest assertion failure. Its testkit runner used graphical `boot.ini`
rather than the established `daimos-simh-headless.sh` policy. The v564 fix
uses the shared headless config (with unique DCS and GE ports) before
appending the test-specific device detach/disable commands. No guest oracle
or production kernel code was changed. On a fresh rebuilt image it passed.
Testkit commit: `c5bb9b7`.

The nine remaining row-023 targets all passed as a sequential continuation:
D6FS context dispatch; D6FS multimount; D6FS direct backing on DRM;
BCACHE multisource; D6FS remount; D6FS superblock recovery; VFS storage
reservations; BADMAP mapping; and BADMAP root recovery.

DAIMOS TODO acceptance commit: `3dbb9a5`. This closes **row 023 only**;
other GCC/KCC grouped tests, secondary mount acceptance, and the complete
matrix remain distinct gates. Neither resident RAM nor production KCORE
code changed.

## Subsequent storage rows v566–v567

On 2026-10-28 the next two complete matrix rows were also run in target
order, using the unchanged production kernel and test oracles:

- Row 024 (v566): `test-daimos-logstore-v1` PASS;
  `test-daimos-mtc516-online` PASS.
- Row 025 (v567): `test-daimos-dsk270-online` PASS;
  `test-daimos-dsk270-elevator` PASS.

The three adjacent accepted rows 023–025 therefore cover fourteen matrix
targets. This is **not** the complete GCC/KCC acceptance suite. DAIMOS
TODO commit `4e53a0c` records the additional two rows. No kernel or
persistent RAM modifications were required.

## DRM236 row 026 v568–v570

All four row-026 matrix targets passed in order. `drm236-capacity` and
`drm236-online` passed unchanged. `drm236-minit` initially failed because
its assembly/linkage-only fixture invoked the **default full** permanent-size
profile (`SINGLE_ROOT last=040610`, `MULTI_ROOT last=040651`, limit `040000`).
This is an independently known full-profile address-budget limitation, not
an MINIT functional assertion. The testkit runner was corrected to request
`BOOT_PROFILE=lowmem ROOT=drum`, keeping original DRM MRES symbols and
KINIT package checks. On fresh build it passed; testkit commit `6184fad`.
The final `drm-stage1-boot` test then passed the actual boot path DRM Stage1
→ DRM/D6FS → INIT → LOGIN with DSK0 unattached. DAIMOS acceptance commit
`69400e0`. This does not waive the full-profile size gate.
