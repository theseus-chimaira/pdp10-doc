# Native PDP-6 compression and decompression tradeoffs v1429

Date: 2026-11-10. Status: measured native encoder and decoder
tradeoffs, plus native BLT optimization of the existing D6LZ36
shared decoder. No process compression has been activated.
This extends the preceding D6CODEC DSH live capture v1426.

## Experimental method

Three actual resident DSH user-process snapshots from a running
PDP-6 DAIMOS guest were obtained read-only in v1426. The guest
was idle, had executed six SET commands, or had defined and
executed a function. Their prospective PURE-retained physical
suffixes were 3200, 3200 and 5248 36-bit words respectively.
The current DSH binary still has no PURE flag; process memory
was not actually released.

Four bounded, direct-memory C encoders were compiled for native
PDP-6 and exercised with the actual captured words:
the unchanged DAIMOS RAM RLE v1376 code, a small C89 D6LZ36
encoder port using the production greedy 128-word search,
and C89 grouped-v1420 and block-v1424 encoders reproducing
the established independent host reference streams exactly.
All seven source sets (three DSH and four test controls) pass
for all four native encoders; no static encoder DATA/BSS.

Only the RAM RLE encoder is existing production-intended code.
The other three are isolated testkit versions, not production
kernel implementations. The D6LZ36 port is distinct from the
full filesystem-based D6LZ userland utility.

All native timing counts are PDP-6 SIMH retired-instruction
STEP counts. Each BLT is counted once no matter how many words
it copies, and exact hardware wall time was not measured.
Native encoder tests compare compressed words exactly with
independent reference implementations.

## Raw and encoded memory sizes

| Real DSH state | Raw | D6LZ36 | Grouped | Block | RAM RLE |
|---|---:|---:|---:|---:|---:|
| Idle | 3200 | 277 | 246 | 280 | 263 |
| Six variables | 3200 | 508 | 439 | 532 | 537 |
| Function | 5248 | 614 | 524 | 656 | 677 |

Grouped streams are consistently smallest, but the memory
savings relative to RLE are only 17, 98 and 153 words for
these three states. Per-process metadata, code residency,
allocations and compression transaction buffers are excluded.

## Native PDP-6 encoder and decoder code size

| Component | TEXT words |
|---|---:|
| In-memory D6LZ36 C89 encoder prototype | 174 |
| In-memory block C89 encoder prototype | 227 |
| In-memory grouped C89 encoder prototype | 264 |
| Existing RAM RLE v1376 C measure + encode + decode | 448 |
| BLT-optimized D6LZ36 assembly decoder and adapter | 96 |
| Existing compact block v1424 decoder and adapter | 164 |
| Grouped v1420 decoder and adapter | 364 |

Indicative combined test implementation totals:
D6LZ36 270 words, block 391, grouped 628, RLE 448.
The combined totals do not include a uniform process-packing
ABI or an exact-size prepass for the three experimental encoders.
In addition, decoder adapters may contain VFS-specific overhead
that a direct process-memory codec does not need. These are
standalone component sizes, NOT measured linked KCORE deltas.

The measured test-call stack-canary high-water figures were
approximately 17 words for D6LZ36 and block, 22 for RLE,
66 for grouped, including the common test call overhead.

## Native compression and decompression instructions

| Encoder | Idle | Six variables | Function |
|---|---:|---:|---:|
| RAM RLE encode call | 116k | 129k | 202k |
| RAM RLE additional exact-size prepass | 112k | 122k | 192k |
| Grouped | 611k | 1301k | 1586k |
| Block | 686k | 1480k | 1836k |
| D6LZ36 | 1961k | 2930k | 3641k |

| Decoder | Idle | Six variables | Function |
|---|---:|---:|---:|
| RAM RLE v1376 KCC C decoder | 38.7k | 42.7k | 67.5k |
| Grouped assembly | 9.1k | 18.0k | 23.1k |
| Block v1424 assembly | 1.7k | 3.4k | 5.1k |
| D6LZ36 BLT assembly | 3.2k | 5.8k | 7.3k |

These are direct codec calls in native PDP-6 tests. The RAM
RLE code has a separate exact-size pass; other encoder fixtures
assume a caller-provided output buffer sized by a host reference,
so their eventual exact-size allocation cost is not accounted
for here.

## Full native cycle costs

| Pair | Idle | Six variables | Function |
|---|---:|---:|---:|
| RAM RLE encode + decode | 154k | 172k | 269k |
| RAM RLE encode + measure + decode | 265k | 293k | 460k |
| Grouped encoder + decoder | 620k | 1319k | 1609k |
| Block encoder + decoder | 687k | 1484k | 1841k |
| D6LZ36 encoder + BLT decoder | 1964k | 2936k | 3648k |

Only the RLE pair includes the required size-measurement
prepass in its second row. Other codecs may also need a
prepass, or temporary over-allocation, when used for atomic
process-memory compression. This comparison therefore
understates the relative cost of those alternatives.

## D6LZ36 BLT low-core optimization

Separately, the production shared fixed-address D6LZ36 core
was changed from a per-word match loop to a 4-instruction
forward BLT. The native core shrank from 40 to 39 words,
the combined core + VFS adapter from 97 to 96, and an
isolated linked KCORE DXR initialized image from 9601 to
9600 words without changing BSS or relocations.

On the three real DSH snapshots, the original D6LZ36
decoder required 17928, 19377, 30637 steps, and the
BLT decoder required 3238, 5791, 7291. The old and
new bitstream formats remain identical for this separate
hot-path optimization. Twenty-three native stream/refill/
overlap/malformed tests pass, all five Stage-1 assemblies
shrink by one word, and a rebuilt DAIMOS KCORE plus DSK
Stage-1 successfully boots through ROOT DSH and ECHO.
Existing DTC Stage-1 test size-cap mismatch predates this
change: DTC was 140 words before, 139 afterward against
an older 106-word test cap.

## Decision and next optimization gate

Select RAM RLE v1376 as the leading **baseline** for
mutable-process state. Both compression and decompression
must be efficient: RLE is the lowest measured total CPU
cost, even when charging its extra exact-size prepass.
Its compression is within 17 to 153 encoded words of the
best grouped stream for real DSH samples, but its complete
448-word KCC implementation is larger than some test-only
encoder/decoder combinations.

Next optimize the RAM RLE decoder into compact PDP-6
assembly using BLT for zero, repeated and literal blocks,
then measure entire encoder/decoder/measurement text,
stack, scratch and linked/folded KCORE RAM. Consider
replacing the C encoder with compact assembler only when
total code and cycles improve. Do not add permanent
duplicate process compression implementations.

A possible future format improvement is RLE with a tiny
8- or 16-entry recent-word match cache. This is a
hypothesis, NOT an implemented or measured improvement;
its dictionary RAM and CPU cost need to justify any
compression gain. Avoid exhaustive 128-distance greedy
LZ search when process packing CPU matters.

Source and reproducible tests:
daimos-testkit/tests/validation/d6codec-encoder-v1429,
d6lz36-blt-v1428 and d6lz36-dsh-live-v1427.
