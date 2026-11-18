# v1663 PDP-6 bounded PURE bitmap compression frontend

Date: 2026-11-18. Parent DAIMOS v1662
`06f34cc23d4e5a16e6c735801930089bf0583e5f`.

## Purpose and ownership

The opt-in `proc_pure_bitmap_compress_v1663.c` supplies the missing
bounded preparation frontend to the v1662 prepared-only commit.
It does not change permanent KCORE or installed PURE MRES code.
The caller owns PROCESS TRANSITION on a noncurrent sleeping or
stopped process with valid immutable PURE backing, stable u-area,
no runnable APR mapping and storage mode2 (uncompressed RAW).

From the authoritative MM extent table it proves a single exact
unpinned PACKED_PROCESS suffix. It pins RAW and verifies exactly
one pin, counts nonzero 36-bit words, then calculates the original
v1451 bitmap length `ceil(raw_words/36) + nonzero`. A no-gain
candidate is rejected before any allocation. All physical writes
are bounded by the exact candidate length and verified MM owner
geometry; the source RAW span is always physically disjoint.

The original `mm_alloc_aligned_noreclaim` obtains an exact HIGH
candidate without swap, MM compaction, or recursive reclamation.
The unchanged 30-word v1451 PDP-6 codec writes the pinned bitmap
owner. After an exact verified candidate unpin, the existing v1662
C89 commit independently validates EVERY canonical RAW/encoded
word, current immutable backing and MM owner, and then deletes the
once-pinned RAW via the failure-atomic v1662 assembler leaf under
PI exclusion. In the same PI-disabled section the existing two-bit
storage map is written from RAW mode2 to compressed mode3.

No new per-process descriptor or scratch RAW-sized buffer exists;
the MM extent is the compressed length authority. The compressed
process remains TRANSITION-held; this function never advertises
READY or permits scheduler execution of unimplemented mode3.

## Cleanup under MM uncertainty

The only recoverable RAW image is not retired until v1662 has
verified the entire stream. No-gain or allocation failure restores
the original unpinned RAW owner and leaves mode2. A refused pin or
invalid bitmap after encoding triggers candidate-free FIRST, RAW
unpin LAST, with exact post-operation owner proofs. If target pin,
unpin, free or allocator outcome cannot be verified, return BUSY
and retain TRANSITION and surviving MM owners; no blind repair.
Even a falsely successful MM unpin/free is checked against the
source-extracted real MM table before ordinary error return.

Before cleanup of a refused v1662 commit, the frontend confirms
that the storage map still says mode2. Unexpected mode changes
are quarantined rather than destroying a potentially authoritative
compressed candidate. It refuses overlapping or forged candidate
descriptors before encoding; it never uses an unproved allocator
output as permission to overwrite RAW physical memory.

## Native PDP-6 evidence

The testkit links the actual original PDP-6 no-reclaim allocator,
fit, descriptor insertion, pin/unpin/free, the fixed mode2 ASM,
the original 30-word bitmap codec, v1662 commit/MM-retire, v1660
bounded decode and v1661 terminal release. It simulates only PI
observability and injected erroneous MM return values.

With 3,456 RAW words and 65 nonzero words, 96 presence masks plus
65 literals make exactly **161 encoded words**. The original MM
allocator reserves physical `050122` octal in the historical
883-word free gap. After mode3 publication an unrelated original
MM allocation reuses 02000-octal words starting at the released
RAW base `040000`; full bitmap decode and terminal release PASS.
An entirely zero-filled 3,456-word RAW suffix needs only 96 masks
at `050223` octal; native decode, reallocation and terminal PASS.
Dense data returns no-gain before allocating.

The 44 native v1663 cases consist of three success configurations
and 41 refusal/no-gain cases, covering original physical NOMEM
and descriptor exhaustion, injected MM failures, false successes,
invalid source/target pins, duplicate or unrelated owners, bad
mount/backing, corrupt encoder, mode changes, and forged overlap.
Executed unsafe source mutations disabling candidate geometry
verification or real candidate unpin are both rejected by the
native oracle. The entire original v1662/v1661/v1660/v1659
source-linked regression chain also PASSes.

Standalone unlinked C frontend: **776 TEXT words**, **0 DATA/BSS**.
Do not interpret this prototype size as installed MRES cost;
production must minimize duplicated owner scans and measure the
fully linked image to establish a net RAM benefit.

## Production blockers

Mode3 is still explicitly rejected by permanent storage-mode
assembly and unsupported in the live terminal, RUN, WAIT and
PURE restore paths. A bounded FULL restore must allocate an APR,
reconstruct immutable text, preflight compressed owner and stream,
decode the mutable suffix, and failure-atomically publish RESIDENT
without ever losing the only recoverable source. Then integrate
the terminal dispatcher, PI6 scheduler and all retry/finalizer
fault cases. Only after actual 48K/64K physical guests and
continuous peak RAM measurements can installed savings be claimed.
PURE remains OFF by default; strict physical 32K deferred.
