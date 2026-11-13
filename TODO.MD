## 2026-11-13 v1539: one TSFS root member physically lost (ACCEPTED)

- [x] Full 256K PDP-6 selected three-member TSFS root: physically
  detach DTC0 only, retain attached DTC1/DTC2, then issue a required
  uncached root-file read. Exact root-specific HALT at linked octal
  PC `003743` PASS. No extra resident code or DATA/BSS.
- [x] v1538 full root-set loss and v1534 independent nonroot DTFS
  failure remain PASS with the revised fail-stop policy.
- [ ] Still open: real D6FS disk/drum-root physical outage, separate
  BACKSTORE unavailability escalation with no false transient PI
  HALTs, and combined device-owner acceptance. Do not revive the
  superseded requirement for a hardware proof of permanent root loss.

See `STATUS/KCORE-FEATURE-STATE-V1539.MD` and the testkit v1539
physical witness. The v1538 status section below is retained as
historical evidence and superseded for single-root-member coverage.

## 2026-11-13 v1538: current root-storage P0-A policy and acceptance

- [x] Root provider (selected D6FS or TSFS) now fails stop on a
  genuine backing read/write error after the physical driver's usual
  bounded recovery. Secondary mounted filesystem errors remain local.
  Source-linked native PDP-6 D6FS 11 cases/5 mutants and TSFS 12
  cases/4 mutants PASS; fixed KCORE and matching optional MRES
  KCC links PASS. No new resident DATA/BSS, no size optimization.
- [x] Real 256K KCC TSFS-root guest with physically detached DTC3
  leaves independent DTC4, MEMFS and TSFS root operational (v1534
  current-source revalidation PASS under v1538).
- [x] Real 256K PDP-6 TSFS-root DTC0/DTC1/DTC2 physical detachment
  followed by an actual `/SYSTEM/INIT` root data read: exact v1515
  root HALT PC `003743` PASS. Warm metadata cache hits alone are
  not evidence of a physical root I/O failure. Disk/drum D6FS and
  isolated single-member TSFS-root outages still need acceptance.
- [ ] Separate BACKSTORE-critical policy for truly unavailable swap
  service; preserve accepted v1511/v1512 recoverable PI/watchdog
  behavior and checked held-owner bitmap/slot accounting.
- [ ] Complete physical root and BACKSTORE positive/negative suite;
  no premature declaration of entire P0-A closure. Full P1 work
  remains in DAIMOS `TODO.MD` current head.

## 2026-11-12 v1520: native DTFS random volume ID and mount retention PASS

- [x] Optional native DTFS `DTVOL1` identity record in reserved
  RBN0: 72 random host-generated bits, checksum, zero padding.
  Old all-zero legacy media still mounts; partial/corrupted
  stamps fail closed. Host mkdtfs, C89-compatible userspace probe,
  target MKFS.DTFS clear-before-format, native PDP-6 mount/identity
  storage and five executed negative mutations PASS. Verified
  identity lives in existing per-mount TSFS shape/extent words,
  no extra permanent RAM or MM allocation. Controlled +46
  optional MRES code, zero fixed KCORE. Isolated 256K KCC and
  32K GCC tape-root links PASS; strict 32K size gate still FAILS.
- [ ] Checked *mounted* DTFS media identity transition before
  accepting I/O on old FILE/vnode and cached state after errors
  or replacement. Handle dirty writes, legacy tapes with no ID,
  open descriptors and stale mount-slot reuse safely; no implicit
  continuation onto a different tape. Stamped mount is NOT recovery.
- [ ] True Type-551 physical device loss signal and v1515 TSFS
  root-member fatal dispatch remain OPEN independently.

## 2026-11-12 v1519: native DTFS admission is physically checked

- [x] Direct `SYS_DTFS_MOUNT`/DTFS MOUNT_UNIT must not bypass
  native format validation merely because DTFSPROBE is normally
  run by userland. A fresh raw DTC directory read (not BCACHE)
  now checks magic, reserved allocation owners and native
  terminal tags before VFS publication. Actual PDP-6 source
  test and six negative mutations PASS. DTFS/BCACHE regressions,
  GCC 32K and KCC 256K tape-root full links PASS. Cost +27
  optional MRES words, zero data/fixed KCORE; strict 32K KCORE
  size gate remains FAILED and must not be reported as accepted.
- [ ] Native DTFS lacks a persistent unique volume ID: decide
  exact identity storage/compatibility strategy before enabling
  safe different-tape replacement or resurrecting open/dirty FILE
  and vnode ownership. Same-format magic is not identity.
- [ ] Find real Type-551 physical-loss evidence, route confirmed
  TSFS root member disappearance to v1515 HALT, and physically
  test independent non-root DTFS return/replacement handling.

## 2026-11-12 v1518: DTFS observed-I/O error cache coherency PASS

- [x] Source-linked PDP-6 DTFS physical read/write, directory
  loader and real clean BCACHE now invalidate both cache tags
  and the directory mount identity on observed negative hardware
  return. Four actual negative source mutations rejected; v1490
  BCACHE ABI/MM-refusal, DTFS and TSFS regressions PASS. Controlled
  cost six optional MRES instructions, zero data/fixed KCORE words.
  GCC 32K and KCC 256K tape-root KINIT/MRES LINK; strict 32K
  fixed-KCORE size gate independently FAILS, not accepted.
- [ ] Authoritative Type-551 media removal/loss event (not PI or
  timeout), root-member fatal dispatch and real physical acceptance.
- [ ] Non-root DTFS **replacement** media identity, dirty/open FILE
  safety, BCACHE and checked revalidation/remount. Silent detachment
  cannot be detected solely by this after-error cache fix.

## 2026-11-12 v1517: Type-551 selection-status experiment PASS

