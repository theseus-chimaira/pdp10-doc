## 2026-11-11 v1498: foreign TERM policy for nonresident DISK owners

Testkit v1498 (`f9ab499`) executes the unchanged production
`proc_event_apply` on a native PDP-6 interpreter with a nonresident
DISK process. WAIT_CHILD and saved executive-mode PC defer TERM to
the owning executive continuation, even though the user VM is absent;
repeated TERM coalesces in the stable u-area pending bits. Clearing
WAIT/pending state and selecting a user-mode saved PC permits foreign
finalization. Controlled VM teardown refusal retains DISK ownership;
a separate successful finalize commits. A deliberately unsafe source
mutation bypassing the saved-PC guard is rejected. No kernel code,
RAM/ABI or production scheduler changes.

This validates the policy but does not prove which gate the v1495
256K physical guest hit. A v1499 disposable physical diagnostic is
intended to capture the actual saved PC/WAIT/FD-control state and
the event's return and branch witnesses. The entire fault-specific
physical BACKSTORE acceptance remains OPEN. See DAIMOS
`design/kcore-backstore-term-policy-v1498.md`.

## 2026-11-11 v1497: physical corrupt-BACKSTORE lifecycle classified

Fresh 256K KCC PDP-6 guest v1495 reached real HALT/EXIT. One physical
allocation bitmap bit was cleared after a successful backing read.
Final counters: one fault, three matching `backstore_free` calls, one
bitmap repair, two negative swap-in service entries, two real
`proc_event_apply` returns, final 0/0 process-swap/BACKSTORE blocks.
All 18 F07 guest acknowledgments and 18 reaps were present after the
v1496 SIMH-banner parser correction. Protected-OOM, MM peak, slots,
swap stack and independent owner-lifecycle replay verifiers PASS.

Crucially, both TERM calls returned **without** TRANSITION/PREVM
finalization, and the third matching backing free lacked PREVM as
well. Therefore it was **not demonstrated to be an idle PREVM
finalizer retry**. The v1494 injection assumption is invalid, and
its dedicated verifier correctly fails. The normal v1336 verifier
correctly disallows all fatal swap-in events; it remains unmodified.
The KCORE `proc_event_apply` foreign-fatal path explicitly defers TERM
when a saved private executive continuation or wait class prevents
safe remote teardown. The v1495 result is consistent with that
policy; the exact predicate/event result was not observed, so neither
an unsafe kernel fault nor successful immediate finalization is
proven. Do not bypass the existing stack-safety deferral.

Fault-injection qualification remains OPEN: classify the exact
deferred-event branch and pending TERM bit, prove owner-safe delivery
after restored backing or an explicitly finalizable target's
PREVM/idler cleanup, and separately qualify permanently refused
backing. Maintain a separate strict fault-specific verifier rather
than weakening v1336. Detailed source/replay/sha256 evidence in
DAIMOS `design/kcore-backstore-physical-f07-diagnostic-v1497.md`.

## 2026-11-11 v1496: F07 physical fault-test milestone integrity

The first disposable v1494 full-guest bitmap-corruption run reached
physical PDP-6 HALT with one injected fault, three matching checked
BACKSTORE release attempts, one bitmap repair, and zero final swap
blocks. It did NOT pass end-to-end acceptance: the TERM witness was
zero, the composite diagnostic error counter was one, and normal
v1336 verification rejects fatal swap-in failures by design.

An independent F07 test harness bug is fixed by testkit v1496:
the SIMH startup banner's git revision
`edd19b17+uncommitted-changes` was counted as a guest `+` child
acknowledgment. The saved guest output actually contained exactly
18 guest acknowledgments and 18 reaps. The shared runner now counts
only after the real F07 user `<` marker and before the simulator
HALT/examine trailer. Exact child counts are unchanged; five host
positive/negative parsing cases PASS. Commit `f106218` in
daimos-testkit. This is not a kernel fix.

The regular v1336 acceptance requires zero `swapin_fail` events;
fault-injection acceptance must use an additional explicitly scoped
verifier that proves the expected failure(s), TERM/finalizer ownership
and release, without relaxing the original normal-swap acceptance.
New v1495 disposable-guest diagnostics were committed in testkit to
separate scheduler failure, real event return, TERM witness stack
offset, and PREVM marker timing. The complete physical gate remains
OPEN pending verified evidence from that profile.

## 2026-11-11 v1494: PREVM idle-finalizer retries refused VM teardown

The production `proc_finish_slot` and `proc_finish_service_v881`
were executed in source-linked PDP-6 testkit v1494 with controlled
first VM release refusal, subsequent success, and two independent
u-area MM refusal retries. FINALIZE+PREVM, TRANSITION and exact VM
ownership were retained while release was refused. The idle finalizer
then committed VM destruction once, did not rerun sleeping HUP, and
completed zombie publication only after u-area release. A mutation
that falsely treats VM refusal as success failed. Existing v1081
refused-TTY and v1086 PI6-masked finalizer tests also PASS. No KCORE
production source/ABI/RAM change.

The VM backend in v1494 was test-controlled; v1493 independently
tested real checked BACKSTORE swap-in and the scheduler with an event
observer. The *combined* physical 256K corrupt-bitmap -> TERM ->
PREVM finalizer -> eventual owner resolution sequence remains OPEN;
separate passing native boundaries do not qualify this full guest.
See DAIMOS `design/kcore-backstore-prevm-finalizer-v1494.md`.

## 2026-11-11 v1493: checked BACKSTORE refusal reaches scheduler TERM

The v1405 swap-in checked-free transaction is now source-linked with the
actual `proc_swap_mres_service_v481` slot-0 scheduler service in native
PDP-6 testkit v1493. A deliberately incomplete BACKSTORE bitmap makes
the production free fail. The original DISK record and usage counters
survive, the replacement MM extent is rolled back, and the scheduler
requests exactly one TERM instead of erroneously setting READY_ONCE or
interpreting a corrupted bitmap as ordinary memory pressure. Two
executable negative mutations (unchecked free, false scheduler ready)
are rejected. The original v1405 checked-free transaction regression
also PASSes on the current source, including retry after repairing the
bitmap. No permanent kernel code or ABI changes were needed.

This is **native service-level evidence**, not full guest termination
and finalization: `proc_event_apply` is an observer stub in the v1493
fixture. A physical 256K corrupt-bitmap owner/finalizer test remains
OPEN and cannot be inferred from accepted normal v1405/v1466 F07
guests. See DAIMOS `design/kcore-backstore-scheduler-refusal-v1493.md`.

