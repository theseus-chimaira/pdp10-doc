# Installed SIMS full DAIMOS boot, 2026-11-27 v5

PASS: fresh full-profile disk-root image from DAIMOS 18a44866 booted on
installed SIMS cf52e6c, PDP-6, 256K words, 60 Hz, realtime PCLK,
Type 340 GUI and color scope with SDL dummy driver. Default experimental
kernel options retained. KCC host compiler, native bootstrap 82f074c.

Private source build, Type 270 disk boot, four-member D6FS root; preboot
D6FS check clean (179196 blocks, 360 FCBs). Auxiliary drum converted using
the DAIMOS source-built mkauxstore. CTY, PCLK, PTR, PTP, LPT, CR, CP, DCS,
GE, DPY, DSK270, DRM236, SLV reported OK; TTY, WCNSLS, OCNSLS, BSET,
LOGSTR, MEMFS, D6FS loaded. INIT V1, ROOT login, DSH V1 reached.
Existing testkit SIMH harness v4: ECHO SIMSUPSTREAMBOOTOK, LS /SYSTEM/EXEC,
LS / all PASS (3/3). No simulator or kernel source change required.

Prerequisites corrected during build:
- Missing native KCC bootstrap in PDP10_PREFIX. Rebuild exposed v59u
  sprintf const-format prototype mismatch; corrected implementation and
  formatter signature in KCC 82f074c, pushed after preserving 999e013.
  Documented make install-native completed and installed all five DXRs
  and supporting runtime/libc objects.
- Installed mkauxstore is a different variant without --convert-dboot
  and --confirm-destructive. Built the matching DAIMOS utility with
  strict C99/-Werror and selected it using MKAUXSTORE=<private build path>.
  Did not replace the other installed variant or modify source.

Evidence under TMPDIR/daimos-installed-sims-fullboot-v20261127-5.VU0ZS0:
  build.log, kcc-install.log, kcc-install-retry.log, build-retry.log,
  build-retry2.log, run/simh.log, run/results.tsv, report.md.

Scope: boot and interactive shell/file listing only; not full device I/O,
physical graphics, low-memory certification, native KCC self-compilation,
or guest clean shutdown. Harness terminates the private simulator.