- [x] Native PDP-6 SIMH legal SEARCH/SELECT/CONI probe PASS for
  unattached, attached, detached DTC0 and unrelated DTC7.
  `DTB_ILL=000004` identifies the simulated selection error,
  **not** a hardware-certified physical reel-removal event.
  Testkit `3536d98`; no production KCORE code/data change.
- [ ] Find authoritative Type-551/TU55 hardware-compatible
  device/tape loss signaling rather than inferring loss from
  illegal command/selection bits or elapsed search/transfer time.
  Route confirmed active TSFS root-member loss to v1515 HALT,
  but keep unrelated non-root DTFS failures nonfatal.
- [ ] Physical guest removal versus timeout, root HALT-PC and
  auxiliary DTFS retained mount/FILE, replacement validation,
  BCACHE and dirty/open resource safety.

## 2026-11-12 v1516: exact root-member DECtape classification

- [x] KINIT TSFS boot now retains exact root physical-unit mask
  in unused bits 24..31 of its existing per-mount shape word,
  avoiding unit-zero versus unused-padding confusion. Native
  source-linked query checks namespace root/mount bounds/mask;
  1/2/8 member host, full native four mutation negatives, TSFS
  stored and cross-member reads PASS. GCC 32K and KCC 256K full
  KINIT/MRES links PASS, but strict 32K KCORE size FAILS.
- [ ] Find an authoritative controller/media removal or change
  indication separate from ordinary PI errors, illegal commands
  and Type-136 tape watchdog timeouts; test actual root-set loss
  routed to v1515 HALT and unrelated non-root tape nonfatal path.
- [ ] Non-root DTFS identity/superblock revalidation, BCACHE,
  dirty/open resources and full physical detach/return/replacement.

## 2026-11-12 v1515: role-based fixed/removable device loss

- [x] Implement no-return v1515 root/BACKSTORE critical-device
  HALT leaf with three distinct safe link-symbol PCs and global
  PI disabled. Full fixed KCC KCORE link PASS; six native PDP-6
  positive/mutation cases PASS; +11 code words, 0 data words.
- [x] Clarify that a removable TSFS **root-set DECtape member** is
  as critical as fixed root media; non-root auxiliary DTFS tape
  loss is not. Only selected root-set membership is fatal.
- [ ] Classify reliable permanent loss or removal distinctly from
  individual bad blocks, temporary PI errors and tape/DRM timeouts;
  route actual root-set/BACKSTORE provider loss to v1515 and
  physically verify fatal PCs, including multi-tape TSFS root.
- [ ] Non-root D6FS/DTFS reappearance/replacement with validated
  identity, BCACHE invalidation and dirty/open FILE safety; mixed
  storage providers, PURE/MM/APR, dynamic FSINFO remain OPEN.

## 2026-11-12 v1514: non-root retained mount and checked unmount PASS

- [x] Source-linked PDP-6 native v1514 tests real VFS unmount:
  non-root D6FS sync failure retains mount/RO/root namespace
  and leaves FILE owner alone; return of the same medium permits
  provider cleanup and exact secondary retirement. Three
  negative ordering mutations rejected; no production RAM.
- [ ] Actual physical unplug/return and *replacement* media:
  revalidation, BCACHE invalidation, dirty data, pinned/open FILE
  safe lifecycle. Native failure/return is not full recovery.
- [ ] Confirmed root/BACKSTORE loss classifier and fatal HALT,
  no reconstruction, plus mixed providers, F1 PURE/MM/APR,
  dynamic FSINFO; compression/optimization afterward.

## 2026-11-12 v1513: critical media HALT; secondary media recovery

- [x] Adopt crash-only policy for **confirmed** root and
  BACKSTORE media loss; supersede old reconstruction/recovery
  proposals. Do not panic on a lone PI error or watchdog timeout.
- [x] Native PDP-6 v1513 non-root D6FS failure/retry and root
  reader isolation; no clean cache entry from failed reads;
  three actual negative mutants rejected. Corrected old D6FS
  multi-mount test-only linkage. No production RAM or ABI change.
- [ ] Distinguish permanently offline root/BACKSTORE from a
  transient read failure; implement a minimal terminal HALT
  and optional cheap diagnostic, then physically qualify it.
- [ ] Non-root mounted media return/replacement: checked
  identity and superblocks, BCACHE invalidation, dirty-block
  safety and live FILE/lease behavior, physical acceptance.
- [ ] Mixed DEVICE/VOLUME, DTFS/BCACHE, LOGSTORE/MEMFS errors,
  F1 swapless PURE/MM/APR, F3 FSINFO 1/4/8/26 enumeration;
  optimization/compression deferred until features are done.

## 2026-11-12 v1512: physical Type-236 PI6 watchdog accepted

- [x] Boot real 256K KCC PDP-6 and suppress two selected real
  drum DR_DONE publications without falsifying slot-0 status.
  The original PI6 300-tick watchdog timed out the still-owned
  private descriptor twice, correctly accounting DRM storage
  errors and returning through BACKSTORE read/MM rollback.
  Sixteen fault-on samples preserve exact bad DISK `02350`/`0330`,
  u-area, bitmap, retry marker and pending TERM, with 30 later
  refused checked frees, 32 total negative restores, no phantom
  READY_ONCE, ZOMB or forced reclaim. Strict physical v1512 PASS.
- [x] Independent formerly swapped PID `011` checked-restored
  and executed 562 real SLEEP calls; resident PID 5 progressed
  by 992 further SLEEP calls. Four source, twelve evidence and
  missing-digit-field negative checks rejected. No new KCORE RAM.
- [ ] Actual destroyed/unavailable DEVICE/media reconstruction
  or safe retained ownership without forced free/false parent WAIT.
