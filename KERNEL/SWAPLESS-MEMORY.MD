# Swapless process-memory reclamation

Status: accepted KCORE direction, 2026-11-09 v1375. **Staged PURE-only
and zero-copy transactions pass through v1382; no production scheduler or full
swapless guest acceptance.** This document supersedes the earlier proposal
for executable-opt-in *compressed disk swap* (old decision 5/P0-D). Raw
disk/drum process swapping is unchanged. Compression of disk-swap records is
not part of the release plan.
The old P0-D *feature-list* label must not be confused with DAIMOS's
separate v1298 **P0-D OOM fault/regression matrix**, which stays accepted.

## v1391/v1392: APR-bounded FULL DXR restoration and KCC correctness

**2026-11-10. Staged host/native test acceptance, NOT a production
KCORE or disk-swap change.**

The previously staged FULL DXR callback accepted a backing record,
destination APR base and stable process u-area, but did not receive
the extent's *allocated length*. Validating only the executable
text count was insufficient: a DXR file with the same PURE text
but an oversized initialized image could write past the allocated
APR. This also affected compressed FULL decode.

`proc_pure_ram_restore_v1391.[ch]` now supplies a four-word C ABI
callback argument list: saved backing identity, pinned physical
destination APR base, stable u-area and **trusted allocated APR
word count**. The shared assembler
`exec_image_full_bounded_v1391` first obtains the immutable source
vnode, VFS stat and DXR2/TX2 metadata using the same parser as EXEC
and PURE_PREFIX. Before invoking any payload reader or decompressor,
FULL mode rejects zero/unaligned APR capacity, unaligned APR base,
physical address overflow, or an initialized image where
`image_words + 020 > allocated_APR_words`. An exactly fitting
initialized image is valid. The former unbounded staged FULL entry
is removed, not retained as a compatibility backdoor. The current
production `proc_swap_reload_pure` remains unchanged.

Native KCC/DAS/PDP-6 tests in `daimos-testkit` verify successful
plain/compressed FULL restoration, exactly filled APR, mount 18
and canonical vnode identity, short read, invalid metadata,
compressed decoder failure and rollback. Crucially, both plain
and compressed *oversized* cases observe **zero payload-reader and
decoder invocations**. Independent negative cases cover zero,
misaligned and out-of-range allocations and physical base
misalignment; sentinel words immediately beyond the target APR
remain unchanged. The only valid mutable suffix is copied from
its original owner-pinned raw RAM after the FULL reload succeeds.
These are native component tests with controlled VFS/MM, not
concurrent scheduler/PI or 32K/48K guest acceptance.

### Optimizer integrity and size

Testing uncovered a separate real KCC code-generation defect:
under a larger diagnostic layout, optimized KCC emitted
`MOVEM 7,swap_slots(1)` for the constant slot
`swap_slots[1]`, leaving a dynamic AC1 index without a valid
reaching definition. `kcc/ccopt.c` now embeds a constant in the
symbolic address only after a straight-line backward register
definition scan proves it, stopping at skips, branches, calls and
register modifications. This is a compiler fix, not a DAIMOS
array-access workaround. All 21 focused native/host PURE and
EXEC regressions pass on the corrected installed KCC; its fresh
optimized release build matches the tested binary. The cleaned
v1392 test also passes on the old compiler, so a smaller
independent old-failing repro and full compiler matrix remain open.
See `TOOLCHAIN/KCC-SYMBOLIC-FOLD-20261110-V1392.MD`.

| Isolated PDP-6 source | TEXT words | Static DATA/BSS |
| --- | ---: | ---: |
| Existing production EXEC loader | 183 | 0 |
| Disposable refactored EXEC caller | 119 | 0 |
| Shared DXR inspector/payload core | 161 | 0 |
| Shared capacity-bounded FULL/PREFIX adapter | 74 | 0 |
| Previous production PURE FULL reload leaf | 50 | 0 |
| Separate v1391 FULL restore C transaction | 359 | 0 |

The hypothetical combined EXEC+FULL replacement costs
`119+161+74=354` TEXT words versus `183+50=233`, i.e.
**+121 words before p10fold** (and without counting the new C
transaction). Thus the safety correction is staged but not a
resident-RAM saving. Do not link the v1391 C body alongside the
older restore transaction as a permanent duplicate. Net physical
memory benefit still requires actual folded KCORE measurements and
multiple inactive-process workloads at 32K/48K. Grouped compressed
PURE_PREFIX, PI/VFS ownership concurrency and booted lifecycle
regressions remain open.

## v1389/v1390: smaller shared loader and one FULL backing entry

**Staged and native PDP-6 tested, not linked into production
(2026-11-10).** The v1386 shared DXR inspector and FULL/PURE_PREFIX
payload engine now use contiguous BLT register-save frames rather
than seven individual AC PUSH/POP pairs. This preserves the established
PDP-6 AC10..AC16 calling convention and reduces the standalone
shared assembler object from 225 to **212 TEXT words**, zero static
DATA/BSS. Native v1386 parser, v1387 in-place restoration and the
v1388 disposable EXEC caller tests still PASS.

