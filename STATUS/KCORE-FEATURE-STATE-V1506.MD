# DAIMOS KCORE feature state, v1506

Date: 2026-11-11. Incremental authoritative update to
`STATUS/KCORE-FEATURE-STATE-V1505.MD`: the exact real physical
BACKSTORE repairable-fault owner-return boundary is now accepted.
No source optimization, 32K work, standalone MRES development,
general userland redesign or 0.9 release claim.

## New accepted physical BACKSTORE boundary

Testkit `2bf10ff` v1504 completed a fresh 256K KCC PDP-6 guest
with a real allocated BACKSTORE bitmap bit deliberately corrupted
after physical payload read, two failed checked releases, real
coalesced TERM requests and an exact repair at the third matching
release. Actual selected nonresident owner: slot `010`, original
backing first `02350`, count `0330` (octal). The original TERM was
already pending at injection time; the F07 source explicitly sends
TERM to each extra child before waiting for it. The exact first
delivery origin is consistent with that test protocol; the v1504
witness retains only the last of three per-slot event kinds.

After successful restore, the same selected process entered the
real `proc_deferred_term_v970` exactly once, consumed pending TERM
from FD-control `000004,,004010` to `000000,,004014` (private fatal
close latch), and executed `proc_fatal_close_current_v879`,
`proc_finish_common_v932` and `proc_finish_zombie` each exactly
once with terminal status `0400001`. Original backing was released;
final process-swap and BACKSTORE used counts are both zero. Exact
F07 real child acknowledgments/reaps were 18/18.

The dedicated strict physical v1504 verifier requires earlier
v1503's exact owner/bitmap/TERM/F07 assertions and independently
rejects **20** owner-return/wrong-slot/pending-bit/duplicate-close/
status/finish mutations. v1503 rejects another 15 mutations. All
PASS on the completed physical guest. Separate physical v1159
protected owner, v1203 MM peak, v1205 real process slots, v1207
swap stack and v1349 owner lifecycle checks also PASS.

Evidence SHA256 and technical source audit are recorded in DAIMOS
`design/kcore-backstore-owner-return-physical-v1506.md`.

The normal v1336 zero-failed-swap-in postflight intentionally
rejects the injected two failures. The separate v1504 recovered-
fault acceptance does not relax any normal-swap requirement.

## Current feature status and remaining ordered tasks

| Workstream | Status |
| --- | --- |
| Production basic KCORE, EXEC/RUN/WAIT and normal swapping | Accepted in specified prior profiles |
| Requester-only BRK OOM | v1298 256K full release acceptance |
| Grouped D6 codec | Production v1464 accepted |
| Restricted raw MEMFS XIP, remount/rollback | v1469-v1485 profiles accepted |
| 48K non-XIP boot | v1486 accepted; 32K HOLD |
| BCACHE DTC write and refused MM free | v1490 native accepted; physical tape-root OPEN |
| Repeated corrupt BACKSTORE rollback | v1505 native accepted |
| Actual owner-return TERM after repaired BACKSTORE | **v1504/v1506 physical ACCEPTED** |
| Permanent corrupt backing/DEVICE loss terminal policy | **OPEN** |
| Mixed media, BCACHE/DTFS/LOGSTORE and DEVICE failure gates | OPEN |
| Swapless PURE-text reclamation and APR restoration | OPEN |
| Dynamic mount enumeration ABI | v1492 local WIP, not accepted |
| Optional mutable-RAM compression | DEFERRED pending PURE and positive RAM measurements |

Remaining priority sequence:

1. Design, implement and physically qualify **unrecoverable**
   BACKSTORE/DEVICE error behavior: bounded retry or quarantine
   while keeping the authoritative DISK run, stable u-area and
   active private executive continuation. Do not treat a repaired
   bitmap or six forced v1505 retries as permanent-fault acceptance.
2. Complete remaining distinct mixed-device/VOLUME, DTFS tape-root,
   BCACHE/LOGSTORE/MEMFS failure and recovery integrations.
3. Integrate uncompressed swapless PURE-text drop/reclaim, exact
   MM/immutable backing and full APR reconstruction across EXEC,
   exit, WAIT, rollback and scheduling. Do not add the measured
   +180 TEXT-word isolated v1487 candidate.
4. Finish the v1492 dynamic FSINFO enumeration up to 26 slots,
   legacy consumers and physical 1/4/8/26 acceptance.
5. Only after PURE, evaluate optional compression using matched
   resident-RAM accounting; complete KINIT/clock/device gates,
   then undertake final KCORE size optimization.

Unified TSFS and online D6FS packer remain separate 0.9 work;
MRES runtime code unload/reload remains POST-1.0. VERSION is 0.5.