- [ ] Mixed DEVICE/VOLUME/BCACHE/DTFS/LOGSTORE/MEMFS recovery;
  F1 swapless PURE/MM/APR; F3 FSINFO mounts; optimization later.

## 2026-11-12 v1511: physical DRM BACKSTORE read errors accepted

- [x] Real 256K KCC PDP-6 checked two Type-167/236 PI2 error
  completions, actual DRM accounting and private descriptor
  wakeup, BACKSTORE negative read/MM rollback, 25 checked-release
  refusals, 27 negative source-linked slot-0 completions, intact
  DISK `02350`/`0330`, u-area, bitmap, pending TERM/retry, no
  phantom READY_ONCE/ZOMB. A separate swapped PID `011` restored
  and executed 397 real SLEEP calls. Strict verifier PASS;
  five source and eleven physical mutation controls rejected.
- [ ] v1512 actual booted Type-236 lost-PI2/PI6 watchdog return
  and owner rollback; native/prerequisite tests PASS only.
- [ ] True persistent hardware/media loss, reconstruction and
  safe long-lived owned-failure policy (never force free).
- [ ] Mixed DEVICE/VOLUME, DTFS/BCACHE, LOGSTORE/MEMFS;
  F1 PURE/MM/APR and F3 FSINFO mounts; optimization afterward.

## 2026-11-12 v1510: independent swapped owner physically accepted

- [x] Under one genuine permanently refused BACKSTORE checked free,
  independently observe healthy swapped PID `011` restoring through
  a checked production transaction and executing 331 real SLEEP
  syscalls. Original failed slot `010` preserved through 13
  fault-on physical samples, while resident PID 5 also advanced.
  Strict v1510 physical verifier PASS with eight source and eleven
  false-evidence mutations rejected. Zero production RAM/state.
- [ ] Physical DEVICE/media-error propagation, watchdog and safe
  preservation or reconstruction of a genuinely irreparable owner.
- [ ] Remaining KCORE mixed storage failures, PURE/MM/APR and
  dynamic FSINFO feature integration before final optimization.

## 2026-11-12 v1509: physical runnable fairness after permanent swap failure

- [x] Real 256K PDP-6 physically held failed DISK slot `010`
  across six clock-progressing snapshots with 3..14 refused
  checked BACKSTORE frees, unchanged u-area/bitmap/TERM and
  no premature finalization. Independent PID 5 was truly live
  and resident, with real SLEEP entries 98..456 (+358) during
  this failure. Strict physical v1509 and source/evidence mutation
  regressions PASS; no production process state or RAM growth.
- [ ] Demonstrate independent *swapped* contender restoration and
  user dispatch under the same persistent-fault conditions.
- [ ] Investigate genuine permanent DEVICE/media loss and safe
  recovery or explicitly retained ownership without forced free.
- [ ] Complete mixed storage DEVICE/VOLUME, DTFS/BCACHE,
  LOGSTORE/MEMFS, swapless PURE/MM/APR and dynamic FSINFO gates.

## 2026-11-12 v1508: physical persistent BACKSTORE refusal

- [x] Test genuinely booted 256K PDP-6 source-linked selected DISK
  owner with every checked BACKSTORE free refused. Strict physical
  v1508 verifier PASSes three clock-progressing fault-on samples,
  1/4/7 failed frees, preserved authoritative DISK bitmap/u-area,
  pending TERM and zero premature finalizers; ten unsafe mutations
  rejected. No new production state or RAM.
- [ ] Verify independently runnable/swapped bystander progress
  physically under the selected persistent failure (native v1507
  selection/fairness already passed).
- [ ] Qualify genuinely irrecoverable backing/DEVICE faults and safe
  retained ownership, or genuine reconstruction before teardown;
  do not turn retry throttling into a false ZOMB/release claim.
- [ ] Continue mixed DEVICE/VOLUME, BCACHE/DTFS/LOGSTORE/MEMFS,
  PURE/MM/APR and dynamic FSINFO feature integration.

### 2026-11-07 v1298 F12/F13 production OOM signoff

The requester-only BRK physical-OOM production policy is now enabled
and accepted in the strict 256K KCC PDP-6 DSK/D6FS release profile:
64/64 OOM gates CLOSED. F03 14/14, P0-D native 18/18, physical
17/17 individually and full-slot F07 both policies pass; archived
independent Android/Linux post-activation E13 release media and strict
F10 diagnostics verify against the same one-switch candidate.
`KERNEL/OOM-F13-PRODUCTION-ACCEPTANCE-V1298.MD` contains authoritative
source revisions, scoped acceptance and evidence SHA256s. Supersede
older F12/F13-OPEN pointers, but do not mark the four pre-KCORE-sweep
functional tracks complete: D6CODEC, BSTORE, restricted MEMFS XIP
and (since v1375) swapless-only PURE text dropping / opt-in
mutable-RAM compression. Compressed disk swapping is cancelled.
Unified TSFS and final RAM optimization remain separate scope.

### Storage vocabulary and 0.9 feature acceptance (v661)

The current source terminology is DEVICE (physical unit), VOLUME
(optional multi-device mapping), REGION (bounded physical range),
BACKING (temporary authoritative swap/MEMFS blocks), LOG (durable
records), and CACHE (discardable copies). AUXST1 parsing is boot-time
configuration, not a new permanent dispatch layer. Direct LOG REGION
bootstrap routing landed v661. Remaining runtime feature integration,
failure recovery, CACHESTORE L2 and the online D6FS packer are
version 0.9 scope. DTFS/TSFS implementation is temporarily deferred,
not removed from release scope. No KCORE size sweep is underway.

# PDP10-DOC TODO

## 2026-11-12 v1507: per-owner BACKSTORE retry native PASS