The `exec_image_full_backing_v1389` entry consists of a zero-prefix
selection and branch into the *same* backing adapter. It uses the
existing `proc_swap_unpack_backing` vnode/mount reconstruction,
VFS stat, validated DXR inspector and FULL payload reader; no
duplicate DXR parsing or decompressor code is added. The same
adapter's `exec_image_prefix_backing_v1386` entry invokes bounded
plain PURE_PREFIX instead. With FULL, the fresh, separate APR
destination receives the entire initialized DXR image and the
restoration transaction subsequently overlays the saved writable
suffix. Thus even compressed FULL DXR uses the original decoder
without overwriting the only authoritative source suffix.

The native PDP-6 `daimos-testkit/tests/validation/
exec-image-full-backing-v1390` test links the **actual staged**
v1381 RAM drop/restore transaction and real backing-unpack assembly
to this common FULL path. Controlled immutable providers exercise
plain and compressed DXR2, mount 1 and mount 18, failed short reads,
invalid TX2 and corrupt compressed input. The source suffix is
retained on failure and overlaid after successful FULL reload.
This PASS does not prove the production disk swap or 32K guest
behavior. The existing `proc_swap_reload_pure` is unchanged.

The final native test sweep also found an **out-of-contract
test-provider write**: the v1387/v1390 mock VFS header reader wrote
three words when the shared inspector requested only two. That
extra write could corrupt adjacent frame scratch and made a mount-18
case appear sensitive to an unrelated, wider compression-mode
predicate. Correcting the fixture to write TX2 only on a
three-word request (while DXR2 inspector reads TX2 separately)
restored reliable PASS in both predicate configurations. This
finding does not demonstrate any KCC, assembler or production VFS
regression; the corrected regression now respects actual VFS read
bounds.

### Size and capacity gate

| Independently assembled PDP-6 source | TEXT words |
| --- | ---: |
| Existing production EXEC loader | 183 |
| Alternate shared-engine EXEC caller | 119 |
| Shared DXR parser and payload engine only | **161** |
| Shared FULL/PREFIX backing adapter | **51** |
| Existing independent PURE FULL reload leaf | 50 |
| **Combined alternate EXEC + FULL replacement, before folding** | **331** |
| **Current EXEC + FULL leaf baseline, before folding** | **233** |

The hypothetical replacement is therefore **+98 words**, before
p10fold and before all new swapless MM/scheduler transaction code.
There are no new static data words, but this does NOT establish net
system RAM savings. In particular, measuring only a 256-word PURE
prefix saved from one suspended process while ignoring the resident
transaction and allocator code would be misleading. Net physical
RAM must be evaluated for real 32K/48K workloads, including all
inactive processes, actual KCORE linked text/BSS, heap/stack, and
pressure/retry costs. The executable v1390 size regression records
these isolated measures and rejects unexpected growth.

**Mandatory safety gate before connecting FULL to live swapping:**
the older three-argument PURE reload callback does not expose the
physical destination's allocated word count to the common DXR
reader. Even with a pinned immutable vnode and matching TX2 text
count, a larger initialized-image length in a malformed/backing
file could overrun the allocated APR extent. A final callback ABI
must carry explicit destination capacity and reject
`initialized_words + EXEC_USER_ORIGIN` greater than that capacity
*before* any read/decompress/write operation. The native tests use
valid controlled images and do not close this hostile-header bound.
The separate compressed PURE_PREFIX output-boundary constraint and
full 32K/48K tape-root integration remain open.

## v1386-v1388: actual shared DXR loader staging and cost gate

**2026-11-10: native tested, deliberately NOT production linked.**

`system/kernel/proc/exec_image_common_v1386.s` is the first executable
format/loading component shared *in executable native tests* between
ordinary EXEC and swapless inactive-PURE restoration. It defines a
single DXR1/DXR2 inspector and a single payload loader. The parsed
metadata occupies exactly four caller-owned temporary words:

| Word | Left 18 bits | Right 18 bits |
| --- | --- | --- |
| 0 | Entry offset | Uncompressed initialized image words |
| 1 | DXR flags | BSS words |
| 2 | Uncompressed PURE text words | DXR header words (2 or 3) |
| 3 | Relocation-map words | Compressed payload words |

The inspector validates the executable magic, entry and image limits,
DXR1/DXR2 file shape, BSS/flag bounds, compressed payload length and
text metadata/TX2 tag before publishing that four-word description.
The loader interprets it in `FULL` mode (DXR1/2 plain, DXR2 compressed
via the existing D6LZ36 decoder), or `PURE_PREFIX` mode (DXR2 PURE,
copy exactly the missing prefix and write **no** mutable word). The
current compressed-prefix case returns unsupported and triggers the
already proven full-image restore fallback. Neither mode allocates
kernel process tables or additional permanent state.

`exec_image_prefix_backing_v1386` is a bridge within the same staged
source, not a new parser: it uses `proc_swap_unpack_backing` to
reconstruct the original vnode and high mount ID, obtains file size
and regular-file type via VFS, checks the parsed TX2 text size
against the saved PURE record, and uses the common prefix loader.
It does not own MM, alter the process's execution state or bypass
existing immutable-vnode/unmount exclusion. Its caller is the
previously staged v1384 MM restore transaction. The original v1385
standalone loader is kept **only in daimos-testkit** as historical
validation evidence; it is not in DAIMOS source.

The native PDP-6 testkit results are:

- v1386: shared parser and image reader, DXR1/DXR2 FULL, compressed
  FULL delegation, STRICT prefix output boundaries, invalid shapes
  and short read rollback: PASS.
