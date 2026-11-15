# KCORE v1588: recoverable post-DMA checked-release refusal PASS

Date: 2026-11-15. Two genuine 128K KCC PDP-6 physical cold boots
on the original four-member DR0..DR3 Type-236 D6FS root VOLUME,
BLOCKSET INTERLEAVE, mapped BACKSTORE, MEMFS/XIP and LOGSTORE.

The original PI6 timer wakes one sleeping, disk-backed process
at snapshot shutdown and the original slot-0 scheduler performs
an independent swap-in. The original MM allocates/pins an aligned
process extent, a **complete mapped physical BACKSTORE READ** receives
real Type-236 PI2 DR_DONE, and the original MM unpins. Exactly one
checked `backstore_free` entry then returns a test-only refusal
BEFORE modifying the backing bitmap. The original post-I/O
`proc_swap_in_free_fail` executes original MM rollback, retains
the authoritative DISK record and packed storage mode, clears
TRANSITION, and delivers its normal TERM event. Other original
swap-ins continue to publish successfully, and both snapshots commit.

| Original measured witness | Boot 1 | Boot 2 |
| --- | ---: | ---: |
| Original PI6 timer wake | 1 | 1 |
| Controlled post-DMA checked-release refusal | 1 | 1 |
| Completed original MM rollback and TERM | 1 / 1 | 1 / 1 |
| Independently started swap-ins | 4 | 4 |
| Actual completed physical swap READ requests | 4 | 4 |
| Actual Type-236 matching PI2 DR_DONE | 896 | 896 |
| Successful DISK-to-RESIDENT publishes | 3 | 3 |
| Original MM extents before/after | `024` / `023` | `024` / `023` |
| Original allocated VM base still present | 0 | 0 |
| Original process swap ownership | `01650` preserved | `01650` preserved |
| Original BACKSTORE count across refusal | `01734` preserved | `01713` preserved |
| Complete bitmap words compared/changed | 29 / 0 | 29 / 0 |
| Original DISK record, mode and BCACHE metadata | preserved | preserved |

Both normal original privileged HALTs at octal `022004`. Second
CPU/RAM cold boot restores the first persistent MEMFS v2/XIP data
and previous LOGSTORE record, then appends/reads a new one. Raw
four-member physical media contains the committed v2 header at
DR3 `017777`, real 3,648-word pool, 275 additional nonzero backing
sectors and intact D6FSR2 descriptors. Independent oracle rejects
**34 unsafe corruptions** of physical/PI/MM/bitmap/owner/process/
BCACHE/record/mode/commit witnesses. The v1587 pre-I/O rollback
and v1586 fatal swap-READ physical regressions also PASS.

The physical READ and PI2 are genuine; the checked release refusal
is synthetic at the original provider entry, not a naturally damaged
backing bitmap or a device fault. Under the original critical
BACKSTORE policy, actual owned-media I/O errors remain fatal.
BCACHE pointer/tags were unchanged but no allocated cache slab
or populated clean-cache DATA was observed in the rollback window.

**P1-C remains OPEN** for populated-BCACHE/valid-tag mixed provider
faults, simultaneous owner changes and repeated-load stability.
P1-D and P1-E remain OPEN. No installed kernel changes, fixed RAM,
ABI or disk-format differences. VERSION 0.5, production KCORE low
end `024364` octal and strict 32K/optimization HOLD unchanged.

Evidence: DAIMOS `design/kcore-p1c-postread-mm-rollback-v1588.md`,
testkit `tests/validation/daimos-p1c-postread-release-physical-v1588.md`
and retained physical data at
`$TMPDIR/daimos-p1c-postread-rollback-v1588.qWn8Gy`.