- [x] After checked swap-in refusal, delay precisely the failed
  nonresident DISK owner using the stable u-area retry epoch, not
  a new process state or global fatal cooldown. Healthy swapped
  and runnable bystanders remain schedulable, including against
  two independently failing owners and realtime lease selection.
- [x] Test 0100-tick PI6 delay, 17-bit epoch wrap, marker retirement,
  real checked repair and original DISK ownership; three negative
  mutations plus v1493/v1505/v1316-v1317 native regressions PASS.
  Full isolated 256K KCC image build PASS.
- [x] Native v788 paired publication and v854 sector/PURE exchange
  verify checked incoming-owner retry-tag retirement.
- [ ] Qualify the sustained permanent-failure policy in a booted
  physical guest; verify actual bystander progress and that neither
  unsafe forced free nor fake process finalization is exposed.
- [ ] Complete mixed media/BCACHE/LOGSTORE/DTFS/MEMFS, F1 PURE
  and F3 mount enumeration gates before final KCORE optimization.
  See `STATUS/KCORE-FEATURE-STATE-V1507.MD`.

## 2026-11-11 v1506: selected physical TERM owner completed

- [x] Exact selected nonresident DISK owner `010` consumed the
  original pending TERM once after real bitmap repair, executed
  one private fatal close/finish/zombie, and released backing.
- [x] Fault-specific v1504 strict physical verifier plus 20 wrong
  owner/term/status mutations PASS; v1503's 15 bitmap/coalescing
  mutations and independent v1159/v1203/v1205/v1207/v1349 audits
  also PASS; F07 18/18 and final 0/0 backing.
- [ ] Define bounded terminal failure response when the bitmap/
  DEVICE cannot be repaired. Keep DISK ownership and private
  executive continuation; do not repeatedly perform unsafe
  restorations or silently lose backing.
- [ ] Finish other F2 mixed DEVICE/VOLUME/BCACHE/DTFS/LOGSTORE cases,
  F1 swapless PURE with APR, F3 dynamic mount acceptance, optional
  net-RAM-positive compression after PURE, then remaining physical
  gates and KCORE size optimization. See
  `STATUS/KCORE-FEATURE-STATE-V1506.MD`.

## 2026-11-11 v1505: exact repeated BACKSTORE refusal gate

- [x] Source-linked actual checked bitmap/swap-in/slot-0 PDP-6
  transaction retains the DISK owner through six *forced* repeated
  refusals, then restores exactly once after explicit bit repair.
- [x] No MM pin leak, false RAM publish, READY_ONCE or accounting
  decrement; v1493 unsafe mutations re-PASS.
- [x] Correct documented service success AC1 to slot|READY_ONCE,
  and failure to TERM request (not guaranteed immediate termination).
- [ ] Physical selected-owner TERM consumption after repaired backing:
  v1504 running; require target-specific proof, not global zero usage.
- [ ] Permanent corruption terminal policy and bounded retries without
  losing DISK owner/private executive stack; full fault-specific
  physical acceptance remains OPEN. See
  `STATUS/KCORE-FEATURE-STATE-V1505.MD`.

## 2026-11-11 v1503: physical TERM classification and remaining gates

- [x] Re-verify exact v1499 physical 256K pre/post owner snapshots:
  two already-pending TERM requests safely coalesced, saved PC still
  executive, no WAIT, DISK VM unchanged, no immediate PREVM.
  Testkit v1503 15 invalid-evidence mutations all rejected. Do not
  use the old inverted v1499 branch counters for this verdict.
- [ ] Prove actual pending TERM **consumption** at the owning return
  after physical restore, then correct real private close/teardown.
  Native v1501 controls restored VM and final close.
- [ ] Test permanent corrupt BACKSTORE refusal and full physical
  terminate/finalizer/checked backing-release accounting without
  relaxing the strict normal F07/v1336 acceptance.
- [ ] Continue distinct mixed-device/BCACHE/DTFS/LOGSTORE gates,
  uncompressed swapless PURE/full APR integration, uncommitted v1492
  mount-capacity ABI acceptance, optional RAM-positive compression
  after PURE, and remaining targeted physical profiles.

Authoritative current state: `STATUS/KCORE-FEATURE-STATE-V1503.MD`.
MRES code unload/reload remains POST-1.0; 32K stays HOLD; final
KCORE size optimization waits for functional freeze.

## 2026-11-11 v1502: current KCORE feature-completion tasks

Native immediate-TERM/finalizer v1500 and deferred owner-return
v1501 both PASS, with real production event code and controlled
backend; v1498 positive/negative also re-PASS. The initial v1499
physical run produced valid pre/post owner snapshots but inverted
its branch-counter `SKIPN` scope. The testkit v1502 `SKIPE` fix and
native mutation regression PASS, but corrected full-guest branch
classification is still OPEN. Do not claim corrupt-BACKSTORE physical
acceptance or assume that its third free is an idle PREVM retry.

- [ ] F2: physically classify both TERM decisions using corrected
  v1499/v1502 map-backed WAIT/PC/FD-control/owner and branch evidence;
  reject original out-of-scope event counts.
- [ ] F2: prove permanently refused BACKSTORE fail-closed policy,
  eventual safe delivery, finalizer retry and exact backing ownership;
  add a separate fault-specific guest acceptance, without relaxing
  the original normal-swap F07/v1336 matrix.
- [ ] F2: close distinct mixed-device, BCACHE/DTFS tape-root,
  LOGSTORE and AUXSTORE integration failure gates.
- [ ] F1: integrate and accept uncompressed swapless PURE drop and
  complete APR restoration through the production scheduler.
- [ ] F3: finish dynamic FSINFO mount-capacity ABI/consumer v1492
  changes and acceptance; they are uncommitted work, not a released API.
