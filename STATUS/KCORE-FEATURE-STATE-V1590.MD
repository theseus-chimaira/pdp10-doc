# KCORE P1-C v1590: live clean D6FS cache under LOGSTORE physical retry

Date: 2026-11-15. Two complete real 128K KCC PDP-6 cold boots on
the original four-member DR0..DR3 Type-236 root VOLUME with
BLOCKSET INTERLEAVE, mapped BACKSTORE, MEMFS v2/XIP and LOGSTORE.
This stage accepts a clean MM-owned D6FS private BCACHE entry
surviving a recoverable fault/retry in a separate LOGSTORE provider,
**without being reclaimed**. v1589 separately accepted safe cache
reclamation during genuine swap memory pressure.

The ordinary opt-in XIP guest opens and reads actual physical D6FS
root file `/CONFIG/XIP1469.TEST` through the original VFS/D6FS
reader, populating a genuine private clean BCACHE slab with valid
source tag. The v1577 test-only LOGSTORE hook then uses the original
mapped Type-236 write to persist a physically uncommitted record
with commit trailer zero. It reports a controlled completion error
**after** that physical write; original LOGSTORE status/cursor must
remain unchanged. The user checks the torn block, retries the same
original ring slot, and reads back its committed replacement.

The v1590 disposable observer captures the actual private slab,
36-bit D6FS key and 128 36-bit words, then verifies the exact same
tag, owner and 128 words after the torn physical write and after
successful committed retry. The original `bcache_fetch` must return
HIT both times, with all 128 copied words correct. No fake D6FS
cache item or controller completion is introduced by this observer.

| Real linked evidence | Boot 1 | Boot 2 |
| --- | ---: | ---: |
| Private BCACHE slab and key | `0377577`, `100100001010` | `0377577`, `100100001010` |
| Nonzero words of 128 real cached D6FS words | 28 | 28 |
| Physical uncommitted LOGSTORE write observed | 1 | 1 |
| Original failed-append and retry inspections | 1 / 1 | 1 / 1 |
| Changed slab/tag/full cached words after failure | 0 | 0 |
| Changed slab/tag/full cached words after retry | 0 | 0 |
| Original cache fetch after failure/retry | HIT / HIT | HIT / HIT |
| Changed full 128-word fetched blocks | 0 / 0 | 0 / 0 |
| Later independent mapped physical swap READs | 4 | 4 |
| Genuine matching Type-236 swap READ PI2 | 896 | 896 |
| Later successful RESIDENT publications | 3 | 3 |
| Changed original backing bitmap words after MM rollback | 0/29 | 0/29 |
| Retained original DISK process owner | `01650` | `01650` |

Both full privileged HALTs were normal. The second cold boot
restored first-boot persistent MEMFS/XIP and LOGSTORE, then performed
its own torn/retried append and snapshot. Raw physical evidence
contains the committed MEMFS v2 header at DR3 `017777`. The source-
linked independent two-boot oracle rejects **47 deliberately false
witnesses**: 13 original physical/PI/scheduler, 21 original
MM/BACKSTORE/rollback and 13 live-cache/LOGSTORE cases. The older
native v1490 BCACHE/DTC harness is separately repaired and passes
with the authentic production DTFS v1561 write-intent gate and
its original dependencies, not a fabricated authorization stub.

The torn record's physical DMA is genuine; its returned completion
failure is deliberately injected and is NOT an actual Type-236
controller error. Owned-BACKSTORE physical I/O errors remain fatal
under the existing critical-media policy. At a **later independent
swap-in**, the kernel may legitimately reclaim the cache: v1590
does not require retention across that memory-pressure event.

**P1-C OPEN** for truly simultaneous independent owner changes and
mixed-provider faults, later-position failure timing, and loaded
repeatability (including the isolated v1585 timeout). P1-D/E remain
OPEN. No production kernel source/RAM/ABI/media format changes;
VERSION 0.5, fixed KCORE low end `024364` octal and strict 32K/
optimization HOLD remain unchanged.

Sources: DAIMOS `design/kcore-p1c-live-d6fs-cache-logstore-retry-v1590.md`,
testkit `tests/validation/daimos-p1c-live-cache-logfault-physical-v1590.md`.
Physical evidence preserved at
`$TMPDIR/daimos-p1c-live-cache-logfault-v1590.jhggKy`.
