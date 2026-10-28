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