- [ ] F4: consider mutable-only RAM compression only if F1 passes and
  matched complete-memory accounting proves savings.
- [ ] F5: close remaining targeted guest device/clock profiles.

Versioned status: `STATUS/KCORE-FEATURE-STATE-V1502.MD`.
No 32K testing, MRES module unload/reload, standalone MRES work or
size optimization is reopened by this documentation reconciliation.

## 2026-11-11 v1500: integrated real TERM/finalizer native coverage

The v1500 source-linked native PDP-6 test executes real event
delivery, process FINALIZE+PREVM and idle retry together, with
controlled refused VM and u-area MM release. Exact DISK ownership,
deferred retries, once-only FILE/HUP/VM effects, zombie publication
and negative missing-PREVM mutation PASS. Full physical BACKSTORE
failure acceptance still requires v1499 branch classification,
owner-safe TERM consumption and permanently invalid backing checks.
See DAIMOS `design/kcore-backstore-event-finalizer-v1500.md`.

## 2026-11-11 v1498: source-linked TERM deferral gate

The actual `proc_event_apply` now has a native v1498 DISK-owner
positive/negative gate: WAIT/saved executive PC cause safe deferred
TERM, repeated requests coalesce, and a saved user PC permits
foreign finalization whose refused VM release preserves ownership.
Pending physical acceptance: identify exact event predicates for
v1495, prove correct deferred or immediate TERM completion under
corrupt backing, and separately test permanently unrecoverable
bitmap refusal. Keep normal v1336 acceptance unchanged.

## 2026-11-11 v1497: precise physical BACKSTORE fault outcome

The v1495 physical 256K KCC guest reached HALT and final zero backing
blocks, with 1 bitmap fault, 3 matching releases, 2 negative swap-in
service returns and 2 real TERM calls, neither entering PREVM. The
third release lacked PREVM too, disproving the v1494 finalizer-retry
assumption. Independent MM/owner/slot/stack verifiers pass; the full
fault-specific gate stays OPEN. Next: classify deferred TERM against
WAIT/CTX_PC and pending event bits, test owner-safe delivery (or an
immediately finalizable owner), and independently test permanent
corruption without false ownership release. Keep normal v1336 strict.
Source: DAIMOS `design/kcore-backstore-physical-f07-diagnostic-v1497.md`.

## 2026-11-11 v1496: F07 fault-injection acceptance prerequisites

The F07 shared runner incorrectly counted SIMH's
`+uncommitted-changes` banner as a process acknowledgement. Fixed in
testkit v1496; five strict marker positive/negative cases PASS, and
the captured physical run contains exactly 18 guest acknowledgments
and 18 reaps. Normal v1336 acceptance rejects all terminal swap-in
failures, correctly for an unmodified guest; a separate fault-specific
verifier must link each expected failure with the injected owner and
full TERM/PREVM cleanup. The v1494 physical run is NOT accepted;
v1495 diagnostics separate those remaining uncertainties.

## 2026-11-11 v1494: KCORE PREVM finalizer recovery gate

The production process finalizer and idle PREVM retry execute in
testkit v1494 after a controlled VM destroy refusal, followed by
real finalizer stage retries. Retained owner/status, no repeated HUP
or double VM free, no premature zombie, and premature-commit negative
mutation all PASS. Existing v1081/v1086 regressions PASS. The real
BACKSTORE bitmap/TERM/finalizer **combined** physical guest still
needs acceptance. See `KERNEL/BSTORE.MD` v1494.

## 2026-11-11 v1493: KCORE checked-release scheduler acceptance

Native PDP-6 source-linked slot-0 scheduler + real swap-in + real
BACKSTORE bitmap checked-release refusal PASS. The DISK owner survives,
replacement MM rollback succeeds, TERM is requested exactly once,
and READY_ONCE/exchange/cooldown stay disabled. Two executable
negative mutations and the existing v1405 transaction test PASS.
This closes only the native service boundary; real full-guest TERM/
finalizer/physical-corruption recovery remains OPEN. No code-size
optimization or MRES module changes. See `KERNEL/BSTORE.MD` v1493.

## 2026-11-11 v1490: BCACHE/DTC feature-correctness closure

Completed in KCORE: DTC write preserves AC1..AC3 and the clean L1
cache invalidates even if the private MM slab cannot be freed. PDP-6
source-linked v1490 and two negative mutations PASS; no new allocator
or independent MRES work. Current evidence and remaining separate
physical/combined-media tests are in DAIMOS
`design/kcore-bcache-write-reclaim-v1490.md`. Historical v572
disk-root DTFS mount-success acceptance is stale: present boot policy
does not install the DTFS MRES on disk root, and the later v1 online
oracle intentionally requires refusal. v578 tape-root test's retired
host-tool build step has been updated to use installed `mktsfs`.

## v1489 feature-only KCORE sweep (2026-11-11)

See DAIMOS `design/kcore-feature-source-sweep-v1489.md` for the precise
remaining KCORE tasks: storage failure/mixed-media tests, swapless
PURE scheduler/MM transaction and FULL APR restoration, dynamic-mount
enumeration interface above slot four, and conditional mutable-RAM
compression after uncompressed reclaim is proven. KCORE KINIT clock/
hardware regressions remain targeted acceptance, not missing APIs.
The identity-only unlinked kpack v1 stub was removed; other staged
revisioned sources remain because testkit uses them. This is a
feature-only inventory and does not reopen 32K, MRES module unloading,
independent filesystem/userland work or KCORE size optimization.

## 2026-11-11 v1488: documentation-source reconciliation

