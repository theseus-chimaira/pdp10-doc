# v1664: bounded mode3 PURE bitmap FULL APR restore, PDP-6 native

Date: 2026-11-18. Parent DAIMOS v1663 commit
`640be3b6d37cba9082cf6f60f24e6b8d90e3ceef`.

## Compressed MM owner and original executable backing

The opt-in C89 `proc_pure_bitmap_restore_v1664.c` implements the
missing bounded FULL reconstruction of a compressed-mode3 PURE
process. The caller holds PROCESS TRANSITION for a stopped/sleeping
owner, or a runnable owner being restored by scheduler slot 0.
The existing two-bit storage map identifies mode3; the one exact
unpinned PACKED_PROCESS MM extent identifies both encoded base
and length. The original logical APR length and omitted PURE text
prefix follow from the saved executable backing record. No
additional per-process metadata or decoded-size scratch is needed.

Before touching the encoded payload, the restore verifies a live
immutable executable mount, saved u-area backing, absent APR,
original 02000-octal APR alignment, source MM type/owner/length,
18-bit physical range and the minimum mask capacity of
`ceil(raw_words/36)`. It pins the compressed owner, then uses the
original source-linked `mm_alloc_aligned_noreclaim` to reserve one
disjoint full-size APR PROCESS owner aligned for hardware
relocation. The target is also pinned while a four-argument
capacity-bounded loader reconstructs immutable executable text.
The v1660 bitmap decoder first checks every presence bitmap,
literal, trailing mask and exact stream consumption before writing
the authoritative decoded mutable suffix over the loaded APR.
The original unbounded 30-word decoder is never run on malformed
compressed data.

## Atomic transition to RESIDENT

New opt-in PDP-6 assembly `mm_pure_bitmap_finish_v1664.s` verifies
exactly two process-owned extents: once-pinned source of type
PACKED_PROCESS and once-pinned target of type PROCESS, each with
the expected base, length, alignment, 18-bit physical bounds and
disjointness. It refuses extra process owners, incorrect pins,
reserved/invalid types and corrupted MM capacity. MODULE and
KERNEL_DYNAMIC use distinct owner namespaces, even when numeric
IDs match the process slot.

After all checks, while PI and PROCESS TRANSITION are held, the
MM leaf deletes the compressed source descriptor and consumes
the target APR pin using only nonfallible descriptor stores. The
C caller then publishes the same original VM-state APR base,
clears the saved PURE backing, changes mode3 to RESIDENT (mode0)
and clears TRANSITION before restoring PI. No filesystem or MM
allocation occurs in this final PI-disabled window; no fallible
operation follows retirement of the last compressed image.

On ordinary refusal the still-authoritative compressed source
remains pinned while the target is unpinned, freed, and verified;
source unpin is last. Every cleanup step checks exact MM ownership
after the operation. Unexpected source or target pins, false MM
success, changed TRANSITION/mode/backing/APR or failure to free
the destination returns BUSY without guessing an owner state.
No process may resume from an uncertain or corrupt restoration.

## Native original-MM verification

The new testkit suite links actual original PDP-6 MM allocation,
fit/insertion, pin/unpin/free/delete, permanent mode2 expectation,
original 30-word bitmap codec, v1660 bounded decoder, and the
v1661/v1662/v1663 compressed ownership path. The loader is an
explicitly bounded **test callback**, not yet an integrated VFS
executable read. A second physical MM arena holds exactly one
4,096-word aligned APR at octal `0100000` while the prior arena
holds the 161-word compressed bitmap. Every restored APR word is
compared to its expected immutable prefix or decoded 3,456-word
RAW suffix. All-zero suffixes (96 masks), slot-0 SRUN and an
unrelated MODULE sharing the process ID also pass.

The direct MM oracle additionally checks source-first and
target-first MM descriptor order, the full 136-entry MM extent
table, extra owner refusal, wrong type/pins/length/geometry and
physical boundary checks. Overall 62 native v1664 scenarios
(8 positive, 54 refusal/corruption) pass. Both separately built
unsafe mutations--skipping the bitmap decoder and ignoring a
second PACKED_PROCESS MM owner--are executed and detected by the
native PDP-6 test. Full original v1663/v1662/v1661/v1660/v1659
native regressions pass as well.

Standalone unlinked sizes: **969 TEXT words** for C89 FULL
restore and **133 TEXT words** for atomic PDP-6 MM finish,
each zero DATA/BSS. These are staging measurements, not new
permanent KCORE code or measured optimized MRES footprint.
Source-owner checks and error paths should be shared/folded before
acceptance on memory-limited physical PDP-6 configurations.

## Production integration still required

The existing permanent storage runtime continues to refuse
mode3; fast PREFIX restore still accepts only uncompressed RAW,
and scheduler wake, process exit, WAIT and `proc_swap_detach`
are not yet wired to the new compressed owner path. This stage
proves bounded FULL reconstruction and atomic MM publication in
native tests, NOT physical full-guest lifecycle correctness or
end-to-end executable VFS read. Next integrate the compressed
terminal dispatcher, scheduler PI6/failure matrix and full
source-built 48K/64K guest acceptance with continuous net and
peak physical RAM accounting. PURE default OFF, strict 32K
physical acceptance deferred.
