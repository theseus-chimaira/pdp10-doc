# Swapless TSFS RAM compression: zero-bitmap experiment v1432

**Later v1451 optimization:** the bitmap pair now assembles to
30 rather than 55 words and has no bounds/stream checks, since
the only permissible producer and consumer are cooperating MM
operations over trusted, kernel-owned process memory. AC10..AC15
are preserved without a frame; the normal MRES image is 30
resident words. This page retains the original v1432 experiment;
see D6CODEC-BITMAP-TRUSTED-OPTIMIZATION-V1451.MD for final
API, tests, speed and folding results.

Date: 2026-11-10. **Completed native PDP-6 component study, no
production integration.** The established swapless PURE-first
direction in SWAPLESS-MEMORY.MD remains unchanged. Full scripts
and source are in daimos-testkit/tests/validation/
d6codec-compressor-v1432/. This document is an analysis of whether
an additional in-core compressor is worth its permanent RAM cost.

## Why it was reconsidered

DAIMOS's target minimal TSFS-root, swapless system lacks process
disk swap. Therefore compacting an inactive process means keeping
its mutable image in RAM, never writing to TSFS. Releasing
reconstructible immutable PURE text is an independent first step.
For the three actual DSH captures, a future PURE prefix release
would save 15,232 words, before any compression. DSH is not
currently tagged PURE and this release is not yet active.

The prior exhaustive LZ encoders cost 0.65-3.6 million native
instructions per sample, too slow for the desired 0.1-0.5 s
budget at nominal 0.25 MIPS. Their decoders are fast, but this
does not make their *encoders* acceptable. The established
RLE v1376 encoder cost ~115k-202k native instructions on these
snapshots. A simpler format can exploit the fact that most
retained user words are exactly zero.

## Bitmap representation and measured code

Every 36 decoded words are described by one 36-bit presence
bitmap, followed by precisely the nonzero literal words. Bit 35
describes the first word; the number of decoded words is supplied
by the MM descriptor. The last group may be incomplete.
There is no sliding window, match dictionary, heap allocation,
static BSS or userland I/O. The assembly prototype uses AC
registers and separate source/destination extents.

The native PDP-6 DAS object is exactly **55 TEXT words**
for both encoder and decoder together, with **0 DATA and
0 BSS**. This includes neither process-storage ownership
logic nor production ABI save/restore. It is not yet hardened
for malformed trailing unused bits in the final bitmap.

## Actual DSH payload and independent native step results

| State | Raw retained suffix | Nonzero words | Bitmap payload | Additional saved |
|---|---:|---:|---:|---:|
| Idle prompt | 3,200 | 225 | 314 | 2,886 |
| Six SET commands | 3,200 | 435 | 524 | 2,676 |
| Function defined/executed | 5,248 | 531 | 677 | 4,571 |

Each sample is an actual read-only PDP-6 SIMH user-memory
capture from v1426. The input SHA256 hashes are included in
testkit results-v1432.tsv; raw guest memory stays local.
All three exact native encode/decode roundtrips and five
additional edge cases PASS.

## Why the bitmap must NOT replace the TSFS file codec

The real 29-image initialized-DXR corpus, plus the actual DSH
executable, was compared with the unchanged production D6LZ36
host compressor and existing v1420/v1424 host reference
encoders. Sizes below count compressed 36-bit codec words,
not media descriptors, boot headers, allocation maps or
DECTape seeking. The independently repeated 256-word restart
experiment is also recorded in the versioned corpus TSV.

| Actual initialized image | Original | Bitmap | D6LZ36 | Grouped | Block |
|---|---:|---:|---:|---:|---:|
| KCORE | 9,418 | 9,658 | 9,189 | 8,788 | 9,125 |
| DSH | 15,331 | 15,724 | 13,400 | 12,385 | 13,617 |
| MonitorFS MRES | 1,592 | 1,637 | 1,573 | 1,517 | 1,560 |
| Tape MRES | 390 | 401 | 392 | 379 | 386 |
| KINIT (atypically zero-filled) | 29,097 | 20,174 | 18,465 | 17,449 | 18,456 |

