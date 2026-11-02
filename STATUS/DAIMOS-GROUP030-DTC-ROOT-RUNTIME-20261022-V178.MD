# DAIMOS group 030: root-selected DECtape runtime V178

Date: 2026-10-22

Against immutable DAIMOS commit `40bc094`, testkit
`daimos-simh-dtc-selected-v34.sh` successfully built and booted a
real `BOOT=dtc ROOT=tape` image on the corrected SIMH PDP-6.
The simulator transcript reported `DTC OK`, `DTFS LOADED`, INIT,
ROOT login and DSH. Both external ECHO and PWD probes passed.
Evidence: `$TMPDIR/daimos-dtfs-root-supported-v178.log` and
`$TMPDIR/daimos-simh-dtc-selected-v34-30045/report-v34.md`.

The testkit now exposes this supported configuration through
`test-daimos-dtfs-tape-root-runtime-v178`, included in both
group-030 matrix definitions.

This does **not** replace the older `dtfs-runtime` integration
fixture. That suite still boots from D6FS and demands a secondary
DTFS mount although `dtfs_minit` installs its provider only when
DECtape is selected as the root. The suite's file creation,
overwrite, rename, deletion and reboot-persistence checks must
eventually be reworked into a valid configuration, without making
DTFS/TSFS permanently resident on D6FS boots. Full group-030
KCC/GCC functional acceptance is not yet complete.