- v1387: real v1381 zero-copy PURE drop, v1384 MM claim/rollback/commit
  and the same v1386 shared loader, using the existing genuine backing
  unpacker, including mount 18, provider validation and safe
  compressed-prefix refusal: PASS.
- v1388: generate a throwaway EXEC caller from the current production
  `exec_load.s` with explicit source-anchor checks. Real PDP-6
  execution of this alternate caller together with the v1386 common
  engine passes plain DXR1/DXR2 and compressed FULL, RT root-only
  admission, malformed headers, VM NOMEM, read/decode failure and
  PURE-attachment rollback. The generated caller is **not** the
  production EXEC implementation: PASS.

The native tests caught a caller-clobbered register bug in v1386
header parsing (a DXR2 header count lost across the VFS callback).
The corrected routine preserves the value across VFS. The v1386
negative suite also corrected a test fixture which had failed to
generate the intended malformed file length. These fixes are in
the staged/test code; production source remains unchanged.

### KCORE size gate

| Comparable isolated PDP-6 source | TEXT words |
| --- | ---: |
| Existing production `exec_load.s` | 183 |
| Throwaway v1388 EXEC caller | 119 |
| Shared v1386 inspector + payload core, no bridge | 169 |
| **Combined alternate EXEC baseline** | **288 (+105)** |
| Full v1386 module with restoration bridge | 225 |

All these stand-alone sources have zero static DATA/BSS. The
combined sizes are **pre-fold component measurements**, not a final
KCORE link comparison. Adding the shared parser and loader to the
currently mandatory resident EXEC path would increase text words
before folding despite reducing the caller from 183 to 119. The
extra bridge adds 56 TEXT words relative to the 169-word parser/
payload core. This must be reduced, folded safely, or selected only
for a genuinely swapless configuration with measured net RAM saved
before any production link acceptance. The existing three fixed
resident C translation units and raw disk/drum swapping are unchanged.

**Still open:** actual source-to-production EXEC cutover and booted
32K/48K acceptance, `proc_swap_reload_pure` FULL path consolidation,
grouped D6codec bounded compressed PURE_PREFIX, hardware PI/VFS
concurrency and full MM/scheduler/EXIT/WAIT lifecycle. The v1388
native PASS does not close these conditions or establish a positive
production link-size delta.

## v1384/v1385: prefix restore proof and ONE shared EXEC loader

**Staged and native tested, not production linked (2026-11-10).**

The latest raw-PURE prototype can restore into its *original* physical
APR extent if the released prefix remains wholly free. A new
`mm_pure_expand_v1384.s` three-operation MM interface claims the
original lower prefix, reverses that claim after a read failure or
retypes the fully reconstructed image as PROCESS. Preflight checks
the exact slot owner, PACKED_PROCESS type, suffix length, exactly one
pin, proper APR alignment, physical 18-bit limits, preceding MM
neighbor and one continuous managed physical arena. No data is
copied or allocated by these MM operations, and the descriptor
remains PACKED_PROCESS throughout I/O, so the intermediate image
cannot be scheduled.

The staging transaction `proc_pure_restore_v1384.c` keeps
TRANSITION and executable backing immutability across the operation.
It briefly disables PI for the claim and for the MM/VM/storage-mode
commit, but executes potentially slow filesystem I/O with normal
interrupts. Failed partial reads reverse the MM claim and leave all
original mutable suffix words intact; an occupied or unmanaged gap
returns a distinct result permitting the already tested full-image
restore fallback. A compressed DXR not known to support bounded
prefix decoding also takes that safe fallback. The claimed memory
must not become a runnable PROCESS extent until the complete PURE
prefix has been validated and loaded.

`daimos-testkit/tests/validation/pure-ram-prefix-v1385/exec_prefix_fixture_v1385.s`
verifies the actual native DXR2 plain-prefix operation, including
canonical vnode reconstruction
for mounts 1 and 18, fifth-bit mount ownership, exact TX2 text size,
full-sector prefix bounds, malformed headers, short reads and
compressed-format rejection. It directly reads only
`[APR_base + 020, APR_base + pure_prefix)`; it never writes the
already-owned mutable suffix. These tests PASS in
`daimos-testkit/tests/system/pure-ram-prefix-v1384` and native
`tests/validation/pure-ram-prefix-v1384`/`pure-ram-prefix-v1385`.
An inverted PDP-6 arena comparison was found by native tests and
corrected. No active KCORE linking is altered.

### Loader consolidation (required before production)

The v1385 plain-prefix function exists **only in daimos-testkit**, not
as a second DAIMOS production DXR loader. Existing
`exec_load_process` handles initial
EXEC image construction, while `proc_swap_reload_pure` is used for
full-image PURE reconstruction. A future single DXR image-loader
engine must replace duplicated header parsing and payload handling
and expose an explicit output mode:

| Shared loader mode | Destination contract |
| --- | --- |
| `FULL` | Initialize the complete executable image in a new APR allocation; the EXEC caller owns VM setup and process publication |
| `PURE_PREFIX` | Write only the missing immutable prefix to an already-claimed MM extent, never user ACs or the existing mutable suffix; RESTORE owns MM rollback and process publication |

