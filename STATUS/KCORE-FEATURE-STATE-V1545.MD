# KCORE v1545: mounted secondary D6FS physical disk/drum failure

Date: 2026-11-13. Feature acceptance, not optimization.

## Found and fixed a production KINIT calling-convention bug

`module_minit.c` previously patched the D6FS physical DSK270
adapter exports 7/8/11/12 to the relocated low-level MRES driver
entry, bypassing its `dsk270_read_sector` and
`dsk270_write_sector` fixed wrappers. The filesystem supplies
physical `(unit, sector, buffer)` arguments; the low-level MRES
entry requires `(encoded Type-270 controller address, buffer)`.
Without the wrapper, a real mounted secondary D6FS failed its
first FCB READ even before device loss. DRM236 already routed
through its corresponding wrapper. The fix restores the correct
KINIT binding for all four DSK slots. Only reclaimed KINIT
code changes, **zero extra fixed KCORE instructions/data**.

## Actual PDP-6 noncritical mounted-secondary fault isolation

Testkit builds four-member independent D6FS disk/drum media via
the normal installed `mkdsk`/`mkd6fs` producers and checks the
media with `d6fsck`; the 14-word read-only mount handoff comes
from verified real DBOOT descriptors and matching CLEAN
superblocks. Six corrupted handoff cases are rejected. The
real guest PID1 calls `dsys_d6fs_mount`, checks FSINFO public
mount ID 2/provider 6, and opens a real secondary file before
the host physically detaches its media.

* With D6FS root on DRM236 DR0..3, physical **DSK0..3** detachment
  causes the independently mounted secondary D6FS INIT data read
  to return a negative I/O error; a new ROOT file-data read
  succeeds; the PDP-6 does **not** HALT. **PASS.**
* With D6FS root on DSK270 DSK0..3, physical **DR0..3** detachment
  causes the same nonfatal mounted secondary error and surviving
  root file-data read. **PASS.**

Native source/KINIT binding tests reject four wrong patch targets,
and a real PDP-6 DSK270 address-encoding test rejects direct
entry into the MRES with the wrong ABI (five negative mutants
in total). Previous v1544 native DSK/DRM 24 cases/six mutants,
v1513 nonroot return and v1515 distinct HALT tests remain PASS.
No syscall, media format, per-process state or resident RAM change.

## Remaining feature acceptance

Positive critical ROOT loss is accepted for TSFS (all/one tape)
and D6FS (both DSK270 and DRM236 roots). Owned BACKSTORE
Type-236 PI2 error-to-BACKSTORE-HALT has full-guest acceptance.
Secondary DTFS and D6FS mounted disk/drum physical errors now
also have positive noncritical-return acceptance.

**P0-A remains open for shared ROOT/BACKSTORE DEVICE/VOLUME
physical failure ordering**, exact selected critical HALT reason
and preservation of authoritative backing ownership. This is
not another secondary-error test. VERSION 0.5, strict 32K and
general optimization HOLD.

References: DAIMOS `design/kcore-secondary-d6fs-physical-v1545.md`,
daimos-testkit `daimos-secondary-d6fs-physical-v1545.md`.
