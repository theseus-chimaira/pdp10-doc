# v1662: prepared PURE compressed owner, atomic RAW MM retirement

Date: 2026-11-18. Parent DAIMOS v1661
`2fe9d857b1152a6d1eeb35e91c57116fc2ae5627`.

## Why the MM leaf is necessary

Calling `mm_unpin(raw)` followed by `mm_free(raw)` cannot provide a
failure-atomic compressed-process commit. The free may fail after
the RAW owner's final pin is removed. There is no safe rationale
for guessing the pin state and blindly repairing a rejected MM
transaction. Instead the optional PDP-6 `mm_pure_bitmap_retire_v1662`
checks the exact once-pinned RAW PACKED_PROCESS MM descriptor,
the separate unpinned shorter encoded descriptor, matching process
owner slot, uniqueness, all geometry and physical capacity before
performing a single nonfallible descriptor-delete transaction.

Two process MM descriptors are allowed only under TRANSITION during
compressed preparation. A duplicate, pinned target, incorrect RAW
pin, extra PROCESS owner or illegal type refuses without changing
either descriptor. Other MODULE/KERNEL_DYNAMIC owner namespaces
do not prevent safe process release. The same two-word MM extent
format and the already-installed two-bit per-slot storage map are
reused; there is no new process registry or permanent metadata.

## Prepared-only commit

`proc_pure_bitmap_commit_v1662.c` performs full RAW/bitmap comparison
before any MM release, accepting only the canonical original v1451
format: masks, exact stream consumption, nonzero literals and
zero-valued absent RAW words. It validates the executable mount
identity, stopped/sleeping TRANSITION owner, saved u-area backing,
absent APR, valid PURE prefix and mode2. Under PI exclusion it
rechecks mode2, invokes the new MM leaf, and only upon successful
RAW deletion writes mode3 into the existing two-bit slot. No
fallible operation occurs after RAW deletion. The process remains
TRANSITION-held; source content is not overwritten or restored.

This API intentionally consumes an already prepared candidate.
It is NOT a bounded compressor frontend and does not allocate,
encode, unpin, roll back, or unlock that candidate itself.

## Native original-MM evidence

Source-linked KCC/PDP-6 tests use original source-extracted MM
allocator, `mm_find_fit`, `mm_extent_insert`, pin/unpin, free and
delete, plus the original mode2 expectation ASM. The actual v1657
3,456 RAW words/65 nonzero words are encoded as 96 masks + 65
literals = **161 encoded words**. The original real MM allocator
fits the candidate in the historical 883-word gap at octal
`050122`. The v1662 MM leaf removes the RAW owner and commits mode3
under PI exclusion. A subsequent independent original-MM request
then successfully allocates 02000 octal words at physical octal
`040000`, precisely inside the newly reclaimed RAW region.
After that independent owner is released, the original bounded
v1660 decoder reconstructs all RAW words and the v1661 terminal
leaf releases the compressed owner.

The native matrix passes 24 C transaction refusal cases plus
21 direct MM leaf refusals with unchanged MM/pin/mode. It covers
RAW-first or packed-first table order, full 136-descriptor capacity,
and a MODULE extent sharing the same numeric owner ID. Two
independently compiled/assembled dangerous mutants (skip canonical
data verification; ignore a third packed MM owner) are executed
and rejected. Full v1661/v1660/v1659 native baseline PASS.

Standalone unlinked code sizes: C89 prepared commit **606 TEXT**,
new MM ASM **125 TEXT**, zero DATA and BSS for each. These are
prototype sizes; not KCORE growth or accepted live MRES footprint.

## Remaining prerequisites for live compression

The bounded allocation/encoding/rollback frontend is not yet
installed. Fixed permanent mode checks still refuse mode3, and
`proc_swap_detach` still dispatches RAW-only terminal teardown.
The actual FULL process restore must rebuild immutable PURE text,
check compressed source ownership and stream, perform bounded
decode to an APR image, and publish a runnable process with no
fallible post-commit MM operation. Scheduler PI6 and terminal
failure matrix, physical 48K/64K guest and continuous peak memory
measurement remain open. The current native real-MM-reuse witness
is NOT proof of installed net RAM savings. PURE default OFF,
strict 32K physical acceptance deferred.