For **all 29** available built DXRs, the exact totals were
48,487 original words, 40,088 bitmap, 37,310 D6LZ36,
35,540 grouped and 37,190 block. However, KINIT alone
contains 9,732 zero words and dominates the bitmap's apparent
benefit. **Excluding KINIT**, the remaining 28 images contain
19,390 words, and bitmap **expands** them to 19,914
(+2.70%); D6LZ36 reduces them to 18,845, grouped
to 18,091 and block to 18,734. This is a direct measurement
on compiled source images, not synthetic mutable process RAM.

The bitmap can only exploit zero words. Instruction-dense
executables contain almost none and often contain repeated
*nonzero* sequences that LZ can compress. TSFS's disk/tape
block count and restart framing also make payload reduction
material to transfer and seek costs. The 55-word bitmap codec
is therefore **only an optional mutable-process RAM codec**,
not a proposal to replace the canonical file or executable
decoder with the bitmap format. KINIT is a special sparse
exception, not a representative regular-file result.

| State | Encode instructions | Decode instructions | Encode s at 250k IPS | Decode s | Roundtrip s |
|---|---:|---:|---:|---:|---:|
| Idle | 24,658 | 30,565 | 0.0986 | 0.1223 | 0.2209 |
| Six SET | 25,918 | 31,615 | 0.1037 | 0.1265 | 0.2302 |
| Function | 41,400 | 50,926 | 0.1656 | 0.2037 | 0.3693 |

These are exact **SIMH instruction counts** of each native
encode-only/decode-only path, including a minimal test call but
excluding word-by-word validation loops. Timing is just
instruction_count / 250,000; it is NOT measured physical PDP-6
time, and code/allocator/PI transaction overhead remains
unmeasured. Unlike the BLT decoder experiments, bitmap v1432
uses no BLT, so the timing inference does not hide bulk
transfers charged as one instruction.

The codec meets the given 0.1-0.5 s per-direction **nominal**
budget on all three measured DSH samples. The earlier v1376
RLE encoder took about 115k/129k/202k instructions; the
grouped/block LZ engines about 653k-1.96M, and exhaustive
D6LZ36 about 1.93M-3.59M, depending on the snapshot. The
55-word bitmap pair is significantly smaller than the
449-word v1376 C code.

## Real-memory impact

For a 32,768-word PDP-6, compressing an idle DSH suffix
from 3,200 to 314 would free **2,886 words**, about 8.8%
of all physical core, *after* the more important independent
15,232-word PURE reclamation. The function-heavy case could
free **4,571 words**, about 14.0% of 32K. Net actual free
RAM must subtract the 55 words of code, ownership metadata,
any temporary extent and fragmentation effects.

With only a retained raw mutable suffix, an idle shell costs
3,200 words of managed process storage; with the bitmap it
would cost 314 words plus MM metadata. That could change
whether multiple dormant processes can coexist on a small
swapless system. But if a runnable process cannot regain a
full contiguous APR-aligned extent, compression does not
solve the restore headroom problem.

DECtape hardware has slow mechanical positioning: TU56
specifications include roughly 150 ms start, 200 ms
turnaround and up to ~30 seconds travel across a whole tape
(https://www.pdp8online.com/tu56/tu56.shtml). This is
independent of RAM compression: **TSFS holds the immutable
executable that PURE restore rereads; the packed mutable
suffix never goes to tape**. In particular, a sleeping
interactive shell may wake on each input character.
Automatically dropping PURE or packing on every sleep would
be disastrous for interactive latency even if encode takes
only 0.1 s.

## Proposed implementation order

First complete PURE-only drop/restore and the unified bounded
EXEC/PURE loader, then profile 32K/48K live process residency,
APR fragmentation, slow tape restore and memory pressure.
Do not automatically pack every SLEEP. Only a cold,
quiescent victim under *proven memory pressure* might be
eligible, with a raw representation fallback.

If those tests show that ~2.7k-4.6k extra recovered words
frequently avoid OOM or allow useful concurrency, a second
tiny **bitmap decoder plus encoder** is defensible alongside
the canonical immutable-file D6LZ36 decoder. The combined
bitmap pair is just 55 extra standalone TEXT words. If the
memory benefit is too rare or the restore hazard too high,
leave the codec unlinked. Do not put exhaustive LZ encoding
into KCORE simply to reuse the executable decoder.

Before a production decision: complete malformed-bitmap
validation, target ABI callee saves, exact MM compressed
ownership/encoded count, admission/rollback, reserve
restore headroom, measure linked/folded KCORE code growth
and real 32K/48K throughput. No activation is claimed.
