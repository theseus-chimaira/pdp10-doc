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

## 2026-11-11 v1502: current KCORE feature-completion tasks

Native immediate-TERM/finalizer v1500 and deferred owner-return
v1501 both PASS, with real production event code and controlled
backend; v1498 positive/negative also re-PASS. The v1499 physical
branch runner only PREPARES. Do not claim corrupt-BACKSTORE physical
acceptance or assume that its third free is an idle PREVM retry.

- [ ] F2: physically classify the two TERM decisions using v1499
  map-backed WAIT/PC/FD-control/owner snapshots.
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