## 2026-11-11 v1490: BCACHE invalidation independent of MM release

The installed DTFS DTC write veneer previously used `bcache_reclaim()`
as a pre-write invalidation operation. That routine calls MM using
AC1..AC3, clobbering the physical DTC unit, block and buffer arguments
before the device service. Furthermore, a refused `mm_free` left both
cache tags valid even when the caller needed to discard them for media
write or identity reuse. This could return stale clean data.

KCORE now exports `bcache_invalidate_all()`: clear both clean tags
without freeing the 0201-octal-word private slab and without changing
AC1..AC3. `dtfs_dtc_write` uses this before the hardware call;
`bcache_reclaim` uses it before MM free. Refused free retains the MM
owner pointer for retry but can no longer retain a stale cache identity.
No second cache, new allocation, process state, or device record was
introduced. DAIMOS production commit `3835c5ae`; daimos-testkit
`d81b039` tests the real PDP-6 source with nonzero physical DTC unit,
exact write arguments, read-after-write, refused MM free, retry and
two executable negative controls, all PASS. Existing allocation,
multisource and v688 cache checks PASS; KCC full KCORE and DTFS MRES
link succeeded. Physical installed-provider tape-root acceptance is
separate and not inferred from these native tests. Source evidence:
DAIMOS `design/kcore-bcache-write-reclaim-v1490.md`.

## 2026-11-10 v1406: full 256K checked-release guest acceptance

The v1405 production SWAP MRES passed the fresh strict v1349 256K KCC
PDP-6 DSK/D6FS F07 full-guest postflight, including HALT and SIMH EXIT.
Actual normal physical swap: 34 successful swap-ins, one true MM allocation
failure/no-partner cooldown, no exchanges and zero outstanding process
swap/BACKSTORE blocks at HALT. All four independent physical MM owners
were released after cooldown (sample 13) with no error. Original held-
provider and child cleanup gates and all six owner-evidence negative
controls also PASS. This is a new full-system acceptance for the committed
v1405 checked-free change, not merely a native source-linked test.

Details and archived SHA256-linked evidence:
`KERNEL/SWAP-FULLGUEST-CHECKED-RELEASE-V1406.MD`. Bitmap-corruption
release refusal through scheduler termination remains an explicit separate
test gap. No v1406 production RAM growth or codec change.

## 2026-11-10 v1405: SWAP-MRES checked release and no false publication

The seven `backstore_free` calls in the movable PDP-6 process swap code
now check the complete-release return introduced in v1398. The most
important change is swap-in commit ordering: after restoring the saved
disk words and successfully unpinning the new MM extent, the process
releases its old BACKSTORE run **while the original DISK record and storage
mode are still authoritative**. Only a successful free allows publication
of the restored physical base, decrement of `proc_swap_blocks_used`,
replacement of the record with the executable backing identity, and the
DISK->RESIDENT mode transition. A refused release invokes the existing
post-unpin MM cleanup, retains the original disk owner and can be retried
without accounting/bitmap drift. An MM rollback refusal still enters the
existing fatal MM invariant.
This retry was demonstrated by the source-linked native caller, not by
normal scheduler dispatch. `proc_swap_mres_service_v481` still handles a
negative `proc_swap_in` result through `SYS_EVENT_TERM`; unlike a positive
`MM_ERR_NOMEM` pressure result, it does not automatically schedule a
cooldown/retry. Abnormal bitmap-refusal cleanup needs separate scheduler-
level acceptance.

Exchange accounting, reservation prepare/cancel, failed swap-out rollback,
and optional PURE verifier scratch release cannot all be rolled back once
their particular cleanup points are reached. They now check the allocator
result and enter the established storage invariant rather than claiming
that a disk allocation was released when it remains owned. No new flags,
bitmap, provider object or permanent KCORE state was introduced.

A new source-linked PDP-6 v1405 test executes the production swap-in,
the real BACKSTORE bitmap validator and the real free routine. An
incomplete bitmap causes rollback, without changing the original disk
record or accounting; after restoring the bitmap, the same process
successfully commits once. Removing the status check as an executable
negative control is rejected. Native v781/v782/v787 cover successful and
refused exchange cleanups; native swap-PURE and exact backing/data
integration remain PASS. The older integrated v812 fixture fails its own
assertion on both v1404 and v1405; it is not a new SWAP-MRES regression.

Measured 32K lowmem: fixed KCORE **9687** (unchanged), movable SWAP MRES
**1757 -> 1762 (+5) words**, combined KCORE+MRES **16362 -> 16367**,
single-root permanent placement **31 words over** instead of 26.
The strict 256K F07 full-guest postflight is a separate acceptance gate.

## 2026-11-10 v1398: existing-owner release after STORAGECTL disable

The historical `backstore_free(first, blocks)` void PDP-6 entry now reports
**0 on full release, -1 on refusal** through AC1; old machine-code callers
which discard the result retain their original calling convention. Both
the verifier and checked free use the same complete bitmap ownership scan.
The allocation-enable flag is checked only for allocating/extending new
BACKSTORE records, not for examining or releasing previously owned blocks.
Disabling new swapping must never strand an existing on-disk process image.

Resident `proc_swap_detach()` now relies on this **single checked release**
instead of prechecking via an allocation-enabled verifier and then calling
an unchecked void free. If the release refuses, no used-block decrement,
storage-mode publication or record erasure occurs: VM teardown retains the
existing owner for retry. Native PDP-6 v1393/v795/v777, host v880/v957,
normal and PURE-verify native swap, and bounded source-contract checks PASS.

A matched, clean 32K GCC lowmem comparison against committed v1397 yields
**9694 -> 9687 fixed KCORE words (-7)**. SWAP MRES remains 1757 words;
complete KCORE+MRES inventory 16369 -> 16362 words. The strict 16K
single-root permanent boundary is still 26 words over, down from 33.
This is a verified net memory saving; the earlier 9870-word checkpoint
had stale build artifacts and cannot be used to attribute the code change.

Remaining BACKSTORE work: propagate refusals through MEMFS backing-drop,
resize and unmount, and audit remaining SWAP MRES void-return consumers.
Complete live 256K physical swap/OOM matrix acceptance is still required.

## 2026-11-10 v1397: canonical new AUXSTORE v1 empty regions