Current KCORE gates are maintained by DAIMOS/TODO.MD v1488 and the
single source-backed reference `STATUS/KCORE-RELEASE-AUDIT-V1488.MD`.
The v512/v1029/v1269/v1286 snapshots below were not deleted, but
must NOT be read as live open gates: grouped codec v1464 and
restricted MEMFS XIP v1469-v1485 are already in production, and
F12/F13 physical BRK OOM is enabled/accepted v1298. Native v1486
48K non-XIP guest passes; all 32K tests remain on hold.

Swapless PURE scheduler/restore and optional RAM-only mutable
compression remain OPEN, despite successful isolated v1381/v1384/
v1395 and v1451 tests. The v1487 isolated common EXEC candidate
adds 180 TEXT words and stays unlinked. Backing-service remaining
gates, online D6FS packer and post-KCORE/pre-0.9 unified TSFS remain
separate current release tasks. Runtime MRES unload/reload is
preserved for POST-1.0 only and must not be worked on until 0.9
has shipped. All dated notes below are historical evidence.

## 2026-10-27 DAIMOS v512 documentation synchronization

V521 GRANULARITY AUDIT (2026-10-27): The general DAIMOS MM allocator
already supports exact word counts and one-word alignment. The PDP-6 APR
process VM still requires 02000-octal (1024 decimal) word mapping
alignment. The P0 objective is to find and remove avoidable rounding in
software-managed allocations, never to claim unsupported 512-word APR
process mappings. Source-audit evidence is recorded in DAIMOS
`design/mm-granularity-audit-20261027-v521.md`; update implementation
claims in `KERNEL/MEMORY-MANAGEMENT.MD` only after native tests pass.

DECISION 5 (REPLACED 2026-11-09 v1375): opt-in compression of **disk
swap records** is cancelled. Existing raw process swapping remains
unchanged. The replacement is **swapless-only** PURE executable-text
dropping and, independently, opt-in compression of **mutable process
memory in RAM**, with explicit admission and no net RAM penalty. Both
are currently unimplemented and require native scheduler/VM/rollback
acceptance before final KCORE optimization. The PDP-6 APR cannot map
text and data independently: reconstruct one complete physical
process extent before running. See `KERNEL/SWAPLESS-MEMORY.MD`.

DECISION 4 (2026-10-27): production privileged, strictly non-writing
MEMFS XIP EXEC/RUN and native lifecycle validation are required before
the final KCORE optimization and the DAIMOS 0.9 release. Current isolated
pin/admission/VM tests are not production process launch acceptance.

DECISION 3 (2026-10-27): finish remaining BSTORE shared backing,
BCACHE and LOGSTORE migration before the final KCORE optimization and
DAIMOS 0.9 release. Document tested implementation changes in
KERNEL/BSTORE.MD and related filesystem/device documents; distinguish
accepted AUXST1/BACKSTORE support from genuinely unfinished work.

DECISION 2 (revised 2026-11-09): The target is one TSFS format and resident
read engine for optional non-root single-member RW and multi-member RO tape
volumes. A TSFS root is ALWAYS RO. The minimal `ROOT=tape TAPE_AUX=none
TAPE_TMPFS=0` profile has no swap, BACKSTORE, LOGSTORE or temporary RAM
filesystem at link time, even if unconfigured devices are detected. An
explicit build-time `TAPE_AUX=drum|disk` configuration permits auxiliary
backing; detecting hardware alone never activates it. Neither
the old TSFS/DTFS media formats nor foreign OS personalities need to remain
readable. Use D6codec for immutable compressed file payloads, with RW payloads
normally stored uncompressed. Tape-mechanical latency and permanent RAM/code
consumption are both acceptance criteria, with the 32K-word tape root's memory
budget a hard constraint. The sharded-metadata proposal is historical input,
not an independently mandated format. Keep the implementation in removable
filesystem MRES and host image tools; preserve generic KCORE/VFS interfaces
where possible. See `KERNEL/DECTAPE.MD` for open format/durability decisions.
This remains scheduled after the final KCORE optimization. This is a
schedule decision only, not evidence that the unified format is implemented.

P0 UPDATE (2026-10-27): memory allocation granularity reduction to
512 decimal words is a pre-final-KCORE prerequisite, not post-release
research. Document the exact octal/decimal unit conventions and the
PDP-6 APR 1024-word hardware process-map alignment separately from
software MM allocation granularity. Update MEMORY-MANAGEMENT.MD with
measured, tested behavior after implementation; do not claim native
512-word PDP-6 protected process mappings without hardware evidence.

DAIMOS/TODO.MD now owns the consolidated pre-final-KCORE functional freeze
inventory. This document tracks only design/current-state documentation,
not a second implementation queue. Keep clock/DATE/SETTIME, explicit
MKSWAP/AUXSTORE, mixed-device/BLOCKSET inventories, swapped MonitorFS
process metadata and scheduler acceptance synchronized when implemented.
Before declaring the KCORE ABI frozen, record explicit release-scope decisions
for D6 streaming codec cutover, unified TSFS, remaining BSTORE migration,
production MEMFS XIP and swapless memory reclamation. Do not mark these complete
based on design documents or isolated component tests. Accepted v508-v511
32K AUXST1 results supersede earlier provisional failures; avoid a new raw
AUXSTORE D6FS superblock requirement. Post-0.9 overlays and runtime module
movement remain deferred. Update subject documents only after tested changes.


DATE: 2026-09-30

THIS FILE TRACKS OPEN DOCUMENTATION WORK ONLY.  IMPLEMENTATION PRIORITY IS
DEFINED BY DAIMOS/PROJECT-STATE.MD AND DAIMOS/TODO.MD.  COMPLETED DOCUMENTATION
WORK BELONGS IN THE SUBJECT DOCUMENT OR GIT HISTORY, NOT IN THIS TODO LIST.

