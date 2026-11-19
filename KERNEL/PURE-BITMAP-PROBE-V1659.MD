# PDP-6 PURE bitmap compression: bounded MM prototype v1659

Date: 2026-11-18. Parent: accepted v1658 owner-safety branch
`435557a54761d5910f435752c3a03c84afb1bef4`.
The compression installation state is UNCHANGED: PURE default OFF;
no compressed-process owner, no 32K or whole-system RAM claim.
The v1659 test uncovered a separate permanent allocator bug, fixed
below; fixed KCORE therefore differs from v1658 by two words.

## Test-only transaction

The new test-only v1659 C89 helper builds directly on the authoritative
PDP-6 MM descriptor table, TRANSITION, immutable-source admission and
existing two-bit storage mode. It requires a singly owned, unpinned
RAW PACKED_PROCESS suffix of exactly `VM words - PURE skip` words.
It pins RAW, counts every nonzero 36-bit word, calculates
`ceil(raw_words/36) + nonzero_words`, and rejects encodings not
smaller than RAW. It reserves the exact bitmap length separately
with `mm_alloc_aligned_noreclaim`, guaranteeing no self-reclaim.

The 30-word native v1451 encoder writes into the new pinned extent.
The probe independently verifies every RAW word, each presence bit,
unused bits in a final partial mask, and exact packed consumption.
The validation uses an inner group loop without division/modulo and
no second RAW-sized buffer. On success, the temporary encoded extent
is unpinned and freed BEFORE releasing the original source pin.
RAW content and storage mode remain unchanged; TRANSITION remains
owned by the caller. The reported number is a measured hypothetical
compressed allocation length, NOT published process residency.

In particular, target unpin/free and RAW unpin refusals MUST retain
TRANSITION instead of guessing that a failed MM operation succeeded.
A duplicate owner, overlapping allocator witness, malformed bitmap
or invalid backing is never allowed to become runnable.

## Why publication is not yet permitted

The permanent source-linked `proc_storage_expect_v30` and
`proc_storage_publish_v30` reject storage value 3, while mode2 means
an exact-length RAW suffix. The permanent PDP-6 terminal detach
also requires that RAW geometry. Replacing a 3,456-word RAW suffix
with a 161-word bitmap under mode2 would break terminal release
and fast restoration. A separate compressed ownership mode requires
new tested FULL decode, EXIT/WAIT/EXEC, PI6 exclusion, source release,
rollback and bounded MM metadata validation before production use.

## Executed native evidence and cost

Source-linked PDP-6 KCC transactions and original v1451-identical
assembly PASS twelve native cases: nominal 768-word fixture,
NOMEM, refused target pin/unpin/free, refused RAW unpin, poisoned
literal, no-gain density, illegal final bitmap padding, overlapping
MM destination, duplicate MM owner, and exact 3,456-word geometry.
The last case generates 96 bitmap masks and 65 literals, exactly
161 words. The test confirms RAW is preserved.

The original v1658 FULL/ownership regression remains PASS. An
independently compiled and executed test-only mutant removes the
stream verification; its corrupted-literal case is rejected by the
PDP-6 test oracle (`__test_exit=001604`). Neither compilation
failure nor a manually simulated decompressor is counted as proof.

Measured standalone DAS object sizes:

| Unlinked object | PDP-6 TEXT words | DATA/BSS |
| --- | ---: | ---: |
| Original v1451-identical codec | 30 | 0 / 0 |
| Experimental v1659 C89 probe | 586 | 0 / 0 |

That 586-word prototype is not optimized for final installed size,
and cannot be subtracted from the v1657 theoretical 19,770-word
gross six-owner saving to claim true net RAM recovery. The production
but the separate original MM allocator fix DOES alter fixed KCORE.

## Corrected permanent MM type admission

The real PDP-6 MM allocation test (not the controlled C fixture)
revealed the original `mm_alloc_aligned_noreclaim` rejected the
already-declared type 4, `MM_TYPE_PACKED_PROCESS`: the former
`TDNE type,[-4]` admitted only the first three type values. The
source now rejects undefined high bits with `TDNE [-10]` and rejects
types greater than 4 with `CAILE 4`. Type 0 and reserved 5..7 still
fail. This is a root-cause allocator correction, not an exception
path or a special bitmap allocator.

The source-linked real PDP-6 allocator leaves accept a 161-word
provisional PACKED_PROCESS extent within the precise 883-word gap
next to a pinned 3,456-word RAW extent, then unpin/free the target
and restore the original exact descriptor. An 884-word allocation
fails with NOMEM, descriptor exhaustion reports its distinct cause,
and invalid type values 0/5/7/8/-1 never insert an extent. Both
executed old type1..3 and overbroad type1..7 source mutants are
rejected. Existing production MM v1111/v1552 native suites PASS.

The actual newly linked 48K map has KCORE low end `024255` octal
versus v1658 `024253`, exactly two additional permanent words.
KINIT end `113643` octal versus v1658 `113641`, also +2.
The PURE bitmap codec and C probe remain entirely unlinked, and
these deltas are not a physically measured compression saving.

The fresh source-built 48K GCC selected TSFS ROOT / secondary DTC3
PDP-6 PURE guest successfully completes the original six-child
PIPE/WAIT/EXIT and HALT PC000000 with the corrected allocator.
Fixed KCORE stream SHA256:
`0a547e28dd9223e1a26420e4a75f8a6a5ca39836dd62b6a3c3d531ced57fc8f8`.
This is physical acceptance of the new MM guard in the unchanged
RAW-PACKED lifecycle, not actual installed compressed RAM.

DAIMOS implementation: `system/kernel/proc/proc_pure_bitmap_probe_v1659.c`
and `proc_pure_bitmap_codec_v1659.s`. Native testkit:
`tests/validation/pure-bitmap-probe-v1659/run-v1659.sh`.
Full development report:
`design/kcore-pure-bitmap-probe-v1659.md` in DAIMOS.

## Open acceptance work

Implement a recoverable mode3 representation, terminal cleanup,
bounded FULL recovery that cannot overwrite saved mutable bytes,
and exact source/destination ownership with all real MM/PI6 faults.
Then measure complete 48K/64K physical high-water and continuous
peak RAM against matched RAW and PURE-OFF builds. Keep strict 32K
acceptance and size-only optimization deferred until functional freeze.