Original `mkauxstore` v1 descriptors wrote the computed start address even
when BACKSTORE, LOGSTORE or CACHESTORE had a zero length. KINIT already
treated length zero as absent. Rejecting old descriptors for this vestigial
start would make previously formatted media unbootable, so the reader keeps
supporting both representations. The host `mkauxstore` formatter now writes
exact **0,0** for each absent pool, without changing the descriptor version
or the physical placement of nonempty regions. New and legacy descriptors
both pass v507 discovery tests; the actual host formatter's canonical
output and its destructive-write safeguards pass v636. AUXSTORE choice v492
and region bounds v657 PASS. No new resident state and no fixed KCORE growth.

## 2026-11-10 v1393: BACKSTORE allocated-owner release integrity

`backstore_free` now validates the complete run against the existing shared
bitmap **before changing any bit or subtracting from blocks-used**. Incomplete
or duplicate frees are rejected without changing other owners or accounting.
The check shares the allocation verifier's range and bitmap scan but does
not require BACKSTORE to be enabled for new allocations: `STORAGECTL` may
disable swapping while previously allocated runs still need legitimate
cleanup. No new resident state or per-owner table is introduced.

Real source-linked PDP-6 tests v1393, v777 and v795 pass, including a run
crossing a 36-bit bitmap-word boundary, disable/release, duplicate free,
invalid range and allocation-verifier refusal. AUXSTORE region v657,
BACKSTORE gateway v815 and backing publication v665 also pass. This is
a completed allocator correctness increment, **not** proof of complete
AUXSTORE/BACKSTORE feature integration: mixed-media consumers, provider
failures, persistent recovery and current full 256K pressure/OOM acceptance
remain to be completed.

# Root-profile boundary for unified TSFS

**2026-11-09 v1375:** The old compressed process disk-swap feature
is cancelled; ordinary raw swap remains available on configured
backing. A swapless-only option for discarding PURE text and
compressing mutable state *in managed RAM* does not create BACKSTORE
blocks or implicit disk service. Its implementation is not yet
active; see `SWAPLESS-MEMORY.MD`.

The planned minimal unified TSFS root (`ROOT=tape TAPE_AUX=none
TAPE_TMPFS=0`) has **no** process swapping, BACKSTORE, LOGSTORE,
backed MEMFS eviction or RAM-only temporary filesystem. These facilities
are excluded at build/link time, not conditionally enabled by probing a
device. An explicitly configured `TAPE_AUX=drum|disk` tape-root build may
instead provide backing, while its root filesystem remains read-only. The
policies and experiments below concern such configured auxiliary builds,
other roots, or historical implementation behavior. See `DECTAPE.MD`.

# v670 — D6FS mount-stage evidence

Native PDP-6 breakpoint tracing of the v669 scheduler-pressure boot
shows BLOCKSET discovery, repeated physical block reads, and entry to
`d6fs_super_select` before `?RT`. Neither `vfs_mount` nor `vfs_stat`
is reached. Testkit v670 reproduces that failure-stage classification
using linked KINIT symbols. The exact failing superblock or pre-VFS
check remains unproven. Full-system scheduler rollback is still blocked.

# v669 diagnostic refinement — meaning of ?RT

`?RT` is `KINIT_ERR_RT` in `kinit_boot()`: root mount failed or
`root_has_init()` did not find a regular `/SYSTEM/INIT`. The failure
occurs before scheduler pressure begins; it is not evidence that
BACKING rollback itself failed. Testkit v669 now has a root-media
preflight, but which of these two root checks failed remains unknown.

# v669 — full-system swap acceptance still blocked

Isolated native PDP-6 BACKING/MM rollback tests v664-v668 PASS, but the
historic full-system testkit v298 has a stale assumption that
`proc_swap_out` is an absolute KCORE symbol. It now resides in the
loadable SWAP MRES. The corrected testkit v669 runner reaches simulator
execution; without high-volume tracing, the guest prints `?RT` and
halts at `065206` before completion of the pressure workload. A
traced run timed out after repeated breakpoints. These are **not**
full-system rollback acceptance passes. Diagnose boot/startup before
resuming scheduler fault injection. No production code changed.

# v668 — verified recoverable MM-free rollback

An isolated native PDP-6 test now executes the production swap-out
late rollback branch with `mm_unpin` successful and `mm_free` failing.
It confirms the process remains resident, the original swap record is
unchanged, the temporary BACKING allocation is freed, and TRANSITION
and the provisional u-area backing field are cleared. Testkit v668
PASS. This is not a full scheduler-level injected fault or an assertion
of physical multi-block I/O atomicity. No production code changed.

# v667 — unsafe late swap-out unpin cleanup fixed

The v666 audit did not establish safe behavior when `mm_unpin` itself
failed: the prior rollback could clear TRANSITION and resume a process
with unresolved physical pin ownership. The PDP-6 swap-out assembly now
routes either failed unpin (normal postwrite path or error cleanup)
to the existing storage/MM invariant handler, rather than silently
resuming. A successful unpin followed by failed `mm_free` retains its
existing recoverable prepublication rollback.

Testkit v667 source and isolated native PDP-6 fault branches PASS,
the adjusted v666 regression PASSes, and a fresh GCC 32K disk-root
image boots through `proc_boot_init` with BSET/D6FS loaded. Full-system
fault/scheduler testing is not implied. This is a correctness fix,
not KCORE size optimization.

# v666 — swap late-memory-release rollback audit

The PDP-6 `proc_swap_out` implementation writes a provisional BACKING
span, then attempts `mm_unpin` and `mm_free`. Failure of either takes
the prepublication rollback path: release provisional BACKING, clear
the temporary u-area backing field and TRANSITION. Only successful
memory release leads to DISK mode publication. A new testkit v666
source control-flow regression PASSes, as does the earlier native v665
partial-write test. Live fault-injected MM unpin/free and scheduler
recovery remain unverified; no production change was required.

# v665 — multi-block BACKING failure contract

Native PDP-6 instruction test of the production BACKING transfer
routine confirms that a failing second block of a three-block write
returns an error and never writes the third. Separate source checks
confirm swap-out and MEMFS eviction publish their new backing records
only after successful writes, freeing abandoned allocations on the
write-error branch. Both testkit v665 checks PASS.