One DXR parser, one plain payload reader and one accepted grouped
compressed decoder should implement both modes. Do **not** collapse
EXEC's pathname/privilege/runtime admission or RESTORE's existing
process ownership/MM rollback into the low-level image loader.
Those lifecycle operations are distinct and should remain outside
the shared core. The present EXEC plain payload path already makes
a direct VFS read into the allocated image; there is no separate
runtime relocation pass to duplicate. A future grouped compressed
decoder must enforce a hard output boundary, including MATCH tokens
crossing it, or provide independently framed text. Existing
unbounded/full-image compressed decoding is never permitted to
write over the inactive mutable suffix.

### Measurements and remaining limits

| Isolated staged PDP-6 object | Text words | Static DATA/BSS |
| --- | ---: | ---: |
| v1384 prefix transaction, C89-compatible source | 323 | 0 |
| v1384 MM claim/release/commit, assembler | 130 | 0 |
| v1385 plain-prefix testkit fixture, **not production source** | 78 | 0 |

These 531 pre-fold words are **not** production linked-text growth.
The v1385 test-only 78-word fixture must be replaced by the existing
EXEC image-loading engine rather than shipped beside it. Full
linked text and shared-code savings have not been measured.

For a 1024-word process whose first 256 words are reconstructible
PURE, in-place restore holds only the original 768-word suffix plus
the reclaimed 256-word prefix (1024 process words total), instead of
temporarily holding the 768-word suffix plus another complete
1024-word allocation. This assumes the gap remains free and excludes
kernel, device and VFS workspaces. When the gap has been reused,
full-image restore still needs an APR-aligned allocation elsewhere.
Actual scheduler/PI integration, grouped-codec bounded output, the
consolidated loader and a 32K/48K swapless guest remain open.

## v1383: one-word size geometry and safe prefix-only restoration

**Design reconciliation, 2026-11-10. No new production code or test
acceptance.** A PDP-6 word can encode the two uncompressed lengths
needed for swapless PURE restoration:

| Left half (18 bits) | Right half (18 bits) |
| --- | --- |
| Exact immutable text words from the DXR2/TX2 executable | Current retained mutable user-memory suffix words |

These are **uncompressed** counts, not the number of stored compressed
stream words. The correct current suffix count is NOT merely the
original writable DATA section of the executable: runtime BSS,
stack, heap, other writable words and APR padding must be retained
exactly. The low 020-octal user AC words are saved in the stable
u-area and are not copied from the executable during text reload.
For the present full-sector PURE-drop convention:

`released_prefix = (text_words + 020) & ~0177`

`mutable_suffix = current_VM_allocated_words - released_prefix`.

Validate that `released_prefix` is nonzero, less than the current VM
allocation, exactly 0200-word-sector aligned, and the sum is an
APR-legal allocated size. The existing DXR2 header already contains
18-bit *uncompressed initialized image* words and the TX2
uncompressed text count. The process `vm_state` holds its full
current 18-bit VM allocation; the already resident/saved PURE
backing word contains its validated text count. That backing word
currently encodes only **14** text bits alongside vnode and mount
identity. Therefore the two proposed sizes can be reconstructed at
quiescence with **zero extra permanent per-process words** for the
existing supported PURE range. Changing the 14-bit backing limit is
an independent ABI decision; merely defining a larger 18-bit field
elsewhere does not extend it. A packed `text,,suffix` word is an
optional *transient representation*, not automatically an additional
KCORE table or slot.

### In-place RESTORE and compressed executable caveat

The v1381 DROP releases only the lower contiguous
`released_prefix` words, retaining the raw suffix at its original
physical address. A matching in-place RESTORE can reclaim that gap
and reverse the MM descriptor rebase if, and only if, the whole
prefix remains free, the reconstructed full base satisfies APR
alignment, and a transactional MM/PI/PROCESS owner transition is
validated. Reconstruct **only** the missing words at addresses
`[base + 020, base + released_prefix)`, preserving the suffix
bit-for-bit. A plain DXR2 loader can request the shorter count;
neither user ACs nor any mutable DATA may be overwritten.

The current `proc_swap_reload_pure` compressed DXR path is NOT safe
for this operation because it invokes the existing decoder on the
entire initialized image. Furthermore, the trusted-backing
`d6lz36_decode_vfs` ABI can stop when a requested output count is
reached, but the fixed `decompressor.inc` MATCH path subtracts the
whole token length and rejects a match if it runs beyond the
requested prefix end. It cannot simply be passed the smaller
prefix length on every existing compressed DXR. Metadata alone does
not change token boundaries or guarantee partial decoding.

At the atomic grouped-D6codec production cutover, choose either
an explicitly bounded-prefix decoder that safely clips the final
match without touching writable memory, or an independently framed
immutable-text segment guaranteed by the producer. Preserve the
global **no permanent legacy decoder** policy, and test match
crossings, exact lengths, short files, corrupt streams and
restoration rollback against actual native PDP-6 code. A single
36-bit geometry word is adequate for the sizes; stream framing and
data safety require separate proof.

If another MM allocation has occupied the released prefix, size
knowledge cannot make it free. The already staged safe fallback
allocates a complete, aligned APR extent elsewhere, decodes there
and then overlays the exact retained suffix. If memory is
insufficient, the process stays inactive until the scheduler can
retry without violating source ownership; never overwrite the
only valid mutable suffix. This is why in-place RESTORE is a fast
path with a rigorous unavailable-gap fallback, not a guarantee
that any suspended process can resume immediately.

