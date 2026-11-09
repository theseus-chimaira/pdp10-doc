# Swapless process-memory reclamation

Status: accepted KCORE direction, 2026-11-09 v1375. **Not implemented or
native-accepted.** This document supersedes the earlier production proposal
for executable-opt-in *compressed disk swap* (old decision 5/P0-D). Raw
disk/drum process swapping is unchanged. Compression of disk-swap records is
not part of the release plan.
The old P0-D *feature-list* label must not be confused with DAIMOS's
separate v1298 **P0-D OOM fault/regression matrix**, which stays accepted.

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