## 1. CURRENT CONSISTENCY RULES

- KEEP THE 32K-WORD PDP-6 MINIMUM TARGET EXPLICIT WHERE MACHINE ASSUMPTIONS
  MATTER.
- RECORD KCC/DAS AS THE NORMAL DAIMOS COMPILER/ASSEMBLER PATH.  GCC IS A
  COMPARISON AND PORTING TOOL, NOT THE DAIMOS ACCEPTANCE COMPILER.
- KEEP INIT -> LOGIN -> CONFIGURED SHELL AS THE NORMAL MULTIUSER BOOT MODEL.
- KEEP /MONITOR/PROCESSES, /MONITOR/DOMAINS, /MONITOR/DEVICES, THE PROCESS-ONLY
  /PROC ALIAS, AND THE I/O-ONLY /DEV VIEW IN LOCKSTEP WITH THE IMPLEMENTATION.
  DO NOT REINTRODUCE /DOMAIN OR /DEVICE ROOT TREES.
- MIGRATE NEW AUXILIARY STORAGE TO THE BSTORE/AUXSTORE DESIGN IN
  KERNEL/BSTORE.MD: ONE BACKSTORE BITMAP FOR PROCESS SWAP + MEMFS, ONE GENERIC
  CLEAN TWO-BLOCK BCACHE L1 FOR D6FS + DTFS/TSFS, OPTIONAL DIRECT-MAPPED
  CACHESTORE L2 FOR SLOW MEDIA, AND A FIXED LOGSTORE REGION.  KEEP BLOCKSET
  PHYSICAL MAPPING AND DEVICE SCHEDULING SEPARATE.  RETAIN OLD D6FS-OWNED
  SWAP/LOGSTORE RESERVATIONS ONLY AS MIGRATION COMPATIBILITY.
- KEEP THE APR 60 HZ LINE CLOCK AS MONOTONIC TIME AND THE PHIL PETIT PCLK AT
  DEVICE 0730 AS THE STANDARD WALL-CLOCK SOURCE.  SERIAL RTC/RTCD IS NOT A
  CURRENT DAIMOS PATH.
- KEEP RECOVERY/SINGLE-USER IMPLEMENTATION POST-0.9 AND BASED ON NORMAL
  MAINTENANCE TOOLS/BOOTABLE RECOVERY MEDIA, NOT AN INITFS ROOT.

## 2. BOOT/TESTKIT STATUS DOCUMENTATION

THE PRE-DSH ABI AND FILESYSTEM PREREQUISITES ARE IMPLEMENTED.  SUBSTANTIAL
DSH WORK IS ACTIVE, NOT BLOCKED BY A HISTORICAL PRE-DSH GATE.  KEEP THE
FOLLOWING ACCEPTANCE AND STORAGE STATUS ACCURATE AS TESTS CHANGE:

- BOOT-SOURCE/ROOT-CLASS MATRIX, AUTO ROOT PRIORITY, AND LOGIN/LOGOUT/RESPAWN
  TESTS ARE LANDED; DO NOT CONFUSE TEST EXISTENCE WITH A FRESH GREEN RUN.
- THE 2026-09-30 SERIAL RUN TIMED OUT IN AUTO-ROOT ALL-MEDIA AFTER EXPLICIT
  ROOT CASES; RETAIN THIS AS A DATED TEST RESULT, NOT CURRENT DSH STATUS.
- INIT OWNS SWAP/LOGSTORE AUTO/OFF AND OPTIONAL FSTAB MOUNT POLICY; KINIT
  RETAINS DEVICE DISCOVERY AND RESIDENT SERVICE PREPARATION.
- DEMAND-BACKED MEMFS IS IMPLEMENTED; KEEP ITS BACKSTORE/RECLAIM DESCRIPTION
  CONSISTENT WITH CURRENT SOURCE WHILE BSTORE MIGRATION PROCEEDS.

UPDATE BOOT, SELF-HOSTING, MEMORY, AND STATUS DOCUMENTATION WHEN THE
CORRESPONDING IMPLEMENTATION OR TEST RESULTS CHANGE.

## 3. STORAGE AND FILESYSTEM DOCUMENTATION

- KEEP KERNEL/D6FS.MD, KERNEL/BLOCKSET.MD, KERNEL/LOGSTORE.MD, KERNEL/DSK270.MD,
  AND KERNEL/DRM236.MD ALIGNED WITH THE IMPLEMENTED D6FS-OWNED RESERVATION,
  GENERIC BACKING, BLOCKSET, BADMAP, AND DEVICE-SCHEDULING CONTRACTS.
- KEEP KERNEL/BSTORE.MD AS THE TARGET DESIGN/IMPLEMENTATION PLAN UNTIL THE
  MIGRATION IS COMPLETE.  UPDATE D6FS/DTFS/MEMORY/STATUS CURRENT-STATE TEXT IN
  THE SAME CHANGE AS EACH IMPLEMENTATION STAGE; DO NOT CLAIM THE TARGET STATE
  HAS LANDED BEFORE ITS TEST AND PERMANENT-SIZE GATES PASS.
- DOCUMENT A CRASH-SAFE ONLINE BAD-MEDIA REMAP/PUBLICATION PROTOCOL ONLY WHEN
  IT IS IMPLEMENTED.  UNTIL THEN, NEW MEDIA DEFECTS REMAIN MAINTENANCE EVENTS
  AFTER ERROR REPORTING.
- KEEP CURRENT SOURCE STATUS DISTINCT FROM THE UNIFIED TSFS TARGET IN
  `KERNEL/DECTAPE.MD`: THE CURRENT RESIDENT TSFS STILL USES D6LZ36 AND THE
  CURRENT DTFS/TSFS FORMATS ARE DIFFERENT. NEW DESIGN AND MEDIA MUST USE THE
  GROUPED D6CODEC, WITH NO SECOND TAPE-SPECIFIC DECODER OR FOREIGN PERSONALITY.