This v1383 section corrects the earlier framing of full compressed
image reload as the only possible design. It records a feasible
prefix-only alternative, **not** its completed implementation.
The v1381/v1382 native component acceptance and production linking
state remain unchanged.

## v1381/v1382: zero-copy PURE release by an atomic MM rebase

**Preferred staged PURE-only design, 2026-11-10. Not production linked.**

Earlier v1379 DROP needed a second physical allocation for the raw
mutable suffix and copied every word into it before releasing the old
APR extent. This was unnecessary: the suffix already occupies the
correct high-address part of the original physical extent. Retain it
without a physical copy by changing the original sorted MM extent
descriptor from:

`PROCESS [base, base+words)`

to

`PACKED_PROCESS [base+pure_skip, base+words)`.

This releases the lower PURE prefix immediately as a free gap in the
implicit MM allocator. The descriptor stays in the correct sorted
position: the preceding extent still ends no later than the old base,
and the next extent starts no earlier than the original end. The
owner and physical pin count are preserved. No extra extent or
per-process metadata is allocated, and no user memory word moves.
The retained suffix is NOT an APR mapping: it is explicitly
nonrunnable until full restore.

`system/kernel/mm/mm_pure_rebase_v1381.s` is a new standalone assembler
primitive, with its own header, requiring one pin, an owner-matched
MM_TYPE_PROCESS extent, exact old size, 02000-octal APR alignment,
nonzero 0200-octal-word-multiple PURE prefix shorter than the original
process, and a range within the full 18-bit physical address space
(exclusive upper boundary `01000000` octal). It checks ALL failure
conditions before two descriptor stores. Rejected requests change
neither MM metadata nor data. No call to the old
`mm_process_shrink_pinned` is made, because that API must continue
to enforce the 02000-octal APR process size granularity.

The staged `system/kernel/proc/proc_pure_ram_v1381.[ch]` replaces v1379
DROP while retaining registry-free RESTORE and DISCARD behavior.
It validates the resident state and exclusive process transition,
pins the original extent, disables PI for a short critical section,
calls the rebase primitive, and publishes stable PURE backing,
nonresident VM base and the existing packed mode **before restoring
PI**. Inactive suffix pin removal uses the new physical base.
An MM validation failure leaves the original image complete,
restores PI and removes its temporary pin. If post-commit unpin fails,
the caller retains TRANSITION; neither a partial process image nor
spurious disk swap record becomes runnable.

The primitive itself does not disable PI: the caller must exclude
interrupt-time MM mutations until the MM record and process storage
mode are consistent. The critical section contains no data copy,
filesystem I/O, memory allocation or retry loop. The descriptor
search is bounded by MM extent count. The only fallible step before
publication is the exact preflight and it has no side effects.

The v1381 host test exercises zero additional physical allocations
or frees during DROP, repeated drop/restore, exact suffix integrity,
MM pin/type/owner validation, PI pairing, allocation and reload
failures, and ZOMB DISCARD. The native PDP-6 v1381 regression
links the actual KCC transaction C, new MM assembler rebase and
existing assembler mode handlers. Positive, negative and physical
18-bit boundary cases PASS. The v1382 native regression additionally
links actual resident PURE vnode/mount busy scanning: source guard
stays effective through the zero-copy descriptor rebase, return to
resident state, second drop and terminal release. High mount 18 and
low-slot alias rejection PASS.

During this work, the older host MM fixture was corrected to reject
**overlapping physical intervals**, not merely equal allocation
starting addresses. The previous mock could incorrectly allocate a
full restored process image over its still-authoritative in-place
suffix after a failed reload. With the correct check, v1377 and
v1381 host regressions PASS. This is a test-fixture correction, not a
production allocator change.

### Memory and text cost

| Example, decimal physical words | v1379 copy/drop | v1381 zero-copy |
| --- | ---: | ---: |
| Original APR process | 1,024 | 1,024 |
| Additional suffix allocation during DROP | 768 | **0** |
| Peak during DROP | 1,792 | **1,024** |
| Final inactive suffix | 768 | 768 |

For the same source/object compilation mode, v1379 DROP/RESTORE/
DISCARD C was 567 PDP-6 KCC TEXT words; v1381 C is 518 and
`mm_pure_rebase_v1381.s` adds 70, total **588 standalone
pre-fold words**, zero static DATA/BSS. The increase is 21 TEXT
words for zero temporary RAM allocation on DROP. These are
component measurements, NOT a permanent KCORE linked-size delta.
Neither source is in the production Makefile.

**Still open:** RESTORE must presently retain the raw suffix while
allocating a complete aligned APR destination and reloading PURE.
For compressed executable formats the reload may overwrite the
entire destination; it cannot simply reconstruct a missing prefix
in place and overwrite the only valid writable data. Therefore
RESTORE peak and no-partner pressure tests remain blockers, as do
hardware-PI allocator concurrency, live scheduling, the real
VFS/provider lifetime, EXEC/RUN/WAIT/EXIT and a full 32K/48K
swapless tape-root guest. Native tests use controlled MM/PI/provider
fixtures; they do not certify any of these missing integrations.

## v1379/v1380: registry-free RAW PURE staging (not production)

