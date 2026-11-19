# KCORE v1607: remaining PDP-6 feature work

Date: 2026-11-15
Baseline: DAIMOS v1606, VERSION 0.5, fixed KCORE low end
`024364` octal. This is an accepted documentation audit,
NOT a runtime feature or new KCORE image.

The authoritative full work breakdown, source ownership and
acceptance checklist is DAIMOS
`design/kcore-remaining-work-v1607.md`. The active summary
is at the top of DAIMOS `TODO.MD` and `KCORE.MD`.
Do not treat dated earlier status files as cumulative open
work after a later acceptance has superseded them.

## Current release and correctness gates

| Workstream | Current verdict | Grounded acceptance |
|---|---|---|
| P0-A ROOT/BACKSTORE critical failures | COMPLETE | v1548 final combined source-linked and physical matrix |
| P0-B trusted-operator DTVOL2 | COMPLETE UNDER DECLARED POLICY | v1575 writable Type-551, physical DIRTY/CLEAN and partial-write/poisoned-discard/second-boot proof |
| P0-B hardware-enforced reel retention | UNPROVEN / SEPARATE | v1571 physical silent-exchange negative; stock controller has no continuous removable-reel interlock |
| P0-D/E/F requester-only OOM | ACCEPTED FOR FROZEN PROFILE | v1298 F12/F13 64/64 mandatory checks, v1348/v1351 256K no-partner and MM-owner lifecycle |
| P1-C combined storage/swap/BCACHE/LOGSTORE | COMPLETE | v1603 two 128K physical boots, two independent PI6 owners, checked MM/cache/bitmap rollback and physical LOGSTORE retry |
| P1-D dynamic FSINFO/ROOT controllers | COMPLETE | v1601 absent DSK/DRM, tested TSFS/D6FS capacity and native logical boundary |
| P1-E RAW swapless PURE | OPEN | v1381/1384/1391/1395/1604 native prototypes ONLY; not linked to live scheduler |
| Optional mutable-RAM codec | NOT DECIDED | v1451 30-instruction bitmap codec remains standalone/test-only |
| Final source-matched KCORE feature freeze | OPEN | Complete after P1-E; include full native/physical tests and RAM accounting |
| Strict 32K/KCORE optimization | HOLD | Separate authorization after feature acceptance |

### P0-B release statement

An installation meeting v1575's explicit trusted-operator
contract can use accepted writable DTVOL2. It must prevent
physical reel changes while a writable mount owns the volume,
and use checked destructive discard and quarantine after
WRITE_UNCERTAIN. This is a *site operating rule*, not a hardware
interlock. The v1571 physical silent-exchange negative remains
true. Do not call unrestricted/unattended removable-media RW
safe, invent a software generation bit, or reopen v1575's
completed normal partial-write/dirty-recovery implementation
without a new safety requirement.

## Remaining production work: P1-E

The P1-E v1604 architecture review is complete. Its generic
preflight and PDP-6 geometry exist in staged sources. Implement
the following real PDP-6 lifecycle, in this order:

1. **No-swap-backend eligibility and executable identity:** no
   disk fallback, immutable PURE backing held through inactivity,
   exact validated TX2 geometry, mount/unmount/source mutation
   guards, and protected MEMFS XIP exclusion.
2. **Zero-copy RAW inactive memory:** quiesced STOP/SLEEP owner,
   PI/TRANSITION and MM pin checks, v1381 in-place descriptor
   rebase, full mutable suffix retained, atomic storage mode
   2 publication with no extra per-process table.
3. **Correct restore:** v1384 original-gap claim if safe; else
   bounded v1391 complete APR allocation/restore. Source read,
   executable decoding, mutable preservation, owner verification
   and postcommit release must be rollback-safe. No partial APR
   mapping or runnable publication.
4. **Scheduler/lifecycle:** actual PI6 wake, slot-0 pressure and
   service, repeated owners, RUN/EXEC/EXIT/BRK/WAIT/HUP/TERM,
   finalizer, concurrent unmount, no-partner OOM and checked
   failure retries. Ordinary DISK swap and existing MEMFS XIP
   must remain unchanged.
5. **Acceptance:** original KCC PDP-6 no-backing full guests,
   source-linked negative witnesses, refused MM pin/free,
   fragmented restore, invalid source, independent owner
   wakeup, 48K non-XIP and appropriate 96K/128K/256K
   configurations. Report actual KCORE+MRES TEXT/DATA/BSS,
   MM descriptors, permanent RAM, transient restore buffers,
   peak words and achieved reclaim/speed.

The v1487 shared DXR loader pilot increased isolated TEXT
by 180 words and is NOT production. Reuse existing code only
after exact bounded-load and size analysis; do not duplicate
resident parsing simply to enable P1-E.

Do not conflate the v1451 trusted bitmap RAM codec with
compressed *disk* swap. After RAW P1-E is accepted, measure
total net/peak RAM savings for representative process states.
Enable compression only if profitable and fail-closed;
record **NO COMPRESSION** otherwise.

## Final feature freeze and deliberate exclusions

Run a current-source, linked-map and raw-media-verified PDP-6
matrix covering process control, VM/APR/OOM, PI6, disk swap,
ROOT/BACKSTORE fail-stop, DTFS trusted-operator durability,
MEMFS/XIP/LOGSTORE, D6FS clean BCACHE, dynamic FSINFO and
P1-E. Recoverable MM/owner failures must retain correct
references; fatal critical media loss must HALT, not claim
postfatal progress. Capture symbol map, KINIT/boot, KCORE/MRES
footprint, allocators, performance and high-water mark.

The old v1518 DTFS cache test has a source mutation anchor
predating mandatory v1561 authenticated write-intent. Its
current fixture-generation failure is not a passing test.
Update the testkit without bypassing the production gate, or
explicitly document the modern replacement coverage.

The former v1585 timing issue is not an additional P1-C
release blocker after v1603's accepted repeated loaded
guests. Likewise the earlier unchecked P1-D service-matrix
entry is superseded by v1601.

Strict 32K optimization, P10FOLD/P10SUPER, KA10/KT10A, KI10
paging, unified TSFS, D6FS online packer, new DECtape
boot options, and dynamic MRES unload/reload remain **outside
the active KCORE feature-completion scope**. The later CPU
source-map preparations v1605/v1606 stay intact, with no
runtime dispatch or extra PDP-6 mapping state.

## Evidence

* `STATUS/KCORE-FEATURE-STATE-V1603.MD`: P1-C closed.
* `STATUS/KCORE-FEATURE-STATE-V1601.MD`: P1-D closed.
* `STATUS/KCORE-FEATURE-STATE-V1575.MD`: P0-B operator-trust acceptance.
* DAIMOS `design/kcore-dtfs-media-retention-v1571.md`:
  physical strict-interlock gap.
* `KERNEL/SWAPLESS-MEMORY.MD`: P1-E acceptance and v1451 staging.
* `KERNEL/MULTI-CPU-PORTABILITY-V1606.MD`: PDP-6-first source policy.
* DAIMOS `PROJECT-STATE.MD`: v1298/v1351 OOM and current v1606.