This is **failure atomicity of publication**, not physical atomicity
of multi-block storage: earlier blocks can already have been written.
Full-system in-flight controller faults, late MM rollback paths and
persistent snapshot recovery still need acceptance tests. No change
to production code, media layout or KCORE size optimization.

# v664 — isolated native BACKING failure coverage

Native PDP-6 execution verifies that an unavailable multi-device
VOLUME mapper cannot silently fall back to direct DEVICE addressing.
The direct-device path remains operational. This test uses the real
BACKING transfer assembly with instrumented device entry points;
production KCORE and AUXST1 formats are unchanged. Full-system
partial-transfer recovery remains pending.

# v663 — VOLUME compatibility evidence

The existing native multi-member D6FS suite PASSed on two/four-member
HDD volumes after the DEVICE/VOLUME split. Backing-mapper fault-injection
is still open; see KERNEL/BLOCKSET.MD. The testkit v663 launcher avoids
host relay binaries inherited from a different CPU architecture.

# v662 — BACKING runtime consumers

Process swap initialization and MonitorFS SWAP statistics both query
`backing_region_blocks()` for the logical capacity. The historical
`blockset_tail_blocks()` symbol remains a binary-compatible alias;
the linked map places both at octal `005071`. The internal shared
transfer entry is named `backing_region_io` while preserving the
existing `backstore_read/write` and legacy BLOCKSET entry labels.
The change retains a single DEVICE/VOLUME I/O path, no extra resident
state or dispatch layer, and unchanged AUXST1/BLOCKSET formats.

Testkit v662, v660, v659 PASS. The fresh GCC 32K image builds and
boots on PDP-6 SIMH to `proc_boot_init` with BSET/D6FS loaded; D6FS
checks CLEAN. Native multi-member mapper-loss injection remains open.

# v661 — LOG boot REGION routing

LOG bootstrap reads and writes now call the bounded AUXST1 REGION
operations directly. The intermediate `auxstore_boot_log_read/write`
wrappers have been deleted. The preexisting root-backed BLOCKSET LOG
fallback is retained. LOG is a durable record ring, BACKING owns the
temporary swap/MEMFS allocation bitmap, and CACHE contains only
discardable copies; these are independent policies sharing only
physical addressing. No persistent KCORE dispatch layer was added.

# v660 — DEVICE/VOLUME/REGION resident state names

The backing bridge retains its existing three resident configuration
words, now named `device_region_map`, `volume_data_blocks` and
`backing_region_count`. The historical `blockset_direct_*` names remain
compatibility aliases on the **same addresses**; the old tail-count
function is likewise aliased by `backing_region_blocks()`. Process swap
boot calls the latter. These are semantic source names only: no new
storage dispatch layer or persistent metadata format is introduced.

Source regressions v658-v660 PASS, fresh native GCC 32K disk build
PASS, D6FS checker CLEAN, and PDP-6 SIMH reaches `proc_boot_init` with
BSET/D6FS loaded. Multi-member mapper-loss fault injection remains
open. No KCORE size optimization is part of this change.

# v659 — VOLUME mapper-unavailable behavior

The permanent BACKING transfer service now fails closed when a
multi-device VOLUME is configured but its BLOCKSET mapper is absent.
Previously it fell through to direct DEVICE arithmetic, potentially
addressing the wrong sectors. A direct DEVICE range still bypasses
the VOLUME mapper. This changes neither AUXST1 media layout nor
BLOCKSET binary service ABI.

The v659 fail-closed source test and v658 routing test PASS. The
fresh native 32K GCC D6FS/disk image builds and boots through
`proc_boot_init` with BSET/D6FS loaded. Native mapper-loss fault
injection remains outstanding. No code-size optimization was run.

# v658 — separate direct DEVICE regions from VOLUME tail mapping

The permanent storage bridge now exposes `device_range_configure` for
single physical DSK/DRM regions and `volume_tail_configure` for
multi-member root tails translated by the existing BLOCKSET MRES.
`blockset_direct_configure` remains an ABI-compatible symbol alias.
Both I/O paths use one common physical DSK270/DRM236 dispatcher;
neither AUXST1 nor BLOCKSET on-media formats change.

The intended nomenclature is DEVICE (one physical drive), VOLUME
(optional multi-device block mapping), REGION (bounded block range),
BACKING (shared authoritative swap/MEMFS space), LOG (persistent records)
and CACHE (discardable copies). This terminology is a staged source
migration, not a new stack of generic runtime storage layers.

Fresh 32K lowmem GCC PDP-6 SIMH D6FS disk boot reached
`proc_boot_init` with BSET and D6FS loaded. Testkit direct/volume
routing inspection PASS. Multi-member and fault-path integration
remain open; DTFS/TSFS features are deferred for now.

# v657 — shared AUXSTORE raw-region I/O (first production feature increment)

Production AUXSTORE bootstrap exports `auxstore_boot_region_read` and
`auxstore_boot_region_write` for BACKSTORE, LOGSTORE and CACHESTORE,
using partition-relative block addresses. The region selector and
block offset must be valid; no request may escape into an adjacent
partition. LOGSTORE bootstrap read/write now uses this shared
interface. Existing raw BACKSTORE allocation and physical I/O are
preserved pending migration and rollback/recovery validation.

Host testkit v657 region bounds PASS; existing AUXSTORE choice v492
and D6FS overlap v507 PASS. This is feature implementation, not a
KCORE optimization, and does not close the remaining BSTORE migration.

The online D6FS packer and agreed storage/admin extensions are in
version 0.9 release scope. DTFS/TSFS features are temporarily deferred
while KCORE feature implementation proceeds.

### v484: BACKSTORE allocator resident split (2026-10-27)

The authoritative bitmap and four associated state words stay in KCORE,
shared by process swap and MEMFS. The allocator TEXT has moved into the
permanently installed SWAP/shared MRES. KCORE exposes the original four
`backstore_*` functions through one-word JRST entries patched by SWAP
MINIT. Neither the API nor the single-bitmap ownership changes; this
MRES is installed on the existing boot path and is not a removable
optional module while its clients are active.

On the PDP-6 full GCC lowmem link KCORE shrank 8652 -> 8548 words, and
shared SWAP MRES grew 308 -> 412 words, giving no change in combined
permanent occupancy (disk 13415, tape 13339 words). The corrected build
puts `kcore_end.dobj` last: backing state occupies 020620..020623,
`__kcore_low_end` and dynamically allocated MM extents begin at 020624.
The earlier experimental KINIT failure resulted from this boundary
overlap, not a need for another allocator or a change in storage ABI.
Native 32K swap-wait and swap-stress and the standalone PDP-6 BACKSTORE
fixture pass. The boundary regression lives in daimos-testkit.