The v1377 dense/sparse inactive-process layout registry is **not
necessary for the raw PURE-only case**. In the new, still unlinked
`system/kernel/proc/proc_pure_ram_v1379.[ch]`, the already authoritative
MM_TYPE_PACKED_PROCESS extent contains exactly the saved mutable
suffix and its process-slot owner. The original logical process size
is preserved in `vm_state`; the PURE sector-rounded omission is
derived from the validated executable backing record. Exact MM
type/owner/count checks reject duplicate or mismatched extents.
This avoids both extra per-slot metadata and lookup through an
independently allocated registry. The existing resident
`proc_storage_expect_v30` and `proc_storage_publish_v30` assembler
operations still own the two-bit process mode transition.

The source is compiled only with `DAIMOS_SWAPLESS_PURE_RAM_V1379=1`;
no production KCORE build selects it, and the v1377/v1376 prototypes
remain unlinked. Both host and native KCC/PDP-6 v1379 tests PASS
repeated drop/restore, original mutable memory preservation, raw
fallback/no-codec behavior, source/destination MM ownership and
failure rollback, and ZOMB discard. The PDP-6 test links the real
assembler process mode service rather than the staged C map helper.

The v1380 native test also links the *actual* existing
`proc_swap_backing_busy` and `proc_swap_mount_busy` assembler
code. It proves that a PURE process's vnode and mount remain busy
while resident, across the old APR free, while stored as an inactive
raw RAM suffix, during the full reload, across inactive suffix free,
and after the second drop. The saved backing record in the stable
u-area supplies nonresident immutability; after terminal discard
the guards no longer report busy. The high public mount-id 18 is
tested with the separate credential extension bit and negative
checks for aliasing lower mount IDs. Ordinary resident RAW/COMPRESSED
swap and previously accepted OOM guards are unchanged.

### Resource measurement and remaining risk

| Isolated PDP-6 KCC text | Words |
| --- | ---: |
| Previous v1377 transaction alone | 672 |
| Revised v1379 transaction alone | **567** |
| Staged C storage-map/layout/owner dependencies removed | 121+310+273 |

The **704 removed words** are only an isolated test-link difference:
the production kernel already owns assembler mode handling, and the
old C registry never shipped. Neither number is a production
linked-size delta. v1379 has zero static data/BSS. Source-protection
acceptance is limited to the tested VFS busy scanners and controlled
MM/provider transitions, not arbitrary concurrent provider mutation
or a booted guest process.

The present drop still needs a **separate temporary copy of the raw
suffix** before freeing the old APR extent. This makes peak memory
pressure an explicit unsolved blocker for a 32K swapless machine.
The current `mm_process_shrink_pinned` demands 02000-octal-word
process extent granularity and cannot retag a live PROCESS extent
as a compact PACKED_PROCESS extent. Investigate a properly serialized
in-place MM conversion with a validated no-fallible-step-after-overwrite
contract. Never simply copy into the original extent and shrink
afterward without rollback, or publish released tail words while the
copy still uses them.

This work does not complete KINIT capacity setup, actual scheduler
admission, full PROCESS/EXEC/EXIT/WAIT teardown, physical 32K/48K
tape-root boot, or the required peak-RAM/high-water measurements.
Those remain open before production activation.
Fault tests currently cover allocation refusal, failed executable
reload and MM free refusal, not every ambiguous MM pin/unpin outcome.
If unpin fails with uncertain ownership, TRANSITION remains held and
an explicit owner-recovery path is required before production use.

## v1377: staged PURE-only inactive process lifetime

DAIMOS adds `system/kernel/proc/proc_pure_ram_v1377.[ch]`. This is
new, standalone, **unlinked** transaction code. It is not a production
KCORE or scheduler integration.

The three entry points are `proc_pure_ram_drop_v1377(slot)`,
`proc_pure_ram_restore_v1377(slot, reload)`, and
`proc_pure_ram_discard_v1377(slot)`. They reuse the existing
`proc_storage_runtime_v29` two-bit mode map, the inactive dense/sparse
`proc_packed_runtime_v38` descriptor layout and MM-authoritative
slot/type/owner checks. These are only storage primitives: their
historical PACKED names do not mean the new payload is compressed.
The retained image is **raw**, containing exactly the writable suffix
after the omitted complete 0200-octal-word PURE sectors. An initially
running process must first be quiesced, have its user AC state saved in
the stable u-area and hold its lifecycle transition.

Drop requires valid PURE text backing, a sleeping/stopped process,
transient source exclusion, a nonpinned process extent, and an available
registry slot. It pins the old APR mapping, reserves and pins a new
unaligned managed-RAM extent, copies every mutable suffix word, reserves
the registry layout, and frees the old process extent before publishing
the inactive storage state. Only then may TRANSITION be cleared. An
owner-checked failed operation before the MM free does not publish a
partial image. The stable u-area stores the exact backing key to
support the existing nonresident vnode/mount exclusion contract.

Restore verifies the same owner, backing, original word count, PURE
boundary and raw saved suffix length before allocating the complete
APR-aligned process extent without recursive reclaim. It initializes
omitted words, calls the validated executable reloader (which may
decode an entire compressed DXR payload), then overlays every saved
mutable word, before freeing the inactive MM extent. Failed image
reconstruction preserves the saved RAM extent. A successful restore
publishes one complete resident process extent. Discard releases
validated inactive RAM on terminal teardown (including ZOMB), clears
the backing record and the high mount-slot credential bit, and keeps
the process's broader EXIT transition under caller control.

