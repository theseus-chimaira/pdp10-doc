# KCORE v1589: physical D6FS populated BCACHE reclaim and no stale identity

Date: 2026-11-15. Two successful genuine 128K KCC PDP-6 physical
cold boots on original four-member Type-236 DR0..DR3 D6FS root
VOLUME with BLOCKSET mapped BACKSTORE, MEMFS/XIP and LOGSTORE.

The opt-in userland workload uses ordinary VFS OPEN/READ against
physical `/CONFIG/XIP1469.TEST`, populating the actual MM-backed
private D6FS BCACHE slab. The private cache is NOT made up by test
code. Its real source/block key is `100100001010` octal, address
`0307577`, and all 128 cache data words are captured. During an
independently scheduled PI6 slot-0 swap-in, the original
`bcache_reclaim` matches that exact owner and tag. Its full 128-word
data still matches immediately before invalidation, original MM
successfully frees the private cache slab, and a later `bcache_fetch`
of the old key correctly MISSES. No stale identity is published.

| Real original cache/rollback witness | Boot 1 | Boot 2 |
| --- | ---: | ---: |
| Valid D6FS private cache at snapshot entry | 1 | 1 |
| Nonzero words of 128 cached source words | 28 | 28 |
| Exact cache reclaim matched original slab/key | 1 | 1 |
| Independent swap-ins already started at reclaim | 1 | 1 |
| Full 128-word differences before original reclaim | 0 | 0 |
| Original MM successfully freed cache slab | 1 | 1 |
| Cache MM free refused or left in flight | 0 | 0 |
| Stale BCACHE hit after reclaim | 0 | 0 |
| Physical BACKSTORE swap READs | 4 | 4 |
| Original matching Type-236 swap READ PI2 completions | 896 | 896 |
| Successful RESIDENT publishes | 3 | 3 |
| Changed original BACKSTORE bitmap words on rollback | 0/29 | 0/29 |
| Process-owned backing blocks across rollback | `01650` unchanged | `01650` unchanged |

Both native privileged HALTs at octal `022004`; boot 2 physically
restores previous MEMFS XIP/data and LOGSTORE, then appends/reads
its next record. Valid v2 snapshot commit header at DR3 `017777`,
3,648-word MEMFS pool, 282 other nonzero backing sectors and four
unchanged D6FSR2 member descriptors. The independent two-boot oracle
rejects 48 bad physical/PI/MM/owner/bitmap/BCACHE/data/reclaim
witnesses: 34 inherited from v1588, 14 new cache-specific controls.
The inherited v1588 physical two-boot oracle also passes.

**P1-C remains OPEN.** This experiment proves safe *legitimate
reclamation* of a populated valid clean cache, not retention of a
cache that has not been reclaimed. The independent v1588 post-DMA
BACKSTORE release refusal is test-induced; actual owned-media errors
remain fatal. Still open: valid populated cache retention under an
unrelated recoverable provider failure without MM reclaim, mixed
DEVICE/VOLUME provider cases, simultaneous owner transitions, and
repeated-load timing stability including the v1585 timeout.

One separate legacy testkit fixture, v1490 BCACHE/DTC native, now
fails its old minimal source link because modern DTFS write code
requires `dtfs_intent_mount_gate_v1561`. This does not represent a
production failure and must not be bypassed with a synthetic gate.
VERSION 0.5, production KCORE low end `024364` octal, strict 32K/
optimization HOLD, P1-D/E status and production code/RAM/ABI/media
formats are unchanged.

Source-backed report: DAIMOS
`design/kcore-p1c-physical-d6fs-bcache-reclaim-v1589.md` and testkit
`tests/validation/daimos-p1c-populated-cache-physical-v1589.md`.
Retained full physical evidence:
`$TMPDIR/daimos-p1c-populated-cache-v1589.LkkpE8`.