- PROTOTYPE PACKED ONE-BLOCK METADATA PAGES, SIMPLE RW EXTENTS, SPARSE
  REPLICATION FOR RO MEDIA, AND OFFLINE LOCALITY OPTIMIZATION. PROVE REAL
  PARALLEL TRANSPORT MOTION BEFORE ADDING RESIDENT PREPOSITIONING COMPLEXITY;
  SERIALIZE TYPE-136 TRANSFERS AND REUSE THE SHARED BLOCK WORKSPACE.
- CLOSE THE RW CRASH-DURABILITY, MTIME, PERMISSION-CLASS AND PERMANENT-RAM
  LIMIT DECISIONS BEFORE LOCKING THE FORMAT. NEW SOURCE AND TEST NAMES SHOULD
  BE UNVERSIONED; UPDATE ABI REFERENCES TOGETHER WHEN THE CODE CUTOVER LANDS.
- MAKE TSFS ROOT UNCONDITIONALLY RO. SUPPORT EXPLICIT BUILD-TIME
  `TAPE_AUX=none|drum|disk` AND `TAPE_TMPFS=0|1` SELECTION. THE 32K-64K
  `ROOT=tape TAPE_AUX=none TAPE_TMPFS=0` PROFILE EXCLUDES SWAP, BACKSTORE,
  LOGSTORE, AUXSTORE AND MEMFS/TMPFS MODULES AND ASSOCIATED STATE/INIT FROM
  THE LINK. DO NOT AUTO-ENABLE SERVICES FROM DISCOVERED DISK/DRUM DEVICES;
  IF CONFIGURED BACKING IS MISSING, FAIL EXPLICITLY. A SECONDARY SINGLE-TAPE
  RW TSFS MOUNT MUST NOT IMPLICITLY ACTIVATE ANY AUXILIARY SERVICE.
- KEEP TRUE DSK270 MULTI-SECTOR STREAMING, TSFS PARITY/STRIPING, AND TRACE-
  DRIVEN PLACEMENT CLEARLY MARKED DEFERRED UNTIL MEASUREMENTS JUSTIFY THEM.
- KEEP BOOT/RESCUE.MD ON CURRENT DAS/DXR, MKDSK, MKD6FS, AND DEVICE-SPECIFIC
  STAGE1 ARTIFACTS; DO NOT RESTORE REMOVED PYTHON IMAGE-BUILDER WORKFLOWS.

## 4. MEMORY, MODULE, AND OVERLAY DOCUMENTATION

- KEEP KERNEL/MEMORY-MANAGEMENT.MD EXPLICIT THAT CURRENT COMPACTION DIRECTLY
  SLIDES PROCESS EXTENTS AND DOES NOT YET INTEGRATE RUNTIME MODULE MOVEMENT.
- KEEP USERSPACE OVERLAYS DEFERRED UNTIL REAL DSH/MAKE/DAS/DLINK/EDITOR/KCC
  WORKING SETS ARE MEASURED.
- KEEP LOADABLE/UNLOADABLE/RELOCATABLE RUNTIME MODULE INTEGRATION AFTER THE
  OVERLAY DECISION.  DO NOT DOCUMENT THE EXISTING RELOCATION REFERENCE CODE AS
  AN ACTIVE RESIDENT MODULE LIFECYCLE.
- KEEP A LATER MACHINE-SPECIFIC HARDWARE-PAGER BACKEND SEPARATE FROM THE PDP-6
  CONTIGUOUS-EXTENT VM BACKEND.

## 5. USERSPACE AND TOOLCHAIN DOCUMENTATION

- KEEP USER/DSH.MD AND USER/DSH_DESIGN.MD ALIGNED WITH THE COUNTED-SIXBIT
  RUN/EXEC ABI, PIPE/JOB-CONTROL SYSCALLS, TTY MODES, COMPLETION/MANUAL
  METADATA, AND THE FROZEN INITIAL DSH LANGUAGE CONTRACT.
- KEEP KERNEL/TTY.MD ALIGNED WITH THE IMPLEMENTED CANONICAL/ECHO/SIGNAL MODE;
  DO NOT EXPAND IT INTO FULL TERMIOS WITHOUT A REAL PORT REQUIREMENT.
- KEEP CLOCK/D6FS/VFS STAT/TOUCH/DATE/MAKE DOCUMENTATION ALIGNED WITH THE
  TIME36/MTIME CONTRACT.
- KEEP GCC PORTING/CLEANUP DOCUMENTS AS TOOLCHAIN REFERENCE ONLY.  KCC/DAS
  CORRECTNESS DEFECTS DISCOVERED DURING DAIMOS WORK TAKE PRIORITY.
- KEEP DAIMOS HOST/NATIVE TOOL DOCUMENTATION ALIGNED WITH DOBJ1/DARC1/DXR, P10FOLD,
  P10SUPER, MKTSFS/TSFSCHECK, AND CURRENT MEDIA-BUILDER BEHAVIOR.

## 6. TEST/DOCUMENTATION MAINTENANCE

- ADD OR UPDATE DURABLE DESIGN DOCUMENTATION WHEN AN IMPLEMENTED ABI OR FORMAT
  CHANGES.  DO NOT ADD TRANSIENT HANDOFF DOCUMENTS TO PDP10-DOC.
- REMOVE ORPHANED DOCUMENTS WHOSE REFERENCED SOURCE, TOOL, AND BUILD TARGET NO
  LONGER EXIST; GIT HISTORY IS THE ARCHIVE FOR SUPERSEDED WORKFLOWS.