**Build isolation:** the translation unit deliberately refuses
compilation without `DAIMOS_SWAPLESS_PURE_RAM_V1377=1`. It references
no BACKSTORE variables or services and cannot be accidentally pulled
into a normal swap-enabled build. The flag is only enabled by focused
tests today. Future minimal `TAPE_AUX=none` wiring must explicitly
select the source and its proven runtime dependencies.

The testkit host suite executes the actual transaction C and
storage-layout/owner C against instrumented MM and immutable-provider
stubs. Dense and sparse layouts, three consecutive cycles, allocation
refusals, provider reload failure, destination/source MM release
failure and ZOMB discard pass. It revealed a missing source unpin on
failed restore allocation; the production candidate now releases it
before returning. The same source compiled with PDP-6 KCC executes
on SIMH; the target test covers raw drop, a failed full-image allocation
and retry, correct mutable-data overlay after mock DXR reloading, second
drop and terminal discard. Both suites PASS.

The isolated KCC object is 672 text words and zero DATA/BSS.
The previously staged C support objects linked in the focused test are
121 words (storage map), 310 (layout) and 273 (ownership): **1376 text
words pre-fold total**, not the live KCORE increase. The test's own
533 words and fixture data are not production costs. Some layout/planning
code belongs only in KINIT, and the real kernel already has assembler
storage-mode operations; a future production split must account for that
before declaring these prototype costs irreducible. The historical
header interfaces emit nonfatal KCC identifier-truncation notes.

**Still open:** actual VFS executable write/unlink/truncate/unmount
serialization across drop publication, exact 32K/48K boot-time registry
allocation, real MM and scheduler wake/retry, process EXIT/EXEC/WAIT/PID
reuse, saved AC restoration and provider failure tests. No production
activation, full boot, RAM peak, size benefit or time benefit has been
measured or claimed. With a single typical 02000-octal process and
0400-octal omitted prefix, the data reduction is only 256 words before
descriptor/codec/code costs; this alone cannot justify adding 1376
resident text words. The v1376 RAM compressor remains optional and
unlinked. No disk swap behavior was changed.

## v1376: first staged codec component accepted (not a process feature)

The DAIMOS source now contains `system/kernel/core/kpack_ram_rle_v1376.c`
and its header, **unlinked from production**. It is a no-allocation,
lossless 36-bit-word codec for zero runs, repeated nonzero words and raw
literals. It does not require aligned output, disk sectors, an MM table,
BACKSTORE or any dynamic scratch buffer. A measurement pass computes the
exact output length; the caller must reject allocations that would not
produce net physical-RAM savings, including metadata and restore headroom.
This is intentionally a simple memory-first staging baseline, not a claim
that its compressed size or CPU cost beats grouped D6codec.

`daimos-testkit/tests/system/swapless-ram-codec-v1376` verifies host
roundtrips, exact-capacity refusal, malformed control/length rejection
and the maximum 18-bit word count. The native regression in
`tests/validation/swapless-ram-codec-v1376` compiles the same C source
with KCC and runs its encode/decode and failure ABI on a real PDP-6
SIMH instruction path. Both PASS. KCC/DAS/DLINK independently accept
the implementation (449 text words, zero static data/BSS in the isolated
KCC-linked object). **449 words is too expensive to accept without a
complete swapless KCORE/MRES savings comparison**; neither compression
nor PURE-only inactive-state publication is integrated into production.
The initial PURE-only no-compression lifecycle must still be established
before any scheduler admission or RAM-compression integration.

## Applicability and exclusions

This mechanism is for configurations with **no process backing device**,
especially the build-time minimal `ROOT=tape TAPE_AUX=none TAPE_TMPFS=0`
profile. Discovering a disk or drum at runtime must not implicitly enable
disk swap or alter this mode. The initial target is stopped, sleeping or
otherwise schedulable-but-inactive user processes; the current running
process, active executive continuations, real-time pinned owners and any
unresolved VM/MM/provider transaction cannot be discarded or compressed.

Two independently enabled techniques are permitted:

1. **PURE executable-text dropping.** On a process with validated immutable
   executable backing and a software-immutable PURE prefix, retain all
   changeable state in RAM and omit the reconstructible PURE prefix from its
   inactive representation. At restoration, reload that prefix from the
   original vnode, *then* restore saved writable words exactly.
2. **Opt-in RAM compression of changeable process memory.** An approved
   process may retain a lossless compressed copy of its mutable user image
   **in managed RAM** while inactive. If compression is not worthwhile or
   cannot be completed atomically, preserve its original resident image.
   There is no disk record, disk fallback or BACKSTORE allocation.

PURE dropping is not synonymous with MEMFS execute-in-place (XIP): a normal
PURE DXR is still copied to a private APR extent while executing, whereas
restricted XIP executes from a pinned MEMFS-owned extent under a different
trusted no-write policy. Neither feature adds general shared text mappings.

## Hardware and layout invariants

The PDP-6 APR provides one contiguous process mapping, one relocation base
and 02000-octal (1024 decimal)-word alignment, **not** independent
text/data mappings or a text write-protection bit. Thus PURE cannot be
freed independently while a process is executing with private mutable data.
The first implementation must quiesce the process, save its CPU/u-area
context, and transactionally create an **inactive representation** before
releasing the original APR process extent. Restore the full contiguous APR
extent before that process can run again. Do not emulate page faults or
weaken APR isolation.

