# DAIMOS KCORE feature state, v1502

Date: 2026-11-11. This is the current **feature-completion** index for
KCORE, checked against committed production sources, testkit acceptance
and the historical records in `KERNEL/BSTORE.MD` and
`STATUS/KCORE-RELEASE-AUDIT-V1488.MD`. It is not a new release or a
kernel patch. DAIMOS is still version 0.5; release 0.9 is not claimed.

Evidence baseline: DAIMOS `80434835` (v1500), daimos-testkit `7f415a8`
(v1501), pdp10-doc `a94e246` (v1500), plus contemporaneous testkit
`6c3de2c` (v1502) correcting the physical branch observer. Native
v1498, v1500, v1501 and the v1502 positive/negative PDP-6 tests were
re-run successfully during reconciliation. The initial v1499 physical
guest did run and captured two independent pre/post TERM snapshots,
but its optional branch counters used inverted `SKIPN` polarity and
cannot be used for acceptance. The corrected runner has been prepared,
**not** re-qualified with a completed corrected physical guest.
Separate uncommitted v1492 FSINFO/mount changes are not production
acceptance and must not be described as merged functionality.

## Status of the major installed and proposed features

| Feature | Current status | Acceptance boundary |
| --- | --- | --- |
| Process EXEC/RUN/WAIT, native scheduler and ordinary disk/drum swap | Installed; normal profiles accepted | 256K F07 v1466 and earlier process/native regressions |
| Requester-only BRK OOM | Accepted in the defined 256K profile | v1298 physical 64/64 matrix |
| Grouped D6 block codec | Production-linked, accepted | v1464 cutover, KCORE/EXEC/PURE and image producers |
| Root-owned restricted MEMFS XIP and snapshot rollback | Accepted for specified 128K/192K/256K guests | v1469-v1485 physical tests, no general write access |
| Non-XIP low-memory boot | 48K physical accepted; 32K on HOLD | v1486, no MEMFS below 96K |
| BCACHE/DTC write and refused-MM invalidation | Corrected in production; native accepted | v1490; physical tape-root and mixed-media still OPEN |
| Checked BACKSTORE swap-in refusal | Real code native accepted, physical failure case OPEN | v1405/v1493; v1495 diagnostic is **not** fault acceptance |
| Foreign and deferred TERM/finalization of DISK owner | Native real-source boundaries accepted through v1501 | v1498, v1500, v1501; controlled VM/restore backends, not complete physical chain |
| Physical corrupt-backing/TERM owner resolution | OPEN | v1499 physical snapshots exist but event-branch counters were invalid; corrected physical rerun and permanent fault remain unaccepted |
| Device, VOLUME, BCACHE and LOGSTORE mixed failures | OPEN for distinct untested cases | Retain accepted v1484/v1485 snapshots and v1490 native correction |
| Swapless PURE-text drop, full APR reconstruction | OPEN; isolated prototypes only | v1381-v1397 not on production link path |
| Dynamic mount capacity enumeration | Core mount slab installed; consumer/ABI update WIP | v1492 changes are uncommitted; no accepted >4 mount enumeration |
| Optional mutable-only RAM compression | Deferred, conditional | v1451 test-only bitmap; requires full uncompressed PURE and net-positive RAM |

## Precise BACKSTORE/TERM evidence, v1493-v1502

- **v1493**: real `proc_swap_in`, checked bitmap release and slot-0
  service preserve the original DISK owner on a refused free, roll back
  new MM, and request TERM rather than READY_ONCE. Event is an observer
  stub. Two negative controls PASS.
- **v1494**: real `proc_finish_slot` and idle `proc_finish_service_v881`
  retry a controlled refused VM teardown and subsequent u-area MM
  refusals without double HUP/VM release. The VM backend is controlled.
- **v1495 physical / v1497 audit**: a 256K KCC PDP-6 guest reached
  HALT, with one injected bitmap fault, three matching backing-release
  calls, one repair, two failed swap-in services, two real TERM calls,
  and final zero process-swap/BACKSTORE blocks. Neither TERM immediately
  published PREVM; the third free had no PREVM marker. Therefore it
  does **not** prove idle-finalizer recovery and remains FAILED as a
  fault-specific acceptance. Independent MM/owner/slot checks PASS.
- **v1496**: fixed host SIMH banner `+uncommitted-changes` being counted
  as a child acknowledgment. Exactly 18 real guest acknowledgments
  and 18 reaps remain mandatory, with executable parser negatives.
- **v1498**: unchanged real `proc_event_apply` on a native PDP-6
  defers TERM for WAIT or saved executive PC, coalesces pending TERM,
  and preserves the DISK owner on refused finalization. Saved-PC
  protection mutation is rejected.
- **v1499**: separate full-guest diagnostic captures before/after
  event descriptors, saved PC, WAIT/FD-control bits, return result,
  PREVM marker and intended branch counters. Its initial physical
  run produced two independent before/after snapshots, but the
  original `SKIPN`-guarded branch counters incremented *outside*
  the selected event. These counts are invalid for classification.
  A corrected guest has not yet received physical acceptance.
