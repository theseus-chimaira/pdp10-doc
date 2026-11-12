# DAIMOS KCORE current feature state v1521

Date: 2026-11-12. Source/documentation reconciliation, **not a new
release**. Current committed DAIMOS baseline `65ed0b5b` (v1520),
VERSION 0.5. Detailed primary source audit:
`DAIMOS/design/kcore-feature-source-audit-v1521.md`; synchronized
active source-tree checklists: `DAIMOS/KCORE.MD` and `DAIMOS/TODO.MD`.
This document supersedes earlier dated status files **only for the
current OPEN checklist**. Preserve their historical tests and metrics.

## Production versus local experimental work

Production membership was checked against
`DAIMOS/system/kernel/Makefile` and live assembly, C and KINIT
call sites. No obsolete versioned standalone experiment was
declared installed just because its source exists. Fresh
`make -C system/kernel kcore COMPILER=kcc` completed the source-
to-assembler target successfully. This is not a full link,
actual PDP-6 guest or renewed acceptance of earlier test profiles.

The **local, uncommitted** v1492 FSINFO changes in
`fs/vfs.h`, `proc/syscall.h`, `proc/syscall_dispatch_pdp6.s`
and `userland/exec/commands.c` are separately identified:
AC2=-1 queries boot-published mount capacity and MOUNTS/DF
scan that count. Fresh exact-source native PDP-6 test PASSES
capacity result and rejects an unsafe query mutation; actual
command bodies compiled/executed against 1/4/8/26 mount counts
also PASS. The three testkit files remain untracked locally.
**This does not yet close F3:** no committed full guest and
multi-mount provider/error/holes acceptance was established.
Do not overwrite this in-progress source or treat it as merged.

## Installed and accepted, not current tasks

| Feature | Bound of acceptance |
| --- | --- |
| Normal EXEC/RUN/WAIT, raw disk/drum swap, BRK requester-only OOM | v1298 defined 256K 64/64 and v1466 full F07. |
| Canonical grouped block executable/filesystem codec | v1464/v1466 source cutover and 256K full guest; old D6LZ36 migration closed. |
| Restricted privileged nonwriting MEMFS raw XIP and snapshot rollback | v1469-v1485 physical 128/192/256K; no 32K/48K XIP, minimum MEMFS 96K. |
| BACKSTORE failure handling | v1506 repaired TERM/owner return; v1507 bounded retry; v1509 resident and v1510 healthy swapped progress; v1511 genuine Type-236 PI2 error and v1512 PI6 missing-completion timeout. Irreparable run remains owned, not magically reclaimed. |
| Non-root D6FS returning same media | v1513 native read error/return and v1514 checked unmount refusal/retry. Replacement/lifetime not implied. |
| Shared BCACHE and DTFS error/mount admission | v1490 write/refusal safety; v1518 failed I/O invalidation; v1519 physical native directory admission. |
| Native DTFS persistent identity groundwork | v1520 72-bit `DTVOL1` record, native physical mount validation and per-mount ID retention; not post-error open-FILE or cache reuse. |
| Selected TSFS physical root set and critical HALT leaf | v1516 exact member mask, v1515 installed fatal entry; **unconnected** to authoritative physical loss. |
| Dynamic mount slab and 48K non-XIP | KINIT capacity publication installed; v1486 48K non-XIP physical PASS. Neither proves 32K. |

## Remaining KCORE-only work in dependency order

1. **P0 critical root/BACKSTORE loss detection and dispatch.**
   `proc_storage_runtime_v30.s` provides
   `kcore_critical_device_lost_v1515` with native-tested root,
   swap and invalid-reason HALT points, but no production caller.
   The v1517 SIMH Type-551 `000004` select/illegal status is not
   authoritative physical tape loss. Distinguish actual permanent
   absence from controller PI errors, Type-136/236 timeouts,
   transient access failures and unrelated DTC units; for a TSFS
   root, classify exactly the selected physical member mask.
   Dispatch only genuinely confirmed critical loss and validate
   physical HALT point plus false-positive negative cases.
   Root/BACKSTORE **reconstruction and forced owner release are
   cancelled** by v1513's crash-only policy.
2. **P0 non-root mounted-media revalidation.**
   Compare stored v1520 DTFS IDs against independently physically
   reread IDs after observed error/return before any old clean
   BCACHE, vnode or FILE reuse; preserve mount and dirty/partial
   write ownership and independent root operation. Refuse a
   mismatched replacement; handle legacy zero-ID conservatively.
   A silent tape swap without an observed event remains undetected
   by the current controller contract. Qualify real physical
   same-media return, different media, missing media and errors.
3. **P1 remaining mixed storage/provider failures.**
   Check only cases not already accepted by v1484/v1485/v1490
   and v1511-v1514: in-flight DEVICE/VOLUME mapper interruption,
   multi-member selection, DSK/DRM/DTC boot-root plus auxiliary
   layouts, write-through/dirty shared BCACHE and mounted-media
   identity coherency, simultaneous LOGSTORE/MEMFS snapshot and
   physical process BACKSTORE/MM ownership/rollback. No new
   allocator, second cache, spare log path or forced reclaimed
   process owner should be designed merely for the tests.
4. **P1 dynamic FSINFO consumer admission.**
   Reconcile/commit the existing v1492 local ABI and command
   changes and tests, then demonstrate actual linked guest 1/4/8/26
   mount capacities, holes, provider errors, unpublished capacity,
   mount ID 16/26 boundaries and safe enumeration beyond slot 4.
   Only the necessary existing userland consumers are in scope.
5. **P1 uncompressed swapless inactive PURE.**
   The v1381/v1384/v1391/v1395/v1397 drop/rebase/FULL APR
   restoration components are *not production linked*.
   Integrate explicit opt-in with real scheduler/PI, immutable
   backing, mutable suffix and MM ownership, full APR restoration
   before dispatch, EXEC/RUN/EXIT/WAIT, provider/unmount and OOM
   rollback. Native + physical repeated pressure tests mandatory.
   Original DISK swap unchanged; do not adopt v1487's isolated
   +180 TEXT shared-DXR prototype as a memory saving.
6. **Conditional compression choice, then feature freeze.**
   First accept *uncompressed* PURE and measure actual net peak
   physical RAM, including code, buffers and restoration headroom.
   Enable mutable-only RAM compression only if a proven net win;
   choosing no codec is valid. The v33-v45 PACKED registry and
   compressed DISK-swap plan were cancelled by v1375.
   Run final matched source-linked and guest cross-regression.

## Stale-status cleanup and scope guard

Two old v1508 physical-refusal summaries/checklists duplicated
one diagnostic; the source KCORE/TODO active records were reduced
to one. v1502/v1503/v1505 unchecked physical TERM/retry tests
have later accepted v1506-v1512 evidence, and v1519's missing
unique DTFS format ID was completed by v1520. The old
restricted MEMFS XIP and PACKED disk-swap implementation lists
are no longer active. Their technical history is not deleted.
No claim is made that v1520 can recover a live DTFS mount.

**Excluded:** MRES runtime unload/reload (post-1.0),
independent userland, online D6FS packer and unified TSFS,
low-level optimization/folding, and the deferred strict 32K
memory/boot target. Existing 48K non-XIP acceptance stays valid.
VERSION remains 0.5; no 0.9 designation or final KCORE freeze.