### v481 KCORE reduction boundary (2026-10-27)

The process swap *transfer/reclaim* implementation now resides in a pinned
SWAP MRES. This does **not** transfer ownership of the common BACKSTORE
bitmap. `backstore.s` is still KCORE (108 pre-link TEXT, 4 BSS words):
MEMFS uses the same allocation/free interface and backing I/O paths.
Its migration must use a shared service and preserve one authoritative
bitmap; adding a second allocator or moving it behind an optional SWAP
package would violate this contract.

During KINIT, `proc_swap_boot_init` currently allocates one process swap
record per slot, the bitmap (`ceil(backstore_blocks/36)` words), and a
two-bit-per-slot process storage-state map together. Zero backing capacity
does not imply the executable-backing records are unnecessary. Consider
splitting their lifetimes only after validating all PROC, MEMFS and VM
consumers and measuring runtime MM occupation, not merely static KCORE.

# DAIMOS BSTORE / BCACHE DESIGN AND IMPLEMENTATION PLAN

STATUS: STORAGE CONSOLIDATION IMPLEMENTED; AUXSTORE MEDIA TOOLING/LAYOUT
UPDATED 2026-10-06.  THE MIGRATION PLAN BELOW IS RETAINED AS THE DESIGN
CHECKLIST, WITH REMAINING TEST/MAINTENANCE ITEMS CALLED OUT EXPLICITLY.

## 1. GOAL

DAIMOS SHOULD HAVE ONE SMALL 128-WORD BLOCK MECHANISM FOR THE STORAGE CASES
THAT ARE ACTUALLY COMMON, WITHOUT FORCING DIFFERENT PERSISTENCE SEMANTICS INTO
ONE OVERGENERAL INTERFACE.

THE COMMON UNIT IS ONE OCTAL 0200-WORD BLOCK.  D6FS, DTFS/TSFS, DSK270,
DRM236, PROCESS SWAP, MEMFS EVICTION, AND LOGSTORE ALREADY USE THAT SIZE.
NO VARIABLE BLOCK SIZE, BYTE PACKING, OR CONVERSION LAYER IS REQUIRED.

THE TARGET HAS THREE SMALL PARTS:

```TEXT
BACKSTORE   AUTHORITATIVE ALLOCATED BLOCKS FOR PROCESS SWAP AND MEMFS
BCACHE      CLEAN/DISCARDABLE BLOCK COPIES FOR FILESYSTEM READ ACCELERATION
AUXSTORE    HARDWARE-INDEPENDENT DSK/DRM PLACEMENT FOR BACKSTORE, CACHESTORE,
            AND LOGSTORE REGIONS
```

LOGSTORE KEEPS ITS OWN RING/RECOVERY SEMANTICS.  IT USES AUXSTORE I/O BUT IS
NOT AN ALLOCATED BACKSTORE OBJECT AND IS NOT CACHED AUTOMATICALLY.

## 2. LAYERING

```TEXT
                     FILESYSTEMS

            D6FS       DTFS       TSFS
              \          |          /
               \         |         /
                +------ BCACHE ----+
                       /     \
                  RAM L1     CACHESTORE L2
                                |
                             AUXSTORE
                                |
                            DRM / DSK

                     MEMORY BACKING

            PROCESS SWAP       MEMFS
                  \             /
                   \           /
                    BACKSTORE
                        |
                     AUXSTORE
                        |
                     DRM / DSK

                       LOGGING

                     LOGSTORE
                        |
                     AUXSTORE
                        |
                     DRM / DSK
```

BCACHE IS OPT-IN.  RAW PROCESS SWAP AND LOGSTORE I/O MUST NOT PASS THROUGH THE
CACHE.  FILESYSTEMS CHOOSE TO USE BCACHE AROUND THEIR NORMAL BLOCK READ/WRITE
PATHS.

## 3. BACKSTORE

BACKSTORE REPLACES THE SWAP-SPECIFIC ALLOCATOR WITH A GENERAL RAW-BLOCK
ALLOCATOR.  IT OWNS EXACTLY ONE BITMAP, ONE BIT PER 0200-WORD BLOCK.

THE REQUIRED INTERFACE IS CONCEPTUALLY:

```TEXT
BACKSTORE_ALLOC(BLOCKS, RESERVE, &FIRST)
BACKSTORE_FREE(FIRST, BLOCKS)
BACKSTORE_READ(FIRST, BLOCKS, BUFFER)
BACKSTORE_WRITE(FIRST, BLOCKS, BUFFER)
```

`RESERVE` IS THE NUMBER OF FREE BLOCKS THAT MUST REMAIN AFTER AN ALLOCATION.
PROCESS SWAP USES ZERO.  MEMFS USES THE PROCESS-SWAP RESERVE.  THIS REPLACES
THE SPECIAL `SWAP_STORE_ALLOC_MEMFS()` POLICY WITHOUT A SECOND ALLOCATOR.

PROCESS SWAP AND MEMFS KEEP THEIR OWN MAPPING RECORDS ABOVE BACKSTORE.  BSTORE
DOES NOT KNOW WHETHER A RUN BELONGS TO A PROCESS, FILE, OR SNAPSHOT.

PERSISTENT MEMFS MAY RESERVE ITS EXISTING VERSIONED SNAPSHOT EXTENT AT BOOT.
THE GENERAL BACKSTORE ALLOCATION BITMAP DOES NOT BECOME A PERSISTENT ON-MEDIA
ALLOCATOR.

## 4. BCACHE

BCACHE IS CLEAN-ONLY AND WRITE-THROUGH.  IT NEVER OWNS THE ONLY VALID COPY OF
DATA.  THERE IS NO DIRTY LIST, WRITEBACK DAEMON, FLUSH SCHEDULER, OR CACHE
JOURNAL.

READ:

```TEXT
LOOK UP CLEAN COPY
  HIT  -> RETURN IT
  MISS -> CALLER READS AUTHORITATIVE DEVICE, THEN INSERTS COPY
```

WRITE:

```TEXT
CALLER WRITES AUTHORITATIVE DEVICE FIRST
  FAILURE -> CACHE IS NOT UPDATED
  SUCCESS -> UPDATE/REPLACE CLEAN CACHE COPY
```

