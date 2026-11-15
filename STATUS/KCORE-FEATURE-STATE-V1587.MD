# KCORE v1587: exact pre-I/O MM rollback during physical snapshot overlap

Date: 2026-11-15. Accepted two full-profile 128K KCC PDP-6 cold boots
on four original Type-236 DR0..DR3 D6FS root VOLUME members, original
BLOCKSET INTERLEAVE mapped BACKSTORE, persistent MEMFS v2/XIP and
LOGSTORE. This is a **recoverable MM-pin refusal before physical swap
READ**, not a recoverable error from active/owned BACKSTORE media.

One original timed sleeping, nonresident child is woken during MEMFS
snapshot by the original PI6 timer service after controlled, test-only
deadline alignment. Original slot-0 `proc_swap_in` allocates real
aligned MM process VM. A disposable one-shot synthetic `MM_ERR_BUSY`
(-5) is returned at the subsequent pin point. Original `mm_free`,
process transition unlock, packed DISK storage mode preservation and
ordinary TERM event all execute without production changes. No
source bypass of the MM free or scheduler is accepted.

Two independent full physical boots PASSed. The original actual
linked-KCORE witnesses at the immediate transaction boundary were:

| Witness | Boot 1 | Boot 2 |
| --- | ---: | ---: |
| Real independent PI6-woken process slot | 6 | 6 |
| Test-only MM pin refusal / original MM rollback / TERM | 1/1/1 | 1/1/1 |
| New actual aligned MM extent base | `0220000` | `0220000` |
| Original MM extent count before/after | `024` / `023` | `024` / `023` |
| New base still present after MM free | 0 | 0 |
| Original process-owned swap blocks before/after | `01650` / `01650` | `01650` / `01650` |
| BACKSTORE blocks used before/after | `01734` / `01734` | `01713` / `01713` |
| All authoritative BACKSTORE bitmap words compared | 29 | 29 |
| Backing bitmap word mismatches | 0 | 0 |
| Original DISK record, process fields | unchanged | unchanged |
| Original process TRANSITION | correctly cleared | correctly cleared |
| Original authoritative storage mode | DISK | DISK |
| Original BCACHE slab ptr/workspace tag/private tag | unchanged | unchanged |

Original PI6/slot-0 concurrently completed three other swap-ins
with 672 matching physical Type-236 DR_DONE PI2 completions on each
boot. Both native privileged HALTs reached `022004` octal. Boot 2
physically restored the previous MEMFS XIP/data and LOGSTORE record,
then appended/read back a new record. Actual physical v2 commit
header DR3 sector `017777`, 3,648-word MEMFS pool, 275 additional
nonzero backing sectors, four intact on-media D6FSR2 layouts.

Independent source-linked/raw-sector oracle rejected 33 deliberately
false evidence variants, including 13 inherited physical/swap/PI2
mutations and 20 new exact MM/owner/bitmap/storage/BCACHE metadata
mutations. Earlier full physical v1585 healthy-overlap and v1586
critical swap-READ error oracles were independently reverified.

The synthetic failure occurs before physical I/O for that refused
process. It establishes accurate MM and backing/BCACHE **metadata
preservation**, not rollback of corrupted BCACHE data after I/O, nor
recovery from critical owned-BACKSTORE physical faults. P1-C remains
OPEN for actual post-I/O recoverable transactional rollback, multiple
simultaneous owner transitions, and intermittent v1585 timeout
investigation. P1-D and P1-E remain OPEN. VERSION 0.5, fixed
production KCORE low end `024364` octal, strict 32K/optimization HOLD,
no permanent kernel code/RAM/ABI/media format growth.

Engineering report: DAIMOS
`design/kcore-p1c-preio-mm-rollback-v1587.md`. Testkit
`tests/validation/daimos-p1c-pin-rollback-physical-v1587.{sh,md}`
and `daimos-p1c-pin-rollback-verify-v1587.py`. Original retained
physical two-boot evidence:
`$TMPDIR/daimos-p1c-mm-pin-rollback-v1587.b37MyN`.
