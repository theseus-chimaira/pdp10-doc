# Swapless process-memory reclamation

Status: accepted KCORE direction, 2026-11-09 v1375. **Staged PURE-only
and registry-free transactions pass through v1380; no production scheduler or full
swapless guest acceptance.** This document supersedes the earlier proposal
for executable-opt-in *compressed disk swap* (old decision 5/P0-D). Raw
disk/drum process swapping is unchanged. Compression of disk-swap records is
not part of the release plan.
The old P0-D *feature-list* label must not be confused with DAIMOS's
separate v1298 **P0-D OOM fault/regression matrix**, which stays accepted.

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
