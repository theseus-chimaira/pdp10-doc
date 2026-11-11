# DAIMOS KCORE feature state, v1503

Date: 2026-11-11. Authoritative **feature-completion** update to
`STATUS/KCORE-FEATURE-STATE-V1502.MD`; the v1502 report remains the
historical inventory of accepted functionality. This update adds
strictly scoped real physical TERM/owner evidence. No optimization,
standalone MRES work, general userland work or 32K testing.

## New accepted physical event boundary

Testkit `daimos-backstore-coalesced-physical-v1503.py` (`8987b4a`)
strictly verifies map-backed observations from the completed v1499
256K PDP-6/KCC full guest. Both `proc_swap_service_failed` TERM
requests had identical pre/post owner state: unlocked stable u-area,
nonresident DISK VM, RUN state with no WAIT class, saved executive PC,
and an **already-pending TERM** in FD control. Both calls returned
zero, without changing DISK/FD/scheduler/FINALIZE ownership. The
first deciding condition in production `proc_event_apply` is the
already-pending TERM; saved executive PC separately prohibits remote
destruction of the private stack. The earlier expectation of
immediate PREVM finalization for this owner was wrong.

Physical evidence: one corrupted bitmap allocation bit, three
matching checked frees, one fault repair, two terminal swap-in
service outcomes, final zero process-swap/BACKSTORE blocks, 18 real
guest RUN acknowledgments and 18 reaps. The separate v1503 strict
verifier passes with **15 negative mutations rejected**. Existing
physical v1159, v1203, v1205, v1207 and v1349 MM/owner/lifecycle
verifiers were replayed and PASS. Original v1336 normal-swap tests
correctly reject two intentional fatal swap-ins and remain unmodified.

Important: the original v1499 *branch counters* were inverted by
SKIPN and are invalid. v1502 (`6c3de2c`) corrected SKIPE and passed
an executable PDP-6 positive/negative polarity regression. v1503
uses **only** the independent real before/after snapshots and does
not infer anything from the bad counters. A fresh physical test is
needed to use corrected counters, but no longer needed merely to
identify the already-pending TERM predicate in the saved guest.

## Retained KCORE feature acceptance

| Feature | Current scope |
| --- | --- |
| Core EXEC/RUN/WAIT/scheduler, normal disk/drum swap | Accepted in prior specified 256K profile |
| Requester-only BRK OOM | Accepted v1298 256K physical matrix |
| Grouped D6 block codec | Production-linked/accepted v1464 |
| Restricted raw MEMFS XIP / snapshot rollback | Accepted specified v1469-v1485 128K/192K/256K profiles |
| Non-XIP low-memory boot | 48K v1486 accepted; 32K HOLD |
| BCACHE DTC write/invalidated MM refusal | Native v1490 accepted; physical tape-root pending |
| Checked BACKSTORE refusal / rollback | Real source native v1405/v1493 accepted |
| Real foreign TERM + PREVM idle retry | Native v1498/v1500 accepted; VM backend controlled |
| Deferred TERM + owning-return consumption | Native v1501 accepted; restoration/private close controlled |
| Real physical TERM coalescing on DISK owner | **Scoped v1503 ACCEPTED** |
| Complete physical BACKSTORE corruption / eventual TERM | **OPEN** |
| Mixed DEVICE/VOLUME/BCACHE/LOGSTORE failures | OPEN for untested combinations |
| Swapless PURE-text drop/full APR reconstruction | OPEN; isolated v1381-v1397 prototypes only |
| Dynamic FSINFO mount enumeration | WIP v1492, not accepted |
| Optional mutable RAM compression | HOLD until PURE accepted |

## Prioritized remaining KCORE feature tasks

1. **F2a — physical deferred owner completion:** observe real
   `proc_deferred_term_check_v926` consuming the original pending
   TERM after swap recovery, or an actual safe FINALIZE+PREVM owner,
   and prove the exact selected process's VM/BACKSTORE finalization.
   A global zero-block count or unrelated successful reaps do not
   establish that this particular TERM was delivered.
2. **F2b — permanently refused BACKSTORE:** drive irreparable
   allocation-bitmap/device failure, retaining authoritative DISK
   owner and stable u-area, with no false RESIDENT/READY_ONCE, no
   destructive foreign stack teardown and no unbounded unsafe retry.
   Establish a bounded safe terminal policy with actual native and
   physical evidence.
3. **F2c — full physical corrupt-backing acceptance:** separate
   strict fault-profile verifier covering exact slot, checked bitmap,
   real controller, scheduler, TERM, owner return or finalizer,
   FILE/RT/PI/MM accounting, and final release. Preserve unchanged
   v1336 normal zero-`swapin_fail` acceptance.
4. **F2d — remaining mixed storage failures:** DEVICE/VOLUME mapper
   loss, in-flight DSK/DRM/DTC controller errors, BCACHE media
   replacement for both D6FS and DTFS, AUXSTORE and MEMFS LOGSTORE
   coexistence. Finish real installed-provider DTFS tape-root guest.
5. **F1 — swapless PURE-text reclamation:** implement safe inactive
   selection, immutable vnode backing, exact MM pin/rebase and
   rollback, mutable suffix preservation and full APR restoration;
   accept all EXEC/EXIT/WAIT/provider/media failures and physical
   RAM-pressure cases. Do not link the +180 TEXT-word v1487 pilot.
6. **F3 — dynamic mount enumeration:** settle and commit the v1492
   FSINFO runtime-limit ABI and consumers; test holes, failures,
   >4 slots, 16/26 boundary and at least 1/4/8 inventory cases.
   KINIT mount slab is already dynamic and must remain so.
7. **F4 — optional mutable-RAM compression:** only after uncompressed
   PURE meets F1, prove net whole-system RAM savings after buffers,
   decoder, codec and APR reconstruction workspace.
8. **F5 — remaining targeted physical integration:** only distinct
   unaccepted clock/device/scheduler configurations; do not reopen
   previously accepted normal RAM/clock profiles without evidence.

Out of this feature-only scope: 32K is explicitly HOLD; standalone
MRES dynamic unload/reload is POST-1.0; unified TSFS, the online D6FS
packer and general userland work are separate; KCORE size-only
optimization starts **after** functional freeze. DAIMOS VERSION is
still 0.5, with no 0.9 release claim.

Detailed evidence, snapshots and SHA256 values:
DAIMOS `design/kcore-backstore-coalesced-physical-v1503.md` and
daimos-testkit `daimos-backstore-coalesced-physical-v1503.md`.