- **v1500**: unchanged real `proc_event_apply`, `proc_finish_slot` and
  `proc_finish_service_v881` run *together* in one native PDP-6 test.
  Controlled first VM refusal retains TRANSITION+FINALIZE+PREVM and
  the original DISK owner. Idle VM and u-area retries publish exactly
  one zombie, without duplicated close/HUP/VM effects. Missing-PREVM
  mutation is rejected. The physical VM/BACKSTORE backend is controlled.
- **v1501**: unchanged real `proc_event_apply` plus
  `proc_deferred_term_check_v926` source-link in one native run.
  Foreign TERM on a WAIT_CHILD/executive-PC DISK owner is deferred and
  coalesced; after **controlled** swap restoration, owning syscall
  return consumes TERM once and hands off to a controlled private
  fatal-close observer. A mutation disabling pending-bit consumption
  is rejected. This verifies the owner-return seam, not the actual
  `proc_fatal_close_current_v879`/physical VM finalization afterward.
- **v1502**: the diagnostic injector now uses `SKIPE` for an active
  event. Testkit `6c3de2c` executes the extracted guard on PDP-6
  through inactive/active/inactive states and rejects the historical
  inverted `SKIPN` negative mutation. This corrects *test-only*
  instrumentation; it does not modify production KCORE or validate
  the old erroneous counts. Fresh corrected physical evidence remains
  necessary to identify the actual TERM decision branch.

Native immediate-foreign and deferred-owner delivery are both
supported, but the physical v1495 run's exact branch remains unknown.
The original v1336 normal-swap acceptance must continue to require
zero fatal swap-in failures. A separate, strict fault-specific guest
verifier is needed; do not weaken or silently redefine the normal
F07 matrix.

## Open KCORE feature tasks, in dependency order

1. **F2a, physical TERM classification**: rerun corrected v1499/v1502
   instrumentation in a full 256K HALT/EXIT guest; validate its
   scoped branch counters against independent pre/post owner snapshots,
   then identify the exact busy, WAIT, saved-PC or pending-fatal
   predicate and resulting event return/ownership. The old branch
   counter values may not be used.
2. **F2b, BACKSTORE irreparable fault policy**: exercise permanently
   corrupt bitmap/media refusal without orphaning the DISK record,
   false RESIDENT/READY_ONCE, unsafe retry loops or destructive foreign
   stack cleanup. Prove deferred owner return or legitimately safe
   PREVM idle finalization through actual resource ownership.
3. **F2c, full physical fault acceptance**: cover the actual scheduler,
   TERM, owner-safe teardown, finalizer and checked backing release
   end-to-end with exact block/VM/FILE/RT/PI accounting; retain
   v1336's unaffected normal-mode strict pass criteria.
4. **F2d, other storage integration**: device/volume mapper loss,
   in-flight controller errors, clean BCACHE tag invalidation on media
   identity changes, both D6FS/DTFS consumers, DSK/DRM/DTC root and
   AUXSTORE combinations, and MEMFS LOGSTORE/backing/swap coexistence.
   Real physical installed-provider tape-root DTFS remains open.
5. **F1, swapless PURE-text reclamation**: integrate immutable backing,
   inactive-owner selection, MM pin/rebase transaction, mutable suffix,
   complete APR restore, and every EXEC/EXIT/WAIT/provider failure path;
   then test native and physical RAM pressure. Existing v1381-v1397
   isolated modules are NOT installed; v1487 shared-DXR proposal adds
   180 TEXT words and is not a shrink candidate.
6. **F3, dynamic mount enumeration**: complete and commit the v1492
   FSINFO capacity ABI and command consumers, or reconcile their
   contract if it changes; validate small inventory, holes, error
   propagation, slots above four and boundary 16/26. KINIT's
   published dynamic mount slab must not be replaced with a fixed
   four-slot table.
7. **F4, optional mutable-RAM compression**: decide only after F1
   passes and prove positive *whole-system* RAM savings, including
   temporary buffers, encoder, decoder and APR restore headroom.
8. **F5, remaining targeted physical integration**: close verified
   guest-only device/clock/scheduler profiles as needed; specifically
   distinguish previously accepted software-clock and RAM profiles
   from unaccepted physical 50-Hz or mixed-device configurations.

Out of this KCORE feature work: standalone MRES development; 32K
tests (explicitly HOLD); general userland features; the separate 0.9
online D6FS packer and unified TSFS; and size-only optimization. The
online packer remains a distinct 0.9 requirement; unified TSFS is
scheduled after final KCORE optimization and before 0.9. Runtime
MRES module code unload/reload stays POST-1.0. Do not begin KCORE
size-only optimization until functional feature freeze, and measure
matched builds rather than incomparable code/guest RAM profiles.

The dated v1488 audit, v1497 physical report and earlier tests remain
historical evidence; this v1502 index supersedes them **only on the
current feature-status question**, not on their measurements.