THE INITIAL RAM L1 IS ONE CLEAN TWO-BLOCK CACHE SHARED BY D6FS AND DTFS/TSFS.
IT REUSES THE ALREADY-PERMANENT SERIALIZED FILESYSTEM TRANSFER WORKSPACE AS
SLOT 0 RATHER THAN ALLOCATING TWO ADDITIONAL FULL DATA BLOCKS.

CURRENT L1 SHAPE:

```TEXT
KCORE:
    1 WORD         WORKSPACE TAG
    0200 WORDS     FS_BLOCK_WORKSPACE / CACHE SLOT 0

MANAGED CORE:
    WORD 0         PRIVATE-SLOT TAG
    WORDS 1..0200  CACHE SLOT 1
```

THE RECLAIMABLE MANAGED-CORE SLAB IS THEREFORE OCTAL 0201 WORDS (129
DECIMAL), DOWN FROM THE FORMER OCTAL 0403-WORD PRIVATE SLAB.  THE ONE
ADDITIONAL FIXED TAG WORD IS MORE THAN OFFSET BY ELIMINATING ONE DYNAMIC
128-WORD DATA BLOCK.  INCLUDING THE IMPLEMENTATION/CALL-SITE CODE GROWTH,
THE CHANGE SAVES 109 PHYSICAL RAM WORDS WHILE THE CACHE IS ACTIVE.

THE TWO ENTRIES REMAIN FULLY ASSOCIATIVE.  A WORKSPACE-BACKED MISS PRESERVES
THE VALID WORKSPACE BLOCK INTO THE PRIVATE SLOT BEFORE THE CALLER REFILLS THE
WORKSPACE.  A PRIVATE-SLOT HIT REQUESTED INTO THE WORKSPACE PROMOTES THAT KEY
TO SLOT 0.  THERE IS NO VICTIM-HAND WORD, LRU LIST, HASH CHAIN, REFERENCE
COUNT, OR DIRTY STATE.

DIRECT USERS OF FS_BLOCK_WORKSPACE MUST INVALIDATE THE WORKSPACE TAG BEFORE
MODIFYING SCRATCH CONTENTS.  THE COMMON ZERO-WORKSPACE HELPER DOES THIS
AUTOMATICALLY.  D6FS INVALIDATES BEFORE WRITE-THROUGH AND RE-TAGS ONLY AFTER
THE AUTHORITATIVE WRITE SUCCEEDS, SO A FAILED WRITE CANNOT LEAVE MODIFIED
WORKSPACE DATA MARKED CLEAN.

## 5. CACHE KEYS

ONE 36-BIT WORD IDENTIFIES A CLEAN BLOCK COPY.  THE EXACT BIT PACKING MAY BE
TUNED DURING IMPLEMENTATION, BUT IT MUST REPRESENT:

```TEXT
SOURCE / MOUNT IDENTITY
MEDIA OR MOUNT GENERATION
LOGICAL BLOCK NUMBER
```

REMOVABLE MEDIA MUST CHANGE GENERATION WHEN A NEW MEDIUM IS ACCEPTED.  OLD DTC
CACHE ENTRIES THEN BECOME UNREACHABLE WITHOUT A CACHE SCAN.

D6FS USES A STABLE MOUNT/BACKING IDENTITY PLUS LOGICAL BLOCK NUMBER.  DTFS AND
TSFS USE THE DTC UNIT/MEDIA GENERATION PLUS PHYSICAL TAPE BLOCK NUMBER.

## 6. OPTIONAL CACHESTORE L2

CACHESTORE IS A FIXED AUXSTORE REGION USED ONLY WHEN THE AUTHORITATIVE SOURCE
IS MATERIALLY SLOWER THAN AUXSTORE.  THE PRIMARY V0.9 USE IS DECTAPE CACHED ON
A DRUM OR, SECONDARILY, A FASTER DISK.

NORMAL D6FS-ON-DISK MUST NOT COPY DISK BLOCKS INTO ANOTHER REGION ON THE SAME
DISK BY DEFAULT.  D6FS STILL USES RAM L1.

CACHESTORE IS DIRECT-MAPPED.  USE A POWER-OF-TWO NUMBER OF DATA SLOTS:

```TEXT
128, 256, 512, 1024, 2048, ...
```

THE SLOT IS:

```TEXT
HASH(KEY) & (SLOTS - 1)
```

ONE 128-WORD TAG BLOCK HOLDS 128 TAG WORDS.  TAGS LIVE ON AUXSTORE, NOT IN
RESIDENT RAM.  CACHESTORE REQUIRES NO ALLOCATION BITMAP.

CACHESTORE IS DISPOSABLE ACROSS BOOT.  KINIT MAY INVALIDATE IT BY CLEARING THE
TAG BLOCKS.  DO NOT ADD PERSISTENT EPOCH/JOURNAL STATE UNLESS MEASUREMENT SHOWS
THAT BOOT-TIME TAG CLEARING IS A REAL PROBLEM.

## 7. AUXSTORE REGIONS

NEW NON-FILESYSTEM AUXILIARY MEDIA SHOULD USE A SMALL DEVICE-INDEPENDENT
DESCRIPTOR WITH FIXED REGIONS:

```TEXT
AUXSTORE DESCRIPTOR
LOGSTORE
BACKSTORE
CACHESTORE TAGS
CACHESTORE DATA
```

THE DESCRIPTOR RECORDS START/COUNT FOR LOGSTORE, BACKSTORE, AND CACHESTORE.
AS OF V638, HOST `MKAUXSTORE` REQUIRES EXPLICIT DESTRUCTIVE CONFIRMATION;
DRY-RUN AND HOST SAFETY TESTS REFUSE UNKNOWN OR NONEMPTY TARGET REGIONS.
THE NORMAL PDP-6 BUILD CONVERTS ONLY AN EXACTLY VERIFIED ONE-MEMBER CLEAN
DBOOT HEADER TO AUXST1 VIA EXPLICIT `--CONVERT-DBOOT`, CHECKING THE D6FS
BASE/COUNT, SUPERBLOCK OFFSETS, AND RESERVED TAIL BEFORE WRITING. THE
NON-ROOT D6FS PARTITION IS PRESERVED BYTE FOR BYTE. THESE ARE IMAGE-BUILD
OPERATIONS, NOT AUTOMATIC FORMATTING AT DAIMOS BOOT.
THE CACHE TAG/DATA SPLIT IS DERIVED FROM CACHESTORE SIZE.  AUXSTORE PREFERS DRM
WHEN PRESENT, THEN DSK.  IT DOES NOT REQUIRE D6FS ON THE AUXILIARY DEVICE.

