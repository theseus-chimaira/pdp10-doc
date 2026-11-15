# KCORE v1584: physical mapped snapshot commit/recovery PASS; PI overlap OPEN

Date: 2026-11-15. Accepted full-profile 128K KCC PDP-6 two-boot
four-member Type-236 D6FS root VOLUME. This is a **healthy physical
commit/cold-recovery acceptance**, NOT independent swap-in overlap.

Two cold boots used the same four original DRM media but newly
initialized CPU/RAM. Their real BLOCKSET mapped BACKSTORE supported
the original v2 MEMFS/XIP and LOGSTORE, while four 24K-word children
held positive actual process swap blocks. Both original privileged
HALTs executed at octal `021777`. The original hardware-simulator
Type-236 driver produced `051` octal (=41) real PI2 DR_DONE events
inside each snapshot and no observed logical storage error.

On both final HALTs, original `proc_swap_blocks_used=01310` octal and
`backstore_blocks_used=01374` octal. The actual raw four-member media
contains the valid MEMFS v2 snapshot commit header on DR3 block
`017777`, a 3,648-word pool, 271 other nonzero backing sectors and
all four byte-identical D6FSR2 layout descriptors. Boot 2 recovered
the original MEMFS data/XIP and the first boot's LOGSTORE record,
then appended and read back the next record. Ten deliberately
corrupted physical/map/post-HALT evidence variants were rejected.

Disposable test-only instrumentation separately records actual
independently scheduled slot-0 `proc_swap_in`, successful published
DISK-to-RESIDENT state and matching true Type-236 PI2 swap-READ
completion while snapshot shutdown is active. **All three were ZERO
in both boots** despite positive swapped owner counts and snapshot
PI2 activity. The strict verifier's `--require-overlap` mode correctly
rejects this physical data. No independence or concurrency claim is
justified by v1584. P1-C remains OPEN for that overlap and exact
recoverable MM/BCACHE/owner rollback; P1-D and P1-E remain OPEN.

No production code, fixed RAM, ABI or disk-format change. Version
0.5; fixed KCORE low end `024364` octal, optimization and 32K gate
HOLD. The original physical evidence is retained at
`$TMPDIR/daimos-p1c-swap-overlap-v1584.kR7MBC` and the repeatable
test and independent verifier are in daimos-testkit:
`tests/validation/daimos-p1c-snapshot-swap-physical-v1584.sh` and
`daimos-p1c-snapshot-swap-verify-v1584.py`. Engineering report:
DAIMOS `design/kcore-p1c-mapped-snapshot-success-v1584.md`.
