# DAIMOS KCORE v1515: critical-device no-return stop and DECtape policy

Date: 2026-11-12. Source-backed fixed KCORE feature, native PDP-6
acceptance and media-loss classification policy; automatic detection
and physical unplug testing remain OPEN. This supersedes older
proposals for boot-root or BACKSTORE reconstruction after confirmed loss.

## Critical device is determined by active storage role

Confirmed loss of a device providing boot root or BACKSTORE is fatal.
This applies to non-removable disk/drum media **and** removable
DECtape if that tape is a member of the *active TSFS root volume*.
TSFS root may span several tapes: `tsfs_boot_select()` validates the
set; `tsfs_boot_unit_mask_v22()` records selected physical DTC units
for root mount-capacity planning. The TSFS runtime holds the packed
logical-member-to-physical-unit map. Loss of **any root-set member**
is terminal, even when other members remain physically readable.
There is no requirement to replace or rebuild TSFS root online.

Removal of an **auxiliary mounted non-root DTFS** DECtape that holds
neither boot-root data nor BACKSTORE is nonfatal. Its I/O must fail
without invalidating the independent root; the mount and open FILE
ownership remain until checked recovery or clean teardown. The
same reinserted tape and a *different* replacement tape require
identity/superblock and BCACHE/dirty-data validation. The shared
DTC controller itself is not synonymous with critical root membership.
When a physical unit has both root/BACKSTORE and optional mounts,
the critical role takes precedence.

Individual bad blocks, temporary controller PI failures, or a timed-out
Type-136 tape search/transfer (`tape_watchdog_tick_v1089`) do **not**
by themselves prove tape removal. Likewise a Type-270/236 negative
I/O completion does not prove loss of fixed media. Driver status is
currently insufficient to make a reliable automatic permanent-loss
classification. A generic `-1` must never trigger a fatal HALT.

## Implemented fatal leaf v1515

`system/kernel/proc/proc_storage_runtime_v30.s` now exports
`kcore_critical_device_lost_v1515`: AC1=1 stops at root-specific
`HALT .`, AC1=2 at BACKSTORE-specific `HALT .`, and any other
reason at a separate invalid-caller `HALT .`. Before stopping it
disables global PDP-6 priority interrupts. Each continuation returns
to its same HALT, so a subsequent CONTINUE cannot resume normal work.
The linked HALT PC is the diagnostic; no string, extra data word,
process state, ABI change or storage cleanup is used.

Native PDP-6 testkit v1515 checks all three real simulator HALT PCs
against their respective linked code labels and rejects three
executed HALT-removal mutations: six PASS. Full fixed KCC KCORE
build/link PASS. Controlled identical-other-object relinking measures
**+11 fixed instruction words, +0 permanent data words**. The
earlier `HALT 4/5/6` experiment was rejected because those operands
could place the halted PC into low-core vectors rather than at the
safe code locations. Testkit `1f5d8dd`, DAIMOS `28cce2cd`.

No production driver calls this leaf yet. Its entry is intentionally
restricted to a future *authoritatively confirmed* critical-device
loss signal, with VOLUME/root-set ownership checked before dispatch.

DAIMOS report: `design/kcore-critical-device-stop-v1515.md`.

## Remaining KCORE feature gates

1. P0: physical reliable offline/media-removed detection on fixed
   and DTC storage, distinct from transient PI/timer failures;
   resolve root-set and BACKSTORE owners, route to the v1515 leaf,
   physically verify correct fatal PC on critical loss. No recovery.
2. P1: non-root D6FS/DTFS removal, reappearance/replacement, identity
   revalidation, BCACHE invalidation and dirty/open FILE safety.
3. P1: mixed DEVICE/VOLUME, DTFS/BCACHE, LOGSTORE/MEMFS errors.
4. P1: F1 swapless PURE/MM/APR; F3 FSINFO 1/4/8/26 mounts.
5. P2: compression/size optimization only after feature completion.

VERSION 0.5 and 32K HOLD remain. MRES unloading is POST-1.0.