THE NORMAL PDP-6 BUILD FORMATS DRM0 AS A CO-RESIDENT AUXILIARY MEDIUM RATHER
THAN AS A SECOND ROOT SET.  PHYSICAL BLOCK 0 HOLDS THE AUXSTORE DESCRIPTOR.
THE DESCRIPTOR'S OPTIONAL WORDS 5 AND 6 RECORD A NON-ROOT D6FS BASE/COUNT AND
ITS TWO LOGICAL SUPERBLOCK LOCATIONS.  THE D6FS REGION IS INDEPENDENT OF THE
AUXSTORE REGIONS; IT CONTAINS NO BOOT STREAM AND OWNS NO SWAP OR LOGSTORE
RESERVATION.  THE REMAINING PHYSICAL TAIL IS PARTITIONED AS LOGSTORE,
BACKSTORE, THEN CACHESTORE.  `MKAUXSTORE` IN DAIMOS `TOOLS/HOST` FORMATS AND INSPECTS
THIS DESCRIPTOR.  THE NORMAL DEFAULT GEOMETRY IS:

```TEXT
BLOCK 0             AUXSTORE DESCRIPTOR
1..5367             NON-ROOT D6FS, 05367 BLOCKS
5370..5767          LOGSTORE, 0400 BLOCKS
5770..15767         BACKSTORE, 010000 BLOCKS
15770..17777        CACHESTORE, 02010 BLOCKS
```

DRM0 THEREFORE USES THE FULL TYPE-236 020000-BLOCK CAPACITY.  DRM1-3 ARE
BLANK IN THE NORMAL BOOT PROFILE.  AN EXPLICIT DRM BOOT CONTINUES TO USE THE
SEPARATE BOOTABLE DRUM-SET PATH.

THE 2026-10-31 V852 DEFAULT INCREASES BACKSTORE FROM 04000 TO 010000
SECTORS WITHOUT CHANGING THE TOTAL DRUM GEOMETRY OR CACHESTORE SIZE.
THE NON-ROOT D6FS AUXILIARY REGION IS CORRESPONDINGLY SMALLER.  THE
FORMER 04000-SECTOR DEFAULT DID NOT ADMIT THE LAST CHILD OF THE FULL
16-CHILD PDP-6 SWAP-PRESSURE WORKLOAD (EXEC_LOAD_NOMEM).  THE LARGER
BACKSTORE CONFIGURATION ALLOWED ALL 17 RUN CALLS IN V851.  THE SCHEDULER
ALSO REQUIRES A ONE-SHOT DISPATCH HINT AFTER SUCCESSFUL SWAP-IN TO KEEP
THE RUNNABLE RESTORED PROCESS FROM BEING SKIPPED IN FAVOR OF THE NEXT
SWAPPED CANDIDATE.  V851 PASSED THE 16-CHILD FULL-SYSTEM REGRESSION
WITH THE LARGER CAPACITY.  THE DEFAULT-CONFIGURATION V852 REGRESSION
MUST BE VERIFIED SEPARATELY.

OLD D6FS/BLOCKSET SWAP/LOGSTORE RESERVATIONS MAY REMAIN READ-COMPATIBLE DURING
MIGRATION, BUT NEWLY FORMATTED AUXILIARY MEDIA SHOULD PREFER AUXSTORE.

## 8. MEMORY-PRESSURE ORDER

THE TARGET CHEAPEST-FIRST RECLAIM ORDER IS:

```TEXT
1. DISCARD THE CLEAN BCACHE RAM L1 SLAB.
2. EVICT ELIGIBLE MEMFS DATA INTO BACKSTORE.
3. COMPACT CORE IF FRAGMENTATION, NOT FREE WORD COUNT, BLOCKS ALLOCATION.
4. SWAP AN ELIGIBLE PROCESS INTO BACKSTORE.
```

THERE IS NO DIRTY GENERIC CACHE RECLAIM CASE BECAUSE BCACHE NEVER HOLDS DIRTY
DATA.

## 9. EXPLICIT NON-GOALS

V0.9 BSTORE/BCACHE DOES NOT PROVIDE:

- VARIABLE-SIZE CACHE OBJECTS;
- BYTE/CHARACTER CACHE UNITS;
- GENERAL WRITE-BACK CACHING;
- AUTOMATIC CACHING OF EVERY BLOCK-DEVICE REQUEST;
- A PERSISTENT GENERAL-PURPOSE BLOCK ALLOCATOR;
- CACHE LRU CHAINS OR HASH TABLES;
- PROCESS SWAP THROUGH BCACHE;
- LOGSTORE THROUGH BCACHE;
- DYNAMIC SHARING OF CACHESTORE BLOCKS WITH AUTHORITATIVE BACKSTORE.

THE LAST RULE IS INTENTIONAL.  SHARING THE SAME PHYSICAL SLOTS WOULD REQUIRE
EXTRA VALIDITY/GENERATION OR INVALIDATION STATE TO PREVENT AUTHORITATIVE DATA
FROM COLLIDING WITH STALE CACHE TAGS.  FIXED BACKSTORE/CACHESTORE REGIONS ARE
SMALLER AND EASIER TO PROVE CORRECT.

## 10. IMPLEMENTATION PLAN

THE MIGRATION MUST BE CUMULATIVE AND KEEP A BUILDABLE TREE AT EACH COMMITTED
STAGE.

### STAGE 1 - FREEZE THE ABI AND MEDIA LAYOUT

1. DEFINE `BSTORE_BLOCK_WORDS` AS OCTAL 0200 AND ADD BUILD-TIME ASSERTIONS FOR
   D6FS, DTFS, DSK270, AND DRM236 BLOCK SIZE EQUALITY.
2. REPLACE THE IN-PROGRESS AUXCACHE MEDIA SEMANTICS WITH AUXSTORE REGIONS NAMED
   LOGSTORE, BACKSTORE, AND CACHESTORE.
3. DEFINE THE COMPACT AUXSTORE DESCRIPTOR AND VALIDATE RANGE BOUNDS/OVERLAP IN
   KINIT.