The existing single-word PURE backing record and
`proc_swap_reload_pure` are useful starting points; they do **not** already
provide an accepted swapless representation. In particular, the current
disk-swap omission rounds the PURE prefix down to full 0200-octal sectors.
An in-RAM representation may use exact word boundaries if that reduces
RAM, but it must retain the entire mutable boundary, including its
partially shared text/data word or block, and account for relocation,
startup state, zero-fill/BSS, heap and stack. Never infer immutability
solely from a compressed-executable header.

Persist the original logical image word count, PURE boundary, executable
vnode/mount generation and source integrity contract, retained mutable
word count, actual representation type and exact encoded word count.
Reuse existing authoritative process/u-area/storage-map fields where
possible; measure any extra per-slot state against actual RAM savings.
A process must never be marked resident while its full APR image is absent.

## Transaction and safety policy

- **Admission:** restrict PURE dropping to validated immutable image
  sources and valid PURE metadata. Reject self-modifying or unknown images.
  Obtain and retain the executable/provider lifetime guard *before*
  publishing an inactive representation. Reject write, truncate, unlink
  and unmount while an inactive process depends on that source.
- **Opt-in:** compression eligibility belongs to the validated current
  executable or explicit privileged policy, not an untrusted user bit.
  Default is uncompressed. EXEC resets admission; any supported fork must
  define eligibility inheritance. The actual RAM representation is recorded
  independently from eligibility.
- **Reclaim:** quiesce the candidate with valid scheduler, PI6 and MM owner
  state. Prepare a distinct managed-RAM retention extent, copy or compress
  the mutable image, and validate exact reconstruction/count before
  changing published ownership. Only then free the original process
  extent. No in-place overwrite of the only copy, hidden disk write, or
  release of pinned memory is permitted.
- **Benefit test:** reject non-shrinking compression *after* accounting for
  encoded words, metadata, preserved context and temporary peak RAM. Try
  PURE omission without compression when beneficial. If there is no
  provable net gain, keep the original resident image.
- **Restore:** reserve a complete APR-aligned process extent (including
  writable words), reconstruct immutable PURE from the guarded vnode,
  restore/decompress mutable words, validate exact counts and ownership,
  then atomically publish runnable process state. On failure, retain the
  original inactive representation and its backing guards; do not run a
  partial image. A nonrecoverable lost backing source is a typed failure,
  not permission to invent replacement text.
- **Progress:** a compact RAM representation does not guarantee enough
  memory for its later expansion. Reserve/measure restoration headroom,
  apply bounded reclaim/compaction attempts, and fail safely on genuine
  insufficient physical RAM. Prevent cyclic waits on a filesystem provider
  owned by a process that must first be restored. Restore never allocates
  disk backing or forces a provider lock free.
- **Teardown:** EXIT, TERM, fatal event, EXEC replacement, PID reuse and
  MM failure rollback must release exactly the owned representation and
  any executable hold once, even if the image is inactive. Preserve
  per-process status and parent WAIT semantics.

## Implementation order and acceptance

1. Establish an exact current KCC/KCORE size, call-graph and physical-memory
   baseline for a **swapless** 32K/48K tape-root boot. Confirm which SWAP
   gateways/bookkeeping the no-backing link still truly needs; do not
   reintroduce removed disk-swap services in that profile.
2. Specify the minimal inactive-state descriptor and validation contract
   against real `proc_storage_map_v28`, u-area, MM owner/pin and scheduler
   fields. Reuse validated PURE source/loader functions without adding a
   new duplicate decoder.
3. Implement PURE-only stop/drop/restore first, with original writable
   bytes retained verbatim in RAM and **no** compression requirement.
   Verify native PDP-6 resume, repeated cycles, executable protection,
   low-memory and failure rollback.
4. Only then add explicit opt-in mutable-RAM compression using an already
   validated native lossless codec where feasible. Benchmark the whole
   KCORE+MRES growth, stack/scratch, peak temporary RAM, compression time
   and restore time. A codec shared with executable decoding must not be
   linked a second time. Reject negative net RAM on 32K profiles.
5. Integrate scheduler reclamation and VM/BRK/EXEC teardown, prove bounded
   memory-pressure progress, then run mixed PURE/non-PURE,
   compressible/incompressible, multiple inactive processes, provider
   failure, parent WAIT and interrupt/preemption tests. All new tests live
   in `daimos-testkit`, including SIMH and PDP6-AAP where applicable.
6. Repeat the full KCC linked permanent-size and real physical high-water
   checks. Only production-enable after accepted 32K swapless native
   tests; keep the disk/drum raw-swap build behavior unchanged.

The inactive `proc_packed_*` v1-v46 sources are **historical prototypes**:
they are not linked into the production process lifecycle. Their MM
ownership and rollback ideas may be reviewed, but their old compressed
disk-swap eligibility/representation plan is retired. Do not reactivate
them by renaming or linking them without source-level integration and
independent native acceptance.

## Status of production source at this decision

`proc_swap_pdp6.s` currently omits PURE sectors when writing a process
to disk swap and reloads text through `proc_swap_reload_pure`. It does
not drop PURE text in a swapless system. `vm_pdp6_swap.c` still owns
ordinary raw swap bookkeeping. No swapless inactive RAM compression,
restore state or scheduler admission has been enabled. This document is
an implementation specification and not a claim of completed code.
