# KCORE v1541: active or owned BACKSTORE service is critical

Date: 2026-11-13. This is KCORE feature work only. VERSION remains
0.5; general optimization and strict 32K size work remain on HOLD.

## Selected root versus BACKSTORE

KCORE now uses the same principle for its two mandatory storage
services: an actual required operation that cannot complete after
the normal bounded controller recovery is a terminal failure. The
system does not need to prove that the physical device was removed
permanently. These are two distinct no-return v1515 HALT PCs.

- Selected D6FS/TSFS root: the v1538 gate checks the requesting
  provider and public mount ID. A required negative physical read/
  write reaches the **root** HALT. Unrelated mounted filesystem
  failures are local; no resource owner is falsely reconstructed.
- BACKSTORE: the v1541 fixed `fs_mres.s` transfer gateway handles
  direct DSK270/DRM236 DEVICE and mapped BLOCKSET VOLUME alike.
  After the driver's actual final negative READ/WRITE completion,
  it selects v1515 **BACKSTORE** HALT reason 2 when either
  `backstore_enabled` or `backstore_blocks_used` is nonzero.
  Turning off new swap admissions never discards outstanding owners.
  Disabled, unowned optional KINIT snapshot probes and invalid or
  zero-length requests keep their existing nonfatal behavior.

The BACKSTORE change adds no new permanent state or ABI and no
controller-specific media-absence heuristic. It does not free,
reconstruct, unpin or publish any process on a fatal path. The
v1511/v1512 accepted demonstrations of eventual *scheduler-level*
recovery after a negative BACKSTORE I/O are historical now: the
explicit v1541 policy HALTs when that final failure occurs. A
controller error successfully recovered *before* the driver returns
does not trigger the fatal gate.

## Accepted validation

Source-linked native PDP-6 KCORE tests execute 17 success/failure/
invalid/probe cases and reject six unsafe source mutations, including
reason mix-up, disabled-but-owned state and missed VOLUME failure.
Full 256K tape-root KCC fixed KCORE and optional filesystem MRES
links PASS. No new DATA/BSS and no general optimization.

Separately accepted root physical tests: v1538 detaching all selected
TSFS root DECtapes; v1539 detaching only one member while other
root members remain; v1540 detaching DSK0..DSK3 from a D6FS root.
All stop at the root-specific HALT PC. A v1534 physical failure
on the independent secondary DTC3 leaves the selected TSFS root,
DTC4 native DTFS and writable MEMFS operational.

## Remaining gates

The v1541 full-guest selected physical Type-236 PI2 failure and exact
BACKSTORE HALT-PC witness are required independently of the native
source test. Remaining alternate DRM root, unrelated DSK/DRM
secondary and shared DEVICE/VOLUME fault profiles are also pending.
P0-B and P1 feature tasks remain separate. Do not treat this partial
P0-A source integration as all-feature freeze.

DAIMOS: `design/kcore-backstore-service-failstop-v1541.md`.
Testkit: `tests/validation/daimos-backstore-failstop-v1541.md`.