4. KEEP LEGACY D6FS/BLOCKSET RESERVATION DISCOVERY AS A FALLBACK ONLY.

ACCEPTANCE: DSK/D6FS, DRM/D6FS, AND DTC/TSFS BOOT STILL LINK; NO NEW PERMANENT
MEMORY ABOVE THE CONFIGURED LIMIT.

### STAGE 2 - GENERALIZE SWAP_STORE INTO BACKSTORE

1. RENAME/REWORK `SWAP_STORE` AS THE GENERIC ONE-BIT-PER-BLOCK ALLOCATOR.
2. REPLACE `SWAP_STORE_ALLOC_MEMFS()` WITH THE RESERVE PARAMETER.
3. ROUTE PROCESS SWAP THROUGH BACKSTORE WITHOUT CHANGING PROCESS SWAP RECORDS.
4. ROUTE MEMFS EVICTION/FAULT-IN THROUGH THE SAME BACKSTORE API.
5. RETAIN PERSISTENT MEMFS SNAPSHOT RESERVATION USING THE SAME BITMAP.

ACCEPTANCE: PROCESS SWAP AND VOLATILE/PERSISTENT MEMFS TESTS PASS; ONE ALLOCATOR
BITMAP EXISTS; NO SECOND MEMFS BACKING BITMAP EXISTS.

### STAGE 3 - GENERIC RAM BCACHE

1. EXTRACT THE CURRENT D6FS TWO-BLOCK CLEAN CACHE INTO `BCACHE`.
2. REDUCE THE SLAB TO THE MINIMUM GENERIC TAG+DATA SHAPE.
3. REPLACE D6FS `D6FS_CACHE_FETCH/STORE/RECLAIM` WITH BCACHE GET/PUT/RECLAIM.
4. KEEP D6FS WRITE-THROUGH ORDERING: AUTHORITATIVE WRITE BEFORE CACHE UPDATE.

ACCEPTANCE: D6FS CACHE TESTS PASS; D6FS PRIVATE CACHE STATE/CODE IS GONE;
DYNAMIC RAM IS NO LARGER THAN THE OLD OCTAL-0405 SLAB.

### STAGE 4 - DTFS/TSFS USE THE SAME L1

1. GIVE DTC MEDIA A COMPACT CACHE KEY WITH MEDIA GENERATION.
2. REMOVE THE SEPARATE DTC RAM-CACHE IDENTITY/ALLOCATION PATH.
3. CALL BCACHE AROUND THE EXISTING RAW DTC READ/WRITE SERVICES.
4. VERIFY NATIVE DTFS AND TSFS USE THE SAME PATH; TENEX/ITS ARE DISABLED.

ACCEPTANCE: DTC WITHOUT AUXSTORE USES ONLY THE COMMON TWO-BLOCK RAM L1 AND
REMAINS CORRECT ACROSS MEDIA CHANGES.

### STAGE 5 - CACHESTORE L2

1. ADD DIRECT-MAPPED CACHESTORE TAG/DATA ACCESS UNDER BCACHE.
2. ENABLE L2 ONLY FOR SOURCES WHOSE POLICY REQUESTS IT; DTC ENABLES IT WHEN
   AUXSTORE CACHESTORE EXISTS.
3. CLEAR CACHESTORE TAG BLOCKS AT BOOT.
4. KEEP DTC WRITE-THROUGH: TAPE SUCCESS FIRST, THEN BEST-EFFORT L1/L2 UPDATE.

ACCEPTANCE: DTC CACHE HIT/MISS/REPLACEMENT TESTS PASS; MEMFS/BACKSTORE CANNOT
ALIAS CACHESTORE; A DTC ROOT WORKS WITH AND WITHOUT AUXSTORE.

### STAGE 6 - REMOVE LEGACY DUPLICATION

1. DELETE OBSOLETE `AUXCACHE` PIN/VALID BITMAP CODE.
2. DELETE D6FS-SPECIFIC CACHE IMPLEMENTATION.
3. REMOVE SWAP-SPECIFIC ALLOCATOR NAMES AND SPECIAL MEMFS ALLOCATION ENTRY.
4. REMOVE DTC-SPECIFIC CACHE ALLOCATION/GEOMETRY STATE THAT BCACHE REPLACES.
5. AUDIT COMMENTS, SYMBOLS, AND TODO ENTRIES FOR STALE D6FS-OWNED SWAP/CACHE
   LANGUAGE.

ACCEPTANCE: ONE BACKSTORE ALLOCATOR, ONE BCACHE IMPLEMENTATION, ONE AUXSTORE
LAYOUT, AND NO DUPLICATE CACHE/BACKING BITMAPS REMAIN.

### STAGE 7 - TOOLS AND TESTS

1. DONE: ADD THE AUXSTORE FORMAT/INSPECTION TOOL TO DAIMOS `TOOLS/HOST`.
2. ADD HOST TESTS FOR DESCRIPTOR VALIDATION, BACKSTORE FIRST-FIT/RESERVE/FREE,
   CACHE KEYING, AND DIRECT-MAPPED CACHESTORE REPLACEMENT.
3. ADD TARGET/SIMH TESTS FOR PROCESS SWAP, MEMFS EVICTION, D6FS L1, DTFS/TSFS
   L1, AND DTC+AUXSTORE L2.
4. TEST DSK ROOT + AUX DRM, DRM ROOT, DTC ROOT + DRM AUX, DTC ROOT WITHOUT AUX,
   AND NO-BACKSTORE RAM-ONLY MEMFS.
5. RUN THE FULL PERMANENT-SIZE AND LOGIN/DSH ACCEPTANCE GATES.

ACCEPTANCE: ALL TESTS PASS AND THE PERMANENT ADDRESS LIMIT IS NOT EXCEEDED.

## 11. SIZE AND PERFORMANCE RULES

THE IMPLEMENTATION IS ACCEPTABLE ONLY IF IT CONSOLIDATES CODE OR PROVIDES A
MEASURED REQUIRED CAPABILITY WITHOUT VIOLATING THE PERMANENT-MEMORY LIMIT.

PRIORITIES ARE:

1. MINIMUM RESIDENT/PINNED RAM;
2. EXECUTION SPEED;
3. SIMPLE, PROVABLE STATE;
4. ON-MEDIA SPACE.

DO NOT ADD A GENERAL FRAMEWORK FOR A FUTURE DEVICE OR POLICY UNTIL A REAL
CONSUMER REQUIRES IT.
