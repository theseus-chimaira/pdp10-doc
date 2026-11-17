## v1654: correct PURE FULL MM commit and defer 32K acceptance (2026-11-17)

- [x] Correct original opt-in PDP-6 v1391 FULL:
  destination unpin checked BEFORE freeing the
  PACKED RAW source and before RESIDENT publication.
  The small destructive commit is PI-atomic, with
  no device I/O/allocation under PI disable.
  One-shot refusal exact rollback/retry PASS; unknown
  permanent pins remain transition-held and fail closed.
- [x] Native v1395/v1397/v1648 and new permanent
  source/target unpin cases PASS; executed old-order
  target-unpin mutant fails the native MM oracle.
  New original physical 48K PDP-6 DTC3 failed read
  confirms dual pin, source preservation, full target
  rollback, same-reel subsequent FULL, six exact
  WAIT/EXIT and no MM child leak. Two forged
  physical witnesses rejected. Fixed KCORE
  unchanged; optional MRES transient +8 words.
  KERNEL/PURE-FULL-COMMIT-V1654.MD.
- [x] **DEFER 32K strict boot and footprint acceptance**
  until PDP-6 functional KCORE freeze, preserving
  v1653 6289-word shrink goal and full stack/MRES.
- [ ] Finish complete transient RAM accounting, immutable
  fail-closed MM/PI owner acceptance, optional mutable
  compression measured yes/no, stale standalone DTFS
  fixture, source-matched functional release freeze.
  KA10/KI10 ports remain distinct later tasks.
  No blanket KCORE completion or 32K acceptance claim.

## v1653: original 32K PURE ON/OFF KINIT size refusal (2026-11-17)

- [x] Original minimal C89 PURE self-RUN/
  EXIT/WAIT DXR compiled. Matching real
  original 32K lowmem KINIT PURE ON/OFF
  links have byte-identical KCORE and
  fixed low end024253. Host mkimage
  correctly refuses BOTH images before
  actual boot.
- [x] Original core100000 octal,
  reserve0400, safe end077400. OFF
  end110052 exceeds physical core by
  4,138 decimal, safe end by 4,394.
  ON end113621 exceeds core by 6,033,
  safe end by 6,289. PURE only adds
  03547 octal=1,895 transient words;
  OFF baseline is already too large.
  All three false map/log/KCORE witnesses
  rejected. See
  KERNEL/PURE-32K-KINIT-CAPACITY-V1653.MD.
- [ ] Correct genuine KINIT transient
  footprint without suppressing required
  modules or reducing the stack reserve.
  Rebuild and actually boot/run 32K PURE
  before claiming acceptance. Do not
  mislabel size refusal as runtime/MM bug.
  P1-E OPEN, PURE default OFF,
  strict 32K optimization HOLD.

## v1652: independent native PDP-6 64K nine-PACKED profile (2026-11-17)

- [x] Actual original 64K no-AUX/TSFS/secondary-DTFS
  physical guest source-derived from v1643, nine
  admitted RUNs, refused tenth, compressed DXR2
  source match, exact original DATA/PIPE/EXIT057/
  WAIT and normal HALT. Preliminary 10-child
  over-capacity test refused as expected.
- [x] Original source-linked B/A/Z MM replay:
  arena octal 042601..200000, nine unique pin0
  type4 PACKED_PROCESS slots2..10, suffix006600,
  B/A free9404 decimal/maxgap4155. Z zero child
  MM owners, free42769. Three physically falsified
  type/owner/WAIT cases rejected. See
  KERNEL/PURE-64K-OWNERS-V1652.MD.
- [ ] Matched 64K PURE ON/OFF, continuous FULL peak,
  32K original profile, uncertain MM owner recovery,
  and KA10 VM still OPEN. No production change;
  PURE OFF, strict 32K optimization HOLD.

## v1651: original 48K physical MM snapshot accounting (2026-11-17)

- [x] Original v1643 accepted source-linked 48K physical
  secondary-DTFS read failure, later FULL and all EXIT/WAIT
  reaudited PASS. Six original managed MM snapshot totals
  validated against one 31,375-word arena, exact type/
  owner/pin geometry and complete occupied/free accounting.
- [x] B/A occupied27,163/free4,212; FULL_PINNED27,668/
  free3,707; failed FULL rollback23,572/free7,803;
  LATER_FULL27,668/free3,707; final Z4,958/free26,417.
  Largest FULL contiguous gap 883 words, only 640 after
  02000 APR alignment. Full checkpoint net +505 words
  reflects a 4096-word APR, prior child -3456 suffix
  and a -135-word dynamic MM release. Three forged
  arena/dynamic/target-pin witnesses all rejected.
  `KERNEL/PURE-48K-MM-SNAPSHOT-V1651.MD`.
- [ ] Still OPEN: continuous whole-system physical peak
  including MRES/KCORE/provider scratch; matched 48K
  PURE ON/OFF for this larger source; 32K/64K layouts,
  uncertain MM ownership and distinct KA10 VM.
  No production/RAM change. PURE OFF, strict 32K HOLD.

## v1650: native slot-0 FULL scheduler exclusion (2026-11-17)

- [x] Original PDP-6 PI6 idle-PC interval and swap ownership
  verified by eleven native cases, including positive idle
  dispatch. All three unsafe clock-path mutations rejected.
- [x] Existing original physical v1643 48K source-linked
  trace/map confirms slot0 bounded FULL at 035252 outside
  idle017216..017264, with real pin1 RAW and full APR.
  Both forged slot/pin physical witnesses rejected; original
  read-error/rollback/recovery audit rerun and passes.
  This is reuse of an accepted physical trace, not a new run.
  `KERNEL/PURE-SLOT0-SERIAL-V1650.MD`.
- [x] Close impossible independent-user UNMOUNT-during-FULL
  interleaving under existing scheduler; no test-only bypass.
- [ ] P1-E remains OPEN for real device/media failure,
  permanently uncertain MM ownership, 32K/48K/64K
  physical transient RAM peak and separate KA10 backend.
  No production/RAM change; PURE OFF, optimization HOLD.

## v1649: original executable owner guard in FULL two-pin callback (2026-11-17)

- [x] Native PDP-6 real compiled v1391 FULL callback has
  simultaneously pinned original RAW PACKED and separate
  complete PROCESS APR, with exact transition/backing.
  Original source-extracted assembler guards refuse owning
  DTFS mount and executable vnode, but not unrelated sources.
  Original descriptors and backing remain unchanged; full
  restoration succeeds with exact writable suffix and text.
- [x] Two independently compiled invalid guard predicates
  execute and fail at octal 002002. Existing v1563 ownership
  tests pass 26 native cases and reject six mutants.
  `KERNEL/PURE-FULL-MOUNT-GUARD-V1649.MD`.
- [ ] Physical independent UNMOUNT actor under real PI6 during
  blocked FULL provider I/O, uncertain MM ownership, total
  transient peak RAM and 32K/48K/64K/KA10 profiles remain OPEN.
  No production changes; PURE OFF, strict 32K optimization HOLD.

## v1648: native FULL MM source-release and destination-pin rollback (2026-11-17)

- [x] Real compiled v1391 FULL/PDP-6 MM transaction safely
  rejects PACKED source-free BUSY/INVAL and new full APR pin
  BUSY. Existing RAW contents, backing, one unpinned MM owner
  and TRANSITION stay intact, temporary PROCESS allocation is
  gone, and a later retry restores the exact complete user image.
- [x] Both executed invalid-code controls fail as required:
  missing RAW overlay `0706`, leaked target `01004` (octal).
  Native v1395/v1397 rollback and v1611/v1612 service regressions
  PASS. No production KCORE/VM/MM/ABI or resident data changes.
  `KERNEL/PURE-FULL-ATOMIC-V1648.MD`.
- [ ] Physical pinned FULL reload vs actual concurrent DTFS
  source I/O/mutation/unmount, ambiguous permanent MM ownership,
  complete transient peak RAM in 32K/48K/64K and a real KA10
  VM backend remain open. PURE default OFF, 32K optimize HOLD.

## v1647: physical P1-E DTFS V2 quiescence / new PURE RUN PASS

- [x] Original PDP-6 48K source-built real DTVOL2/CLEAN RW
  secondary DTFS mount and compressed DXR2 PURE new RUN.
  Original FSINFO -1 barrier refuses RUN with exact original
  MM/backing intact. Original PI6 CLK advances 1004 ticks;
  unmount, same-reel remount, successful RUN/EXIT/WAIT and
  final no-MM-owner HALT PASS. Seven physical invalid-volume
  cases and three forged clock/backing/RO witnesses rejected.
  Disposable kernel only adds real test scheduling interval;
  no production edit or permanent RAM. Both initial children
  are RESIDENT, not PACKED, and the result is an admission
  test, not a new compression/RAM-saving result. See
  `KERNEL/PURE-DTFS-QUIESCE-V1647.MD`.
- [ ] P1-E still OPEN for simultaneous pinned FULL and DTFS
  I/O/concurrent mutation, uncertain MM rollback, 32K/64K,
  transient peak RAM, and KA10 VM acceptance. PURE OFF by
  default, strict 32K optimization HOLD.

## v1645: physical PI6 staged wake with five surviving PACKED owners (2026-11-17)

- [x] Source-derived original PDP-6 48K no-AUX/TSFS/PURE guest;
  real native DTC3 RW executable mode0777, six sleeping PACKED
  children with original source-sized `006400` octal suffixes
  (compiled TX2 text001454, omit001400). Original parent
  passes all v1644 RW executable mutation/unmount checks.
- [x] Parent writes ONE shared PIPE token (`H`) while keeping
  writer open and repeats original RW source guards. Real
  `dsys_sleep(0200)` yields via original PI6; precisely one
  child completes DATA/PIPE/EXIT-057 (`w`) before `T`, five
  still PACKED on exact original extents and pin0. Another
  RW source guard passes before closing the writer.
- [x] Source-linked physical CLK PI6 and `proc_timer_clock` both
  advance 1691 real ticks between H=8673 and T=10364.
  Post-H guard at tick9098 still sees six matching RAW owners;
  post-T guard at tick11015 sees EXACTLY FIVE matching owners.
  Root remains TSFS provider7/mount1 and secondary DTC3 mount2
  is RW (VFS policy000000000001). Original mount/vnode guard
  PCs003431/003322 entered with correct mount2/file slot0.
- [x] Remaining five EOF/exit and six exact PID/WAIT/status-057
  checks PASS, native DTC3 unmount succeeds, normal HALT and
  physical U/Z MM contains zero child owners. Console ends
  `LHPwTPwwwwwUZ`. Source-linked physical auditor and forged
  clock/source pin/RO-policy controls PASS/reject. Native PI6
  v1086, staged PURE v1563 and rename/PI6 v468 PASS. No
  production KCORE/MM/VFS/PI6 edit, no added resident RAM.
- [ ] OPEN: true overlapping sleeping DTFS WRITE/UNLINK/UNMOUNT
  vs new PURE RUN/EXEC backing publication and simultaneously
  pinned FULL target; uncertain MM error rollback, general
  32K/64K layout and matched transient physical peak RAM.
  Older DTFS v1522 source-only fixture remains stale, NOT PASS.
  PURE default OFF; strict 32K whole-KCORE optimization HOLD.

## v1644: real physical RW executable content and mount guards (2026-11-17)

- [x] Original PDP-6 48K TSFS/no-AUX/PURE user guest source-built
  with a separate genuine DTC3 native DTFS RW mount2, executable
  `/MOUNT/COMP` mode0777, root DTC0..2 provider7/mount1 online.
  Six sleeping PACKED child suffixes have 006400 octal RAW words
  (compiled TX2 text001420, skip001400) and no pins at B/A/
  guard-entry/P. Same original sorted physical MM owners throughout.
- [x] Executed original `proc_swap_mount_busy` PC003431 (AC1=2)
  and `proc_swap_backing_busy` PC003322 (AC1=050202000000:
  DTFS provider5, public mount2, file slot0) while six PACKED.
  VFS policy `000000000001` independently proves DTC3 writable.
  Genuine UNMOUNT, UNLINK, TRUNCATE, O_TRUNC and WRITE refused.
  In-place native RENAME COMP->SAFE->COMP allowed (same underlying
  slot), with UNLINK SAFE still refused; no immutable data change.
- [x] All original six PIPE/private mutable sentinel/EXIT-057
  and PID/WAIT/status-057 checks pass. Only AFTER final child
  teardown can parent UNMOUNT the DTC3 filesystem successfully;
  final `LwwwwwwUZ`, normal HALT, U/Z physical MM zero child owners.
  Source-linked audit and forged RO-policy, RAW pin, other mount
  reject. v1382/v1563 native backing tests PASS; no KCORE edit.
- [ ] Still OPEN: concurrent PI6 source-mutation/retirement races,
  uncertain MM failure cleanup, 32K/64K profiles and matched
  transient physical RAM peak. Previous native DTFS v1522 source
  fixture remains stale after v1575/v1561 (NOT a current PASS).
  PURE default OFF; strict 32K whole-KCORE optimization HOLD.

## v1643: recoverable REAL secondary DTFS FULL read loss (2026-11-17)

- [x] Source-derived original 48K PDP-6 no-AUX TSFS/PURE guest
  runs one plain child from selected TSFS root, five compressed
  PURE children from native DTC3 DTFS public mount2, after actual
  RW file copy, CLEAN owner-safe unmount and RO remount. Root
  DTC0..2/provider7/mount1 remains unchanged. Six actual
  unpinned PACKED owners (2..7), new guest TX2 text 001227,
  skipped 001200, retained 006600-octal suffix each. Honest
  seventh RUN refusal, unchanged PIPE/BRK/WAIT assertions.
- [x] Original physical v1391 bounded FULL PC 035252: slot3
  pinned RAW 111200..120000 plus independently pinned PROCESS
  APR 120000..130000. Real physical DETACH ONLY secondary DTC3
  returns original -4 at service PC 032461: same RAW owner pin0,
  full destination freed. Reattach exact same DTC3 reel: later
  original compressed FULL callback slot4 returns 0. All six
  PIPE/private data/EXIT-057 and exact PID/WAIT/status-057
  outcomes pass, `LwwwwwwZ`/HALT, Z no child owner. Source/real
  MM/original DXR audit + two forged pin/success traces PASS/
  reject. Unmodified production source; v1395/v1612 native PASS.
- [ ] Immutable backing rename/remove/unmount protection while
  PACKED, concurrency PI6/uncertain MM errors, full temporal
  physical peak RAM and 32K/64K cache regressions. The old
  DTFS v1522 standalone testkit must be separately updated for
  v1575 source anchor and v1561 original write-gate linkage;
  it does NOT currently pass on the newest combined source.
  The actual READ_SUSPECT media bit was not sampled directly.
  PURE default OFF; strict 32K whole-KCORE size HOLD.

## v1642: physical selected-root TSFS FULL read fail-stop (2026-11-17)

- [x] Original source-backed PDP-6 48K TSFS/no-AUX/PURE guest
  stops at v1391 bounded FULL PC 035252 with concurrently pinned
  compressed slot3 PACKED suffix 111000..120000 and separate
  PROCESS full target 120000..130000 octal. Physically detach
  all three selected root Type-551 members DTC0/1/2 and verify
  missing device attachments before resuming original executable.
- [x] Actual TSFS read error reaches KCORE root classifier
  PC 003761, provider 7, public mount 1 equal namespace ROOT;
  terminal root-specific HALT PC 003776 with original MM source
  and target still owned. No false child WAIT/normal FULL rollback
  claims. Source-linked MM/guest/TSFS audit PASSes, rejecting
  falsified other-mount and wrong-HALT machine witnesses.
- [ ] Test recoverable physical FULL backing read failure on a
  SECONDARY non-root filesystem, where existing v1538 policy
  returns an error instead of halting. Immutable source guards,
  PI6 and uncertain MM errors, 32K/64K BCACHE profiles and
  matched transient peak RAM remain OPEN. PURE default OFF;
  strict 32K whole-KCORE optimization HOLD.

## v1641: test-injected FULL loader error and physical MM rollback (2026-11-17)

- [x] Original source-built PDP-6 48K no-AUX TSFS v1638
  compressed PURE guest, MRES and backing unchanged. At original
  v1391 post-loader callback PC 034361, real loader returned
  AC1=0; slot3 owns separate pinned PACKED source
  111000..120000 and full PROCESS target 120000..130000.
  Testkit SIMH changes ONLY AC1 to -1 once at this boundary;
  this is NOT a genuine VFS or DECtape failed read.
- [x] Original v1391 returns -4 at PURE service PC 032461;
  slot3 PACKED source remains exact and unpinned, and its
  temporary full target was freed. Next observed original FULL
  callback for compressed slot4 returns success. All six child
  PIPE/mutable data/EXIT-057 and exact parent WAIT checks pass,
  final Z/HALT and zero child PROCESS/PACKED MM owners. Source-
  linked physical audit and two forged pin/loader-return traces
  PASS/reject; no permanent production code change.
- [ ] Actual VFS failure, source removal/unmount and immutable
  backing lifetime; PI6/uncertain MM failure rollback; broad
  32K/64K low-BCACHE layouts and complete matched physical
  transient peak RAM. PURE OFF; strict 32K size HOLD.

## v1640: real physical FULL allocation NOMEM and retry (2026-11-17)

- [x] Source-built PDP-6 48K TSFS/no-AUX six-PACKED guest:
  first compressed PURE child, second plain PURE child, four
  other compressed children. Source-linked v1391 MM allocation
  return PC 034336: genuine -1 NOMEM, slot2 retains its single
  pinned PACKED suffix 121000..130000, no destination APR.
- [x] Actual v1391->v1612 return PC 032461: same six original
  PACKED owners, all zero pins, -1 return preserved. Later SAME
  slot2 at bounded FULL loader PC 035252: pinned PACKED source
  121000..130000 plus independently pinned PROCESS destination
  110000..120000, AC4 010000. Entire six-child original PIPE/
  mutable data/EXIT-057 and parent WAIT statuses pass, HALT;
  final physical MM Z no child owner. Source-linked and two
  falsified owner-pin/return machine witnesses PASS/reject.
- [ ] Actual physical failed VFS reads/source loss/unmount;
  concurrent PI6 and uncertain MM rollback, 32K/64K general
  BCACHE regressions and complete matched transient peak RAM.
  PURE default OFF; strict 32K whole-KCORE optimization HOLD.

## v1639: matched physical 48K six-owner PURE RAM result (2026-11-17)

- [x] Source-identical ON/OFF PDP-6 48K TSFS/no-AUX C89 guest,
  executable INIT DXR and fixed KCORE map, both admit six children,
  refuse seventh RUN, then verify six original PIPE/private-data/
  EXIT-057/exact PID/WAIT status checks and normal HALT.
- [x] Original source-linked MM B/A: ON six unpinned PACKED suffix
  owners, occupied/free 066433/006564 octal; OFF six full resident
  process APRs, occupied/free 074433/004122. ON +1314 decimal
  physical free words at equal six-owner state after including
  1758 decimal installed PURE MRES. Six omitted 512-word prefixes
  gross 3072; net sample 3072 - 1758 = 1314. Final Z both
  child-owner free, but ON now -1887 decimal free due MRES plus
  129 extra valid BCACHE words at ON-only terminal cache state.
  Forged process owner and false parent-WAIT controls rejected.
- [ ] Still need matched TRANSIENT physical peak while FULL v1391
  holds source + destination simultaneously; source/unmount and
  PI6/MM failure-atomic rollback, 32K/64K BCACHE regression,
  P1-E feature acceptance. PURE OFF, strict 32K size HOLD.

## v1638: physical compressed PURE FULL relocated v1391 (2026-11-17)

- [x] Source-matched 48K PDP-6 TSFS/no-AUX/PURE with six
  children: first plain PURE, five compressed PURE. Seventh RUN
  refuses, parent survives and checks all original six child
  PIPE/data/EXIT-057 and exact PID/EXITED/status-057 WAITs,
  terminal `LwwwwwwZ`, normal simulator HALT.
- [x] Physical B/A: six distinct unpinned PACKED suffixes,
  each 007000 words. Original linked FULL v1391 loader breakpoint
  PC 035252 with authenticated opcode and slot-zero scheduler,
  AC2 target 120000, AC4 010000 capacity. At this instruction
  child slot 3 simultaneously owns pinned PACKED suffix
  111000..120000 and separate pinned full PROCESS APR
  120000..130000. After all six WAITs no child PROCESS/PACKED
  owners remain; forged target-owner and false-WAIT controls
  rejected. v1630 FAST and v1638 FULL success are both proven.
- [ ] Physical source-loss/unmount and executable backing guards;
  interrupted FULL/PI6/MM failure rollback and concurrency;
  32K/64K BCACHE layout regressions; matched physical net/peak
  RAM and full P1-E feature acceptance. PURE default OFF;
  strict 32K whole-KCORE size pass still HOLD.

## v1637: six physical 48K PACKED restores after seventh RUN (2026-11-16)

- [x] Original source-built PDP-6 48K no-AUX TSFS/PURE kernel
  admits six sleeping children with low BCACHE; the seventh RUN
  is refused, parent survives and completes six exact child
  PIPE/data/EXIT-057/WAIT statuses and normal HALT (`LwwwwwwZ`).
- [x] Same guest original physical MM B/A each retain six distinct
  unpinned 007000-word PACKED suffixes, slots 7..2; parent at
  130000..140000 octal. Final Z has no child PACKED or PROCESS
  owner. Foreign-owner and false-WAIT controls rejected.
- [ ] Forced relocated FULL restore, immutable-source/unmount
  failures, concurrent PI6/MM rollback, matched physical peak/net
  RAM and 32K/64K cache regressions remain open. PURE default OFF;
  strict 32K optimization HOLD.

## v1636: low BCACHE restores six 48K user APR admissions (2026-11-16)

- [x] Correct BCACHE 0201-word private clean slab allocation
  preference HIGH -> LOW while preserving owner6, no-reclaim
  allocation, source tags, invalidation, and MM free logic.
  Actual 48K PDP-6 TSFS/no-AUX/PURE guest admits six children,
  parent no-growth BRK and exact six original PIPE/data/EXIT-057
  and PID/EXITED/status-057 WAIT checks; normal simulator HALT.
- [x] Source-matched physical MM B/A/Z snapshots: parent now at
  `130000..140000` octal, formerly `126000..136000`; BCACHE at
  low memory, six full PROCESS owners at B/A, none after WAIT.
  Cache behavior regression and forged MM parent/WAIT witnesses
  PASS/reject. This closes earlier sixth-RUN capacity regression.
- [ ] Validate genuine PACKED restore under seventh-RUN pressure
  because six children remained resident in this test. Finish
  32K/64K, source lifetime, full restore/PI6 failure/rollback
  and physical RAM peak/net gates before final feature acceptance.

## v1635: first-VM BCACHE high-memory obstruction (2026-11-16)

- [x] Original source-matched 48K PDP-6 first-VM physical MM
  breakpoint proves BCACHE's 0201-word high-allocation owner6
  `137577..140000` is present before parent APR creation. Parent
  VM request 007067 -> 010000 APR rounds highest legal address
  down to 126000; later parent MM extent confirms that base.
  Without the high cache slab, top base 130000 is feasible in
  the same arena. Forged owner/size witnesses rejected.
- [ ] Prove any low-placed clean-cache policy in a real 48K PURE
  guest with six admitted children, actual PACKED recovery and
  all WAIT statuses. Verify cache contents, reclaim and 32K/64K
  regressions. Do not override RUN failure or move live parent.

## v1634: PURE MRES compiler flag comparison (2026-11-16)

- [x] Original GCC 3.2 -Os PURE MRES rebuilt byte-identically
  at 003336 octal/1758 decimal resident words with exact PDP-6
  DAS -F flags and source/KCORE map. Eight compiler choices
  examined in testkit; best saved only 1 word, not the required
  389-word 48K APR-alignment improvement. No new flags accepted.
- [ ] Correct source-level MRES consolidation, or a different
  proven physical APR layout, must retain fast/FULL restore and
  source/PI6/MM rollback; prove any candidate with real guests.
  Default PURE OFF, 32K whole-KCORE optimization HOLD.

## v1633: original 48K APR corridor lower bound (2026-11-16)

- [x] Physical source-matched MM table: PURE-on low-core dynamic
  high=046605, PURE-off=043247, installed PURE MRES 003336
  octal/1758 decimal words. Fixed parent 126000..136000;
  six full child APRs require dynamic high <=046000. PURE-on
  needs at least 000605 octal/389 decimal words of genuine
  low-core improvement. Testkit rejects false dynamic/parent
  extent geometry. This is a bound, not a demonstrated fix.
- [ ] Physically validate any correctness-preserving MRES shrink
  or alternate APR layout; retain FULL restoration and rollback.
  PACKED suffix relocation alone cannot fit six simultaneously
  runnable full APRs in the existing fixed-parent corridor.
- [ ] Complete source/unmount/PI6/full restore failure tests,
  transient peak/net RAM and P1-E feature acceptance. PURE OFF;
  strict 32K optimization HOLD.

## v1632: original five-owner physical PURE-on/OFF RAM (2026-11-16)

- [x] Source-matched 48K PDP-6 parent + five live children, both
  modes. ON five PACKED extents, physical free 016173; OFF five
  resident full APR processes, free 014330 octal. ON sampled free
  advantage +001643 octal / +931 decimal words includes installed
  1758-word PURE MRES. Original MM audit rejects two forged owner
  types/IDs. ON sample follows sixth RUN refusal, OFF precedes
  sixth RUN; not a complete matched transient peak.
- [ ] Prove correct APR placement for future RUN under RAW
  fragmentation. ON biggest free aligned gap 007000, OFF 012000;
  new process requires 010000. Do not report summed free words as
  an extra process, or introduce a retry/owner workaround.
- [ ] Physical v1391 full restore, source loss/PI6 rollback,
  complete baseline-matched net/peak RAM and full P1-E acceptance.
  PURE OFF by default; strict 32K optimization HOLD.

## v1631: matched 48K PURE-on/OFF admission regression (2026-11-16)

- [x] Same original physical PDP-6 48K no-AUX TSFS guest compiled
  byte-identical (INIT DXR and KCORE symbol map), with PURE module
  on versus off. ON five children PACK and sixth RUN refuses;
  OFF original guest prints `aaaaa!uwwwwww`: sixth RUN actually
  succeeds, parent takes intentional failed expectation, six
  children execute. Both original simulators HALT.
- [x] Source-linked OFF MM after sixth RUN: no PACKED, seven
  unpinned complete 010000 PROCESS extents contiguous 046000..
  136000, arena 037224..140000, occupied/free 074634/003720.
  ON five-PACKED case arena 042562..140000: the exact start delta
  003336 octal is 1758 decimal resident PURE MRES words. ON has
  more fragmented total free RAM but no 010000 aligned APR run
  for the sixth RUN. Forged owner/admission physical evidence
  rejected by testkit. This is an actual process-admission
  regression for this workload, not a compiler or loader bug.
- [ ] Compare live FIVE-child PURE-off/on memory snapshots at
  identical source points and calculate net/peak RAM. Keep source
  lifetime, FULL fallback/rollback and PI6 gates open. PURE
  default OFF, 32K size optimization HOLD.

## v1630: first physical v1384 in-place PURE restore accepted (2026-11-16)

- [x] Replayed source-authentic 48K PDP-6 five-PACKED-child guest,
  with installed PURE MRES instruction verification. Slot 2 reaches
  fast in-place MM CLAIM at PC 034610 (suffix 117000, owner 2,
  prefix 001000, complete APR 010000) and COMMIT at PC 034614
  (original APR 116000, owner 2, same prefix/size). No FULL
  v1391 entry before commit; all five children subsequently print
  authentic `w`, and parent completes WAIT/Z and simulator HALT.
  Source/physical testkit auditor and two false-branch/AC witnesses
  PASS/reject. Existing native v1395/v1397 MM rollback and v1612
  PURE service tests PASS. No production kernel, MRES or ABI edits.
- [ ] Physical relocated FULL v1391 and restoration rollback with
  obstructed prefix, immutable backing loss and PI6 pressure;
  whole-system PURE/no-PURE net RAM and transient peak. Keep
  default PURE OFF and strict 32K optimization HOLD.

## v1629: genuine 48K PACKED restoration and WAIT pass (2026-11-16)

- [x] Actual 48K PDP-6 TSFS/no-AUX PURE guest: five children
  admitted and sleeping, expected sixth RUN returns -1 under
  physical VM pressure, but parent survives and completes its
  original five PIPE/mutable-data/EXIT-057, PID/WAIT/EXITED/status
  checks, finishing `LwwwwwZ` with normal simulator HALT.
- [x] Same source-built guest physically replays with live MM
  owner snapshots: B/A each have five unpinned PACKED 007000-word
  suffixes (owners 6..2), 022 octal extents, occupied/free
  057023/016173; Z has 010 extents, zero PACKED/child PROCESS,
  occupied/free 011536/063460 octal. Original owner/source-linked
  auditor rejects foreign-owner and false-WAIT mutations.
- [ ] Separate fast/relocated RAW restore, source-loss/PI6/MM
  rollback and baseline-matched whole-system net/peak physical
  RAM. Do not count gross 2560 omitted words as net without
  installed 1758-word PURE MRES and transient other costs.

## v1628: 64K P1-E lifecycle did not actually PACK owners (2026-11-16)

- [x] Source-matched original v1627 64K guest replayed with actual
  physical MM snapshots at B, A and terminal parent Z. The arena
  `042602..200000` has 025/025/010 octal descriptors; occupied/
  free words `074654/040322`, `104654/030322`, `021556/113420`.
  All three stages have **zero PACKED** owners. Owner-overlap and
  missing-terminal-snapshot mutants rejected. Normal PIPE/WAIT/EXIT
  passes but is only a resident-process functional acceptance.
- [ ] Exercise five physically PACKED children on 48K after an
  expected sixth RUN resource refusal WITHOUT terminating their
  parent, then prove PACKED restoration, exact WAIT statuses and
  MM/immutable-source cleanup. Measure peak and net physical RAM.

## v1627: 64K P1-E six-child WAIT/EXIT physical acceptance (2026-11-16)

- [x] Separate 64K no-AUX physical PURE source guest accepts all
  six RUN children and completes original PIPE/private mutable
  checks, parent BRK, all six exact PID/EXITED/status-057 WAITs,
  parent data checks and terminal Z. Exact physical console ends
  `LwwwwwwZ`. L is lifecycle-only; the original BRK-time
  additional RAM savings assertion remains a distinct failing
  negative case. Two forged physical witnesses rejected.
- [ ] Sample simultaneous PACKED/PROCESS/owner/pin physical MM
  state before and after pressure and through wake/WAIT. Prove
  immutable backing, full/relocated restore and rollback, and
  measured net/peak system RAM including the PURE MRES before
  enabling P1-E or starting strict 32K size optimization.

## v1626: 48K PURE MM fragmentation and 64K BRK verified (2026-11-16)

- [x] Original 48K six-child PURE physical/MM snapshot fully decoded:
  five PACKED suffixes, each 007000 octal words; free gaps total
  016173 but the largest aligned gap is 007000. Source-matched
  PURE executable image/BSS/stack requires a complete 010000-word
  initial APR VM. Thus the sixth RUN cannot fit in the captured
  geometry. MM owner and overlap mutants rejected by testkit.
- [x] Independent genuine 64K no-AUX/PURE six-child boot admits all
  six RUNs and parent BRK 010000 -> 017000 succeeds. Exact
  MEMINFO B 073470/042522 to A 103470/042522 records only the
  ordinary 010000-word BRK allocation. Old strict additional-drop
  test correctly emits !R. Six children return after parent abort;
  complete WAIT and net RAM remain unaccepted.
- [ ] Accept full source-backed six-child PIPE/WAIT/EXIT without
  requiring a new PURE drop specifically during BRK; separately
  measure exact physical RAM and concurrent executable lifetime.

## v1625: P1-E sixth RUN VM creation failure classified (2026-11-16)

- [x] A source-built 48K PDP-6 TSFS/no-AUX RAW PURE guest with
  six requested children reports `aaaaa!rwwwww`: the sixth RUN
  returns invalid PID before parent BRK, not a PROC_SLEEP wait
  timeout (v1624). The five-child BRK 017000 experiment does not
  return successfully; no net RAM acceptance.
- [x] Physical v1625 breakpoint at original `proc_run_claimed_bad`
  PC `017725` captures AC1 -2 (`EXEC_LOAD_NOMEM`) after five
  admitted children. Original `exec_load.s` returns this only
  when `vm_space_create` fails. Process high slot 010 and capacity
  014 octal exclude simple process-table saturation. Actual
  source/linked-map audit and two corrupted physical witnesses
  PASS/reject. No production KCORE/MRES change.
- [ ] Inspect live MM free-run and aligned-extent geometry to
  classify the VM allocation refusal; establish a feasible
  48K/64K RAW pressure profile and finish parent BRK, all
  PIPE/WAIT/EXIT, immutable owner/rollback and peak/net RAM.
  P1-E stays OFF by default and is NOT yet accepted.

## v1623: DECtape hardware-retention backlog retired (2026-11-16)

The supported DTVOL2 writable policy is v1575's trusted-operator
contract: no reel exchange while a writable mount exists. Physical
Type-551 hardware has no enforceable reel-retention interlock, and
software cannot supply one. No strict P0-B hardware-retention TODO
or KCORE feature-freeze dependency remains. Dated older checklists
are historical evidence, not active implementation requests.

## Current KCORE feature increment: v1621 (2026-11-16)

- [x] **v1621 actual restored user code:**
  native PDP-6 guest prints the
  source-authentic child 'w' marker
  after slot-2 MM unpin and READY_ONCE
  current-slot selection. Original
  source requires PIPE read plus both
  writable data checks first; compiled
  PURE DXR and TSFS manifest match.
  User-signal/source mutations rejected.
- [x] **v1621 physical BRK-fit negative control:**
  four-child 48K test BRK 010000 ->
  016000 succeeds, but logical process
  words increase by exactly 006000
  octal; original '!R' correctly
  rejects this as RAW reclamation.
  Fitting BRK without pressure is
  not a memory acceptance test.
- [x] **v1621 six-child early refusal:**
  five children pass the ready check
  then !c occurs before parent BRK.
  Original code conflates invalid
  RUN pid and bounded wait-to-SLEEP
  refusal; the cause is unclassified.
  Negative-control audit and unsafe
  refusal-marker mutation PASS.
- [ ] **Still OPEN:** physical end-to-end
  full PIPE/WAIT/EXIT under genuine
  but fitting RAW pressure, immutable
  source lifetime, exact 48K/64K
  peak/net physical RAM including
  1758-word installed GCC PURE MRES.
  Investigate a five-child fitting
  profile and classify the sixth-child
  RUN-vs-SLEEP guard before changing
  production kernel or MM behavior.
  Do not default-enable P1-E.

DAIMOS source:
design/kcore-p1e-physical-user-resume-v1621.md.

- [x] **v1620 genuine MM in-place PURE
  prefix commit:** original physical
  PDP-6 reaches mm_pure_commit_prefix_v1384
  at 034614 (slot 2). MM descriptor 024307
  changes PACKED pin1 -> PROCESS pin1 ->
  PROCESS pin0, original APR 116000..126000.
  No duplicate mutable suffix allocation.
- [x] **v1620 restored scheduler selection:**
  real PURE module cursor 001002
  (READY_ONCE|2) at PC 032477 and
  fixed KCORE current slot 2 at PC 016770.
  Source-matched audits PASS; wrong-owner,
  removed initial pin, retained final pin,
  wrong READY_ONCE and wrong current-slot
  mutations are rejected. All physical
  process descriptors remain valid.
- [ ] **v1621 full acceptance remains OPEN:**
  prove actual user-mode return, complete
  PIPE/WAIT/EXIT and source lifetime;
  measure physical net/peak RAM in a fitting
  workload and with alternative memory sizes.
  Three omitted prefixes total 1536 decimal
  words against 1758 for PURE GCC MRES
  resident code; no positive net RAM yet.
  Experimental PURE stays default OFF.

DAIMOS:
design/kcore-p1e-physical-prefix-commit-v1620.md.

- [x] **v1619 root DTC0 RBN/data proof:**
  real TSFS root, read-only near
  block 000004 and long-reverse
  000341 -> 000004 native PDP-6
  source driver tests PASS. No
  physical tape media mutation.
- [x] **v1619 original full PURE read
  completion path:** PI3 matching
  RBN at 031046, read setup 031106,
  reverse-read DCT completion at
  030555; block 000004 and its
  0200-word transfer verified in
  the real 48K physical guest.
  Test-only symbol maps produce
  byte-identical original TAPE DXR.
  Executed unsafe RBN/DCT mutations
  rejected. Incorrect earlier
  PI5 completion probe corrected
  in testkit, no driver workaround.
- [ ] **Next P1-E VFS/MM boundary:**
  verify original prefix rollback
  (034612) or commit (034614),
  unpin and RESIDENT publication,
  scheduler READY_ONCE, PIPE/WAIT/EXIT,
  executable backing lifetime,
  net and peak physical memory.
  Default PURE OFF.

DAIMOS report:
design/kcore-p1e-physical-dct-read-v1619.md.

- [x] **Type-551 rewind/search timeout v1618:**
  increase original 45s to 120s (016040 octal
  60-Hz ticks), accounting for a genuine 60s
  rewind and another 60s headroom. Original
  command and PI3 mark-rearm operands change;
  linked tape module size, KCORE and per-process
  RAM unchanged. MTC 15m control and data
  transfer watchdogs unchanged.
- [x] **Physical 48K PDP-6 timeout acceptance:**
  real TSFS/no-AUX guest samples the v1618
  countdown 3 times, demonstrates 4+5 clock
  ticks and changing RBN mark data. Source
  and linked old/new DXR equivalence tests
  PASS; old timeout and mutated physical
  countdown are rejected. DTC watchdog,
  reverse write and first-mark regressions
  PASS. Historic v1617 45s snapshots are
  still verified against the old linked DXR.
- [ ] **Still OPEN:** real block match/data
  completion or bounded search failure,
  immutable-prefix commit/rollback, MM
  pin release, slot-0 READY_ONCE,
  PIPE/WAIT/EXIT, positive net/peak RAM.
  PURE stays OFF by default.

DAIMOS report:
design/kcore-p1e-dtc-timeout-v1618.md.

- [x] **Physical MRES origin v1617:** KCORE
  trampoline and linked image decode show
  PURE MRES base 032027, but PC 030762
  is Type-551 tape MRES base 030431 plus
  offset 000331 (SKIPN storage_count).
  Original source and physical opcode match.
- [x] **Live DECtape search state:** the
  actual 48K no-AUX PDP-6 stopped in RBN
  search with storage_state 0200, initial
  -05214 tick countdown, four PACKED owners
  (three unpinned RAW suffixes and one
  pinned full-prefix claim). Two unsafe
  image/timeout mutations are rejected.
- [ ] **P1-E restore completion remains OPEN:**
  verify storage_clock_tick and PI3 mark
  progress or bounded tape I/O error,
  prefix commit/rollback and pin release,
  scheduler RUN/PIPE/WAIT, source lifetime,
  peak/net physical RAM. No code or module
  growth, no driver workaround; default OFF.

DAIMOS source report:
design/kcore-p1e-physical-dtc-wait-v1617.md.

- [x] **Real 48K physical RAW PACKED publication:**
  two PACKED owners in interrupted MM snapshot,
  one unpinned 007000-word suffix, another
  full-size 010000-word owner with one pin.
  The latter is an allowed intermediate v1384
  VFS-prefix-claim state, not proof of a leak.
- [x] **Real source-matched BRK debug:** CPU entry
  breakpoint hit at 022104; console EXPECT
  captured five resident owner extents immediately
  before 020000 BRK. Deferred BRK return address
  020761 verified from linked DXR, physical
  before/after snapshot audit PASS.
- [ ] **Correctness/acceptance remains OPEN:**
  post-BRK return trap was not hit before timeout;
  no AC1/AC2 cause confirmed and no completed
  slot-0 restore or PIPE/WAIT. Next observe
  the exact immutable-prefix reload commit or
  rollback boundary and final MM pin release,
  then test physical net/peak RAM and source
  lifetime. Experimental PURE stays default OFF.

DAIMOS detail:
design/kcore-p1e-physical-packed-v1616.md.

- [x] **v1615 physical MM placement baseline:**
  real 48K PURE guest, controlled BRK `014000`
  and actual MM table showing five PROCESS owners,
  zero PACKED, one arena and a `02000`-word
  adjacent APR-aligned gap.
- [x] **v1615 BRK OOM distinction:** a stepped
  `020000` request loses the parent before the
  test veneer prints an error; the post-cleanup
  MM table cannot prove RAW ownership. Production
  `MM_ERR_NOMEM` takes fatal close rather than
  returning AC2 to the requester.
- [ ] **Pre-cleanup AC2 and MM capture:** stop
  the physical guest at the fatal BRK branch,
  then inspect actual MM extents, PACKED owner
  and rollback before attempting full restoration
  and net/peak RAM acceptance. Default OFF.

- [x] **DLINK PURE correctness:** both host and
  native source predicates now recognize AC0..17
  writes and compare direct EA against the
  absolute linked TEXT range. Host C89 and
  native PDP-6 tests PASS; unsafe mutant rejected.
- [x] **Valid physical PURE DXR pressure setup:**
  `dlink --pure` plus `dxrcheck purity=PURE`
  required before boot. The earlier UNKNOWN DXR
  was not eligible for RAW reclamation. Physical
  48K no-AUX guest reached reclaim and v1381
  DROP/rebase entries.
- [ ] **Exact physical MM acceptance:** the BRK
  request still failed. Determine extent layout,
  typed allocator refusal and whether a RAW
  suffix was actually published before changing
  kernel logic. Slot-0 restore, PIPE/WAIT,
  immutable-source lifetime and net/peak RAM
  remain open. PURE remains OFF by default.

DAIMOS details:
`design/kcore-p1e-linker-pure-address-v1614.md`.

- [x] **P1-E no-AUX KINIT v1613 fix:** absent
  AUXST1 discovery (-1) now selects experimental
  PURE when enabled, as does a validated zero
  BACKSTORE. Existing DISK MRES remains selected
  when validated BACKSTORE is nonzero; default
  PURE-disabled boot behavior is unchanged.
  Source-linked C89 positive/negative tests PASS.
- [x] **Real 64K PDP-6 TSFS boot with no AUX media:**
  DTC root, no DSK/DRM attachments, KINIT
  `PURE LOADED` and user `INIT V1` observed,
  sustained until test shutdown. Fixed GCC KCORE
  unchanged; PURE MRES still 1758 words.
- [ ] **Physical RAW reclaim/wakeup is not accepted:**
  two exploratory oversized-child pressure
  workloads failed before successful RAW reclaim.
  Next try an incremental BRK pressure request
  after establishing sleeping owners. Inactive
  PACKED MM suffixes are excluded from current
  `MEMINFO.process_words`, so use exact physical
  MM accounting for net savings. Release remains
  blocked on PI6/WAIT/EXEC/EXIT, source lifetime,
  failure recovery, and peak/net RAM proof.

DAIMOS details:
`design/kcore-p1e-noaux-boot-v1613.md`.

- [x] **P1-E PURE MRES v1612 smaller, still experimental:**
  remove duplicate PACKED owner scan in PDP-6 service,
  and fold a relocation-safe zero-cost 49-word region
  between DROP/FULL. KCC installed module shrinks
  **2275 -> 2196 words**; GCC lowmem
  **1864 -> 1758 words**, without changing fixed KCORE
  (`024534` octal), permanent data, per-process RAM
  or disk swap. Native v1612 duplicate/pinned MM and
  unsafe READY_ONCE tests PASS.
- [x] **Rejected measured shared-C experiment:** one
  common preparation/extent helper produced a 2356-word
  module, larger than v1611; duplicate policy is a valid
  abstraction but KCC call overhead costs more RAM.
- [ ] **P1-E physical acceptance remains OPEN:**
  do not change `DAIMOS_PURE_MRES_ENABLED=0`
  default. Real no-AUXST1 TSFS boot, PI6 scheduler
  and RUN/EXEC/EXIT/WAIT/BRK, executable source
  lifetime, rollback, peak and net RAM measurements
  are required before acceptance.
  See DAIMOS
  `design/kcore-p1e-pure-mres-minimize-v1612.md`.

- [x] **v1611 experimental source-linked PDP-6 PURE service:**
  dedicated no-AUXST1 MRES with real v1381 zero-copy
  SLEEP/STOP reclaim, v1384 in-place prefix and v1391
  bounded FULL RUN/slot-0 restore; original DISK MRES
  retained on AUXST1. Positive/negative native KCC
  service tests PASS; opt-in KINIT/MRES builds and links.
- [x] **Production default is OFF:** explicit
  `DAIMOS_PURE_MRES_ENABLED=1` required to include
  PURE in KINIT. Default KCORE remains byte-identical
  to v1610, low end `024534` octal.
- [x] **KINIT Make dependency ordering:** stack watermark
  default now precedes the versioned target expansion;
  historical TSFS MINIT v490 GCC test PASS.
- [ ] **P1-E real acceptance remains OPEN:** module cost
  2275 KCC or 1864 GCC words, aggregate 16K placement
  gate fails with PURE included, net RAM benefit unproved.
  Validate real no-AUX physical PDP-6 boot, immutable
  source/unmount/PI6, EXEC/WAIT/EXIT/BRK, MM rollback,
  actual installed MRES memory and transient RAM peaks
  before enabling PURE by default.

Detailed evidence in DAIMOS
`design/kcore-p1e-swapless-service-v1611.md`.

- [x] **Removable SWAP MRES v1610, mode-aware slot-0 service:**
  DISK keeps its original restore, pressure and fatal-error
  path. PACKED defers without disk I/O, TERM or READY_ONCE;
  its original raw suffix/backing survive untouched.
  Invalid mode 3 refuses. Native real-service positive
  tests and unsafe PACKED-as-DISK mutant PASS.
  No KCORE DXR/map growth (`024534` octal).
- [x] **Staged v1610 slot-0 RUN restore eligibility:**
  shared DROP/TERMINAL/RESTORE process preflight. RUN
  accepted only in RESTORE while slot 0 holds target
  TRANSITION. Actual v1384 prefix and v1391 bounded FULL
  source-linked native wake tests PASS. RUN DROP refused.
  Existing disk v1316/v1317 and v1608/v1609 regressions
  PASS. See DAIMOS
  `design/kcore-p1e-slot0-mode-v1610.md`.
- [ ] **P1-E live swapless operations:** real immutable
  source lifetime, no-BACKSTORE PURE module, actual
  inactive DROP, prefix/FULL wake, and hardware PI6,
  EXIT/EXEC/WAIT/MM pressure tests still OPEN.

- [x] **P1-E terminal PACKED owner v1609, production linked:**
  correct mode-2 VM destroy and PDP-6 exact MM suffix
  release; no mistaken disk block free, no live VFS mount
  dependency during EXIT; failed owner release is retryable.
  Native PDP-6 real VM/detach, wrong owner/pin/size/backing,
  double destroy and source-identity mutant PASS. KCORE
  +104 code words, zero additional permanent DATA/BSS.
- [x] **Staged fast prefix source check v1609:** the in-place
  v1384 path now validates the same immutable executable mount
  as v1381 DROP and v1391 FULL. Native positive/negative
  prefix and bounded restore regressions PASS.
- [ ] **P1-E source and scheduler gates remain OPEN:**
  a real no-BACKSTORE boot, immutable source lifetime,
  zero-copy inactive DROP, complete runnable restore,
  slot-0/PI6 and EXEC/WAIT/EXIT/BRK/fragmented-MM
  physical guest acceptance. Do not enable compression
  until RAW integration is accepted.

- [x] **P1-E1 source gate v1608 (staged):** no-disk-backing
  and valid executable mount/provider required before RAW
  DROP/bounded FULL restore; source-loss terminal discard
  remains possible. KCC native DROP/prefix/FULL/rollback
  tests and bypass-mutation rejection PASS. Full PDP-6
  KCORE DXR/map match v1606 byte-for-byte.
- [ ] **P1-E1 production lifetime gate / P1-E2 onward:**
  source immutability during actual swapless boot, scheduler
  PI6/slot-0 integration and physical owner/restore tests
  remain OPEN. No codec or runtime P1-E integration yet.

The authoritative remaining-work inventory is
`STATUS/KCORE-REMAINING-WORK-V1607.MD` and DAIMOS
`design/kcore-remaining-work-v1607.md`.

- [x] P0-A critical ROOT/BACKSTORE; P1-C v1603 combined
  storage and P1-D v1601 FSINFO: ACCEPTED, not active work.
- [x] P0-D/E/F OOM: accepted only for the defined v1298
  release profile and v1348/v1351 owner/no-partner evidence.
- [ ] P1-E RAW swapless PURE: production source ownership,
  no-backing gate, quiesced zero-copy rebase, fast/full
  APR restore, actual PI6/scheduler/EXEC/EXIT/WAIT and
  failure-atomic real native/physical PDP-6 tests.
- [ ] Optional compression yes/no after RAW P1-E,
  based solely on net **whole-system** peak RAM benefit.
- [ ] Final matched-source KCORE acceptance matrix and
  footprint report. Strict 32K/optimization and CPU ports
  remain explicitly unscheduled/HOLD.

Older unchecked historical sections below do not reopen v1603,
v1601, v1575's *declared* policy or v1298 acceptance.

- [x] **v1606 PDP-6-first modularity rule:** existing v1605
  source selection remains, with zero runtime CPU abstraction.
  VM-state RH/active check, physical user-pointer interpretation,
  contiguous MM descriptors, machine PI/traps and EXEC/XIP
  destinations are explicitly machine-owned. Do not plan or
  implement alternate CPU targets before PDP-6 completion.
  `KERNEL/MULTI-CPU-PORTABILITY-V1606.MD`.
- [ ] **P1-E PDP-6 RAW swapless PURE production:** complete
  inactive reclaim/restoration, immutable backing owner, scheduler
  and PI exclusion, EXEC/EXIT/WAIT, failure rollback and real
  PDP-6 hardware-pressure tests. Compression remains conditional
  on measured RAM savings. P0-B and 32K/optimization HOLD persist.

- [x] **v1605 multi-CPU source-map, PDP-6 binary identity:**
  the current PDP-6 kernel/boot source selection is centralized
  in `arch/pdp6_source_map_v1605.mk`, with KCC full `ROOT=tape`
  `kcore.dxr` and map exactly matching the unmodified image.
  Source-only changes; no KA10/KI10 implementation, KCORE growth
  or runtime dispatch. Architecture and source classification:
  `KERNEL/MULTI-CPU-PORTABILITY-V1605.MD`.
- [ ] **Source separation only during actual PDP-6 work:** the
  mixed context/scheduler `proc_pdp6.s` and transfer/policy
  `proc_swap_pdp6.s` may be split after source-sensitive testkit
  coverage is migrated, if a real PDP-6 feature requires it.
  Otherwise retain them as selected machine source units and
  make no speculative page/segment interfaces.

- [x] **P1-E cross-machine architecture REVIEW v1604 (not a
  production feature acceptance):** verified original PDP-6
  single-base APR/VM and original context restore, distinguished
  the KA10 hardware's optional independently protected high
  segment. Recommend a common guarded inactive PURE RAW-suffix
  owner transaction and separate processor-specific MM/APR
  restore, beginning with a KA10 verified low-segment-only
  backend rather than assuming KT10A or shared read-only text.
  Current-source native v1381/v1384/v1395 isolated prototypes
  initially failed to link due to the moved `vfs_namespace_root`
  state owner; testkit now links original `device_state.s`, with
  all three and v540 original MM/APR handoff PASS. No dummy
  namespace, no production source alteration, no full-guest
  swapless/KA10 claim. **P1-E implementation HOLD pending explicit
  design decision and target validation.** P0-B reel retention
  OPEN, strict 32K/optimization HOLD. Full review:
  `STATUS/KCORE-P1E-PDP6-KA10-ARCHITECTURE-V1604.MD`.

- [x] **P1-C FINAL v1603 full physical storage/swap/BCACHE CLOSED:**
  two independent actual 128K KCC PDP-6 boots of DR0..DR3 D6FS
  mapped VOLUME root with real LOGSTORE/MEMFS persisted snapshot.
  Each original guest has independently PI6-awakened DISK owners
  **6/7** (one each), five mapped swap READ attempts/**1120** real
  Type-236 DR_DONE PI2, one checked original MM/bitmap/BCACHE/
  process owner rollback. The original physical LOGSTORE torn
  uncommitted WRITE, selected completion error and genuine same-
  slot committed retry leave a populated D6FS 128-word clean cache
  exact (0/0 differences; authentic HIT/HIT). Boot1 snapshot written,
  boot2 restored, 283 other nonzero backing sectors; **44** false
  cache/PI6/owner/LOGSTORE/raw physical witnesses rejected.
  Fresh current-source v1592 and v1590 loaded two-boot physical
  tests PASS. Native original v1602 D6FS clean WRITE-through
  failed/successful DSK270/DRM236 controller cases 4/4, five unsafe
  source mutations rejected, complete independent 128-word clean
  private cache preserved. Original v1544 DSK/DRM owner-adapter
  24 cases/six mutants and v1490/v1538 source-linked tests PASS;
  earlier v1599/v1600 fatal mapped VOLUME/direct DEVICE BACKSTORE
  single READ/PI2 reason-2 HALT `004011`, and v1596 physical
  DTFS DTC6 outage with healthy MEMFS writes, remain accepted.
  There is NO dirty/writeback cache daemon; no postfatal recovery
  or two simultaneous DMA swap READs is claimed. Historical DTFS
  v1518 test has an outdated pre-v1561 source mutation anchor;
  not claimed PASS. **P1-C CLOSED**, no production KCORE/RAM/ABI/
  media change. Distinct P0-B physical reel-retention OPEN,
  P1-E KA10 and strict 32K size/optimization HOLD. Details:
  `STATUS/KCORE-FEATURE-STATE-V1603.MD`.

- [x] **P1-D FINAL v1601 dynamic FSINFO/controller matrix CLOSED:**
  full original 128K PDP-6 four-member DRM236 D6FSR2 root with
  DSK270 physically absent, KINIT FSINFO **2** slots instead of
  six, original linked MM slab and root/MEMFS identity confirmed,
  four unique untouched DRM member sectors, full XIP lifecycle.
  **10** invalid physical/FSINFO/MM/controller witnesses rejected.
  Fresh current-source complementary DSK270 root / Type-167 DP/DRM
  absent v1597 rerun also PASS, four corrupted cases rejected.
  Previously accepted genuine TSFS roots capacity 7/8 and DSK/DRM
  roots capacity 6; four native high-slot DTFS mounts, physical
  missing provider vs empty FSINFO return, different-reel refusal,
  original-reel recovery and unaffected provider work all PASS.
  Source-linked v1492 exact FSINFO/MOUNTS/DF and v22 full/disk
  root inventory, including missing/asymmetric READ/WRITE pairs,
  PASS. The currently modeled installed controller classes support
  up to 9 slots with a one-member root; 26 is representational,
  not a physically installed array. **P1-C** dirty/write-through,
  cache/owner timing and repeat-loaded I/O remain OPEN; **P1-E**
  and strict 32K/optimization HOLD. No new production KCORE/RAM/
  ABI/MRES/media change. See `STATUS/KCORE-FEATURE-STATE-V1601.MD`.

- [x] **P1-C v1600 independently attached direct-DEVICE DRM swap
  failure with DSK270 root:** actual 128K PDP-6 full KCC guest
  with four distinct physical DSK270 D6FSR2 root members plus
  separate DR0 AUXSTORE (original physical direct BACKSTORE
  `05770+010000`, LOGSTORE `05370+0400` octal). Original
  `device_region_map=400000005770` and `volume_data_blocks=0`
  explicitly rule out a mapped VOLUME. Original PI6 service
  wakes two separate swapped SLEEP/DISK children, slots 6 and 7,
  once; scheduler enters one real DR0 direct BACKSTORE swap READ
  for **slot 6**, actual physical `006421`, logical `000431`.
  Authentic DR_DONE PI2 via disposable test-only selected error
  enters unchanged driver/critical BACKSTORE branch and required
  fatal reason-2 HALT octal `004011`. No RESIDENT publication,
  original positive swap and BACKSTORE ownership retained, no
  false persistent snapshot commit, physical AUXSTORE descriptor
  and four real DSK-root layouts independently checked. **27**
  deliberately unsafe witnesses rejected; v1599 23 and v1586 15
  regression mutants still PASS. Only one swap DMA before
  mandatory fatal halt; no postfault service claimed. No
  permanent source/RAM/ABI/media change. Full details in
  `STATUS/KCORE-FEATURE-STATE-V1600.MD`.

- [x] **P1-C v1599 two independently PI6-woken DISK owners at
  fatal physical BACKSTORE READ:** full 128K PDP-6 with original
  DR0..DR3 four-member D6FS root VOLUME, mapped BACKSTORE and
  real LOGSTORE/MEMFS shutdown snapshot. Original timer service
  wakes *both* distinct original SLEEP/DISK owners, slots **6**
  and **7**, once. Original scheduler selects slot 6 for one
  physically mapped Type-236 swap READ; its genuine DR_DONE PI2
  follows a disposable one-shot injected original DRM error
  and unchanged reason-2 BACKSTORE HALT octal **`004011`**.
  No false RESIDENT publication, original positive BACKSTORE/
  process allocation and physical backing content remain,
  uncommitted snapshot and four intact D6FSR2 headers.
  **23** deliberately corrupt owner/PI6/PI2/media/ownership
  controls rejected; original v1586 15 controls still PASS.
  Only ONE actual swap DMA preceded the fatal HALT, no
  post-fault recovery or dual DMA is claimed. No permanent
  KCORE code/RAM/ABI change. `STATUS/KCORE-FEATURE-STATE-V1599.MD`.

- [x] **P1-D v1595 physical high mounted-volume error:** actual
  two-member Type-551 TSFS root with eight FSINFO slots, four
  real native DTC3..DTC6 DTFS secondary volumes, public IDs
  2..5. Live DTC3 physical removal returns FSINFO error, not
  an empty published slot, while all high identities survive.
  Different stamped reel refused; original reel recovers; all
  mounts retire. Independent physical media/checksum oracle
  rejects five unsafe cases. No production RAM/ABI changes.
- [x] **P1-C v1596 real high-slot DTFS loss plus writable MEMFS:**
  physical DTC6 highest published volume ID5 disappears and
  returns while a separate writable MEMFS at ID6 writes and
  rereads exact file data. Original FSINFO refuses missing DTC6;
  all other providers retain identity and progress. Full 256K
  original DAIMOS KCORE guest on corrected Type-551 simulator
  PASS, five authentic physical DTVOL1 reels and eight rejected
  false raw/CTY witnesses. Original simulator v1524 repeatedly
  failed DTC6 return due stale reel `CMD/DSTATE/DELAY` after
  detach/attach, despite valid same media. Root-cause corrected
  in `sims` commit `fe97912` (v1598), not in KCORE; installed
  versionized `pdp6-v1598` preserves former v1524 binary.
- [x] **P1-D v1597 physical missing Type-167/236 hardware:**
  genuine DSK270 D6FSR2 4-member 128K PDP-6 boot, actual
  Type-167 `DP` I/O processor and drum units disabled before
  MINIT. Original FSINFO capacity 2 (D6FS ID1 + MEMFS ID2),
  not 6 with controller present; zero phantom DRM slots. XIP
  lifecycle, four raw physical D6FSR2 root descriptors and four
  mutation controls PASS. Initial test disabled only DR but
  missed Type-167 DP probe; corrected test, no kernel patch.
- [x] **P1-C final remaining cases CLOSED by v1602/v1603:**
  v1599/v1600 direct DEVICE and mapped VOLUME fatal swap,
  v1596 separate physical DTFS/MEMFS provider isolation, native
  v1602 authoritative WRITE-through/BCACHE failure and physical
  v1603 two PI6 owners with actual recoverable LOGSTORE torn/retry,
  exact MM/bitmap/cache rollback and loaded two-boot repeatability
  all accepted. No recovery after a critical owned-media HALT.
- [x] **P1-D final source-matched matrix:** completed by v1601
  reverse no-DSK physical root, current-source no-DRM physical
  rerun and native service-pair cases, together with accepted
  7/8/6/2 root/controller profiles and high mounted Type-551
  failure/return. No physical 26-device claim. P1-E KA10 HOLD.

## Historical KCORE feature increment: v1594 (2026-11-15)

- [x] **P1-D v1594 real four-member DSK270-root physical FSINFO:**
  original 128K KCC PDP-6 boots from four separate DSK0..DSK3
  D6FSR2 physical disk members. KINIT and original FSINFO publish
  capacity six: root D6FS provider 6, MEMFS `/TEMP` provider 4,
  four spare DRM236 positions, no phantom mount in holes 2..5,
  indices 6/7 invalid. The real guest also completes the ordinary
  XIP EXEC/RUN/WAIT regression. A competing-simulator port
  collision in the first attempted run was rejected; corrected
  per-run DCS/GE ports pass with NO simulator errors. Independent
  verifier checks all four distinct original physical D6FSR2
  member identities/geometries (45,056 sectors per disk) and
  rejects 3 false physical/FSINFO witnesses. Full corrected
  guest PASS, no installed code/RAM/ABI/media changes.
- [x] **P1-C v1593 specific independent RESIDENT owner proven:**
  two full 128K PDP-6 physical cold boots on DR0..DR3 Type-236
  mapped root VOLUME, original PI6 timer waking distinct slots
  6 and 7 during persistent MEMFS/LOGSTORE snapshot. Source-linked
  observer after original `proc_storage_publish_v30` success
  shows timer-woken **slot 7** publishes RESIDENT on both boots,
  aligned VM `0220000`, stable u-area, transition clear and
  backing cleared. Slot 6 also published successfully once
  during another transaction before/around its later checked
  release refusal; the separately refused operation rolls back
  original DISK/MM/bitmap/BCACHE owner and sends TERM. Five
  physical mapped swap READs, 1,120 actual Type-236 PI2, four
  RESIDENT publications per boot; boot 2 restores MEMFS/XIP/
  LOGSTORE, v2 header DR3 `017777`. Thirty-nine false witnesses
  rejected. Same real DRM-root guest verifies original FSINFO
  capacity six. Production kernel, RAM and ABI unchanged.
- [ ] **P1-C remaining:** mixed later-stage DEVICE/VOLUME/DTFS
  physical errors, dirty/write-through rollback and repeatable
  loaded-simulator timing. Actual owned-media I/O failure remains
  critical fail-stop, never silently recoverable.
- [ ] **P1-D remaining:** genuinely failed or absent providers,
  high-slot physical provider error/retirement, and fail-closed
  empty/unpublished distinctions under alternate device inventory.
  Physical DTC TSFS seven/eight slots and D6FS DRM/DSK six slots
  are accepted; the 26-slot maximum is representational only.

## Historical KCORE feature increment: v1592 (2026-11-15)

- [x] **P1-C v1592: two distinct real sleeping DISK owners awake in
  the same physical snapshot:** two complete 128K KCC PDP-6 cold
  boots on four-member Type-236 DR0..DR3 root VOLUME, mapped
  BACKSTORE, MEMFS v2/XIP and LOGSTORE. Test-only timer observer
  retimes original slots 6 and 7 to one original PI6 tick during
  snapshot; both original timer-service wakeups observed (1/1).
  Each boot performs five real independent slot-0 swap READs,
  1120 matched Type-236 PI2 and four successful RESIDENT publishes;
  selected slot 6 receives controlled post-DMA checked BACKSTORE
  release refusal with original MM/bitmap/BCACHE/TERM rollback.
  Both normal HALTs, second boot recovers MEMFS/XIP/LOGSTORE,
  physical DR3 v2 snapshot header `017777`, 3,648-word pool and
  277 other nonzero backing sectors. Twenty-nine deliberately
  false linked/physical owner/PI6/MM witnesses rejected. Production
  code, resident RAM, ABI and on-media format unchanged. See
  `STATUS/KCORE-FEATURE-STATE-V1592.MD`.
- [ ] **P1-C remaining:** the four successful publications are
  aggregate; prove the *specific* other timer-woken slot 7
  reached RESIDENT, then mixed DEVICE/VOLUME later failure,
  dirty/write-through/DTFS and repeated loaded-simulator timing.
  Genuine owned-BACKSTORE media faults remain fail-stop.

## Historical KCORE feature increment: v1591 (2026-11-15)

- [x] **P1-D v1591 genuine above-seven-slot physical boot:** an
  unmodified KCC PDP-6 boots a separate TSFS root on *two* physical
  DTC0/DTC1 Type-551 tapes, with DTC2 empty. Original TSFS root
  discovery and KINIT mount slab publish FSINFO capacity **8**.
  Original PID1 mounts four separately 72-bit-stamped DTFS tapes
  DTC3..DTC6 (IDs 2..5); checks empty slots 5..7, invalid 8/9,
  exact provider/path, stable high ID across middle unmount and
  clean retirement. Full physical guest PASS, corrected KCC test
  probe recompiles without warnings; no production source/RAM/
  ABI/media changes. See `STATUS/KCORE-FEATURE-STATE-V1591.MD`.
- [ ] **P1-D remaining:** actual alternate D6FS DSK/DRM-root
  controller/service inventories and provider-failure enumeration.
  The native root-mask inventory tests and 7/8-slot physical
  TSFS profiles are accepted. Twenty-six is not a real physical
  device-count claim.
- [ ] **P1-E architecture hold for KA10:** revisit the formerly
  planned uncompressed swapless PURE/MM rebase/APR integration
  before writing production code. The v1381/v1391 pieces remain
  isolated PDP-6-specific experiments, not a cross-target contract.
  Decide common versus machine-specific VM/text restore and
  protection semantics with later KA10 compatibility in mind.
  Preserve ordinary disk/drum swapping and fail-closed ownership;
  defer RAM compression and all new P1-E implementation until
  this architecture decision is made.

## Historical KCORE feature increment: v1590 (2026-11-15)

- [x] P1-C v1590 live D6FS clean cache preserved through original
  physically uncommitted LOGSTORE write/refusal and same-slot retry:
  two genuine 128K KCC PDP-6 cold boots on four-member Type-236
  DR0..DR3 VOLUME. Original VFS/D6FS READ populates private clean
  BCACHE slab `0377577`, key `100100001010`, 28 nonzero of all 128
  data words. The physical LOGSTORE trailer-zero record write is
  real; the error reported afterward is test-induced. Both after
  refusal and after committed retry, cache owner/key and all 128
  data words are unchanged, original `bcache_fetch` HIT/HIT, zero
  fetched-word differences. Subsequent original PI6/slot-0 swap
  completes four mapped physical READs and 896 matching Type-236
  PI2 per boot; original post-DMA rollback retains process DISK
  backing `01650` octal, all 29 bitmap words, and three successful
  resident publishes. Second cold boot restores MEMFS/XIP/LOGSTORE;
  snapshot header DR3 `017777`. Forty-seven corrupted original/
  physical/cache witnesses rejected. The separate native v1490
  BCACHE/DTC regression was repaired and passed using the genuine
  v1561 DTFS write-intent gate (no bypass).
- [ ] **P1-C remains OPEN:** simultaneous independent owner changes,
  later-stage fault timings and loaded repeated-boot stability,
  including isolated v1585 timeout. The successful cache-retention
  case is separate from v1589 legitimate MM cache reclamation.
  Actual owned-BACKSTORE controller errors remain fatal. P1-D/E,
  VERSION 0.5 and strict 32K/optimization HOLD remain unchanged.

## Historical KCORE feature increment: v1589 (2026-11-15)

- [x] Genuine physical D6FS populated BCACHE reclaimed during
  independent PI6 swap-in: two complete 128K PDP-6 cold boots on
  four-member DR0..DR3 Type-236 root VOLUME. Ordinary VFS OPEN/READ
  populates private cache slab `0307577`, D6FS source/block key
  `100100001010`, 28 nonzero of 128 original data words. At exact
  original `bcache_reclaim`, one independent swap has started;
  all 128 words match, original MM frees slab, old tag later MISSES
  and no stale cache data survives. Separate v1588 post-DMA checked
  release refusal correctly preserves original DISK owner `01650`
  octal and all 29 bitmap words. Four mapped physical swap READs,
  896 matched Type-236 PI2 and three RESIDENT publishes EACH boot;
  both normal HALTs `022004`, second boot recovers MEMFS/XIP and
  LOGSTORE; DR3 commit header `017777`, 282 other nonzero backing
  sectors. Forty-eight bad linked/physical/cache witnesses rejected.
  No production code/RAM/ABI change.
- [ ] **P1-C OPEN:** no-reclaim populated-cache retention under an
  unrelated provider rollback, mixed DEVICE/VOLUME and multi-owner
  transitions, repeated-load stability (v1585 intermittent timeout).
  Separate legacy v1490 BCACHE/DTC native fixture cannot link modern
  `dtfs_intent_mount_gate_v1561`; repair its source-authentic test
  harness without stubbing or bypassing a real write-intent gate.
  P1-D/E and VERSION 0.5/32K optimization HOLD unchanged.

## Historical KCORE feature increment: v1588 (2026-11-15)

- [x] Physical post-DMA checked BACKSTORE release refusal with
  original recoverable MM rollback: two full 128K KCC PDP-6 cold
  boots on four-member Type-236 mapped DR0..DR3 root. Original PI6
  wakes independent slot-0 swap-in; four actual physical BACKSTORE
  swap READs and 896 genuine matching Type-236 PI2 completions per
  boot. One test-only release refusal after physical READ/MM unpin
  invokes original `proc_swap_in_free_fail`; three OTHER restores
  publish RESIDENT. MM extent `024->023`, allocated base `0220000`
  absent, process-owned blocks `01650` octal, backing `01734`/`01713`
  octal, all 29 bitmap words, DISK record/mode and BCACHE metadata
  preserved across cleanup. Both native HALTs `022004`; second
  cold boot restores MEMFS/XIP and previous LOGSTORE; valid physical
  header DR3 `017777`, 275 other nonzero backing sectors. Thirty-four
  corrupted original/physical witnesses rejected; v1586/v1587
  retained physical regressions pass. No production code/RAM change.
- [ ] **P1-C remains OPEN:** real populated BCACHE identity/data
  handling under mixed DEVICE/VOLUME provider failures, simultaneous
  owner transition failures, and v1585 loaded-simulator timing
  stability. Synthetic provider release refusal does NOT make
  owned-media errors recoverable; the original fatal policy holds.
  P1-D/E remain OPEN; VERSION 0.5 and strict 32K/optimization HOLD.

## Historical KCORE feature increment: v1587 (2026-11-15)

- [x] Recoverable pre-I/O MM pin refusal during independent PI6/
  slot-0 swap-in while full physical 128K PDP-6 commits mapped
  MEMFS snapshot. Two real DR0..DR3 D6FS root VOLUME cold boots;
  original MM allocates aligned process base `0220000`; synthetic
  MM pin refusal then original MM free/transition unlock/TERM return
  nonfatally. Both real boots original MM extent count `024->023`,
  full 29-word backing bitmap exactly unchanged, actual owner
  counts and disk record/mode maintained, BCACHE metadata preserved,
  and three additional swap-ins publish successfully with 672
  matching original Type-236 PI2. Physical DR3 v2 snapshot header
  `017777` committed and restored on boot 2 with prior LOGSTORE,
  275 additional nonzero backing sectors. Thirty-three false
  witness cases rejected; v1585/v1586 regressions still pass.
- [ ] **P1-C remains OPEN:** nonfatal post-I/O owner/MM/BCACHE
  restoration after an actual transaction mutation, multiple
  simultaneous transitions and v1585 loaded-timing stability.
  Do not reinterpret fatal owned-BACKSTORE faults as retryable.
  P1-D/E, VERSION 0.5, 32K/optimization HOLD unchanged. No
  production code or RAM growth.

## Historical KCORE feature increment: v1586 (2026-11-15)

- [x] Full 128K PDP-6 independent swap-READ PI2 fail-stop during
  snapshot DMA on the real DR0..DR3 four-member Type-236 root VOLUME.
  Original PI6 wakes a genuinely disk-backed timed sleeping child;
  original slot-0 begins one physical swap READ; the selected actual
  DR_DONE is routed through original DRM error handling and critical
  BACKSTORE reason-2 HALT `004011`. Original driver errors 1,
  successful resident publications 0, process swap blocks `01650`
  octal remain owned, snapshot uncommitted, exact physical selected
  DR1 `017506`/logical `000431`, 268 other nonzero tail sectors and
  unchanged four D6FSR2 layouts. Fifteen deliberately corrupted
  original/physical witnesses rejected; v1583/v1585 regressions
  reverified. Production kernel source, RAM and ABI unchanged.
- [ ] P1-C still OPEN for *recoverable* owner/MM/BCACHE rollback,
  later-sector error timings and multiple simultaneous owner changes,
  and the previously observed rare v1585 timeout under load.
  P1-D/E and VERSION 0.5/32K optimization HOLD unchanged.

## Historical KCORE feature increment: v1585 (2026-11-15)

- [x] Genuine independently scheduled swap-in DURING physical MEMFS
  snapshot acceptance: two full 128K PDP-6 cold boots on the real
  four-member Type-236 mapped DR0..DR3 root VOLUME. Test-only
  deadline alignment wakes one originally sleeping disk-backed
  child via the genuine 60 Hz PI6 timer; the original slot-0
  scheduler completes two process swap-ins/publications each boot,
  with `0700` octal matching original swap-READ DMA DR_DONE and
  `01311` total snapshot-window PI2 per boot. Both normal HALTs
  `022004`, original process-backed blocks `0750`, zero device
  errors. Boot 2 restores MEMFS/XIP and prior LOGSTORE, physical
  v2 header DR3 `017777`, 275 nonzero other sectors, four intact
  members. Seventeen corrupt evidence cases rejected; repaired the
  inherited v1584 negative-test oracle without weakening its
  earlier zero-overlap baseline. No production changes.
- [ ] P1-C still OPEN for exact recoverable MM/BCACHE/owner rollback
  and failure *during* independent swap/snapshot overlap. P1-D and
  P1-E remain OPEN; VERSION 0.5 and the 32K/optimization HOLD.

- [ ] Repeat v1585 physical overlap under loaded simulator timing:
  one additional 180-second probe timed out after partial physical
  snapshot writes, without captured CPU state, while a later full
  two-boot rerun passed all strict evidence and negative checks.
  A bounded 300-second v1585-only timeout now captures CPU/PC on
  failure. Slow host execution and rare scheduler stalls remain to
  be distinguished; do not treat this as proven media corruption or
  a confirmed kernel deadlock.

## Historical KCORE feature increment: v1584 (2026-11-15)

- [x] Full 128K PDP-6 two cold-boot physical snapshot v2 commit and
  persistence recovery on original four-member DR0..DR3 root VOLUME.
  Positive real process swap ownership on both HALTs, `051` octal
  Type-236 snapshot DR_DONE PI2 events each, valid mapped DR3
  sector `017777` header, 3,648-word MEMFS pool, 271 nonzero other
  backing sectors, byte-identical physical member descriptors,
  recovered MEMFS XIP/data and prior LOGSTORE record. Ten corrupted
  evidence variants rejected; no production code or RAM change.
- [ ] **Independent swap-in during snapshot DMA remains UNPROVEN:**
  original slot-0 swap-in start, final publication and matching
  physical swap-READ PI2 counts are all zero in both otherwise
  healthy boots. Optional `--require-overlap` oracle correctly
  rejects this evidence. P1-C remains OPEN for demonstrated
  independently scheduled swap-in and nonfatal MM/BCACHE/owner
  rollback. P1-D/E stay OPEN; VERSION 0.5 and 32K/optimization HOLD.

## Historical KCORE feature increment: v1583 (2026-11-15)

- [x] Real 128K PDP-6 four-member Type-236 D6FS root VOLUME with
  persistent MEMFS/XIP, LOGSTORE and four 24K-word sleeping swapped
  children. Real first invalid snapshot header WRITE at original
  logical last BACKSTORE block, exact selected Type-236 DR_DONE PI2
  error publication and reason-2 HALT `004011`, one original driver
  error and `01374` octal actual owned blocks. Physical mapped
  `075774` data/`02000` tail and DR3 sector `017777` group
  `777770` independently checked across all four raw 36-bit drums;
  248 other nonzero tail sectors, invalid physical commit header and
  unchanged on-media descriptors. Twelve corrupted physical evidence
  cases rejected. No production RAM/source changes.
- [ ] P1-C remains OPEN for actual independent swap-in *during*
  snapshot DMA and recoverable owner/MM/BCACHE fault rollback.
  P1-D/P1-E remain OPEN, VERSION 0.5 and 32K optimization HOLD.

## Current KCORE feature increment: v1582 (2026-11-15)

- [x] Source-linked native PDP-6 real BSET INTERLEAVE translator,
  fixed BACKSTORE gateway, v1515 reason-2 HALT, with three physically
  distinguishable members. 13 successful/error/refusal/range cases
  and 8 rejected executed unsafe source mutants. Every selected
  logical tail sector is verified to the correct member, physical
  sector, DSK/DRM class and buffer stride. Existing swap owner
  retained at fatal HALT; disabled-but-owned fatal and inactive/
  unowned negative returns both proved. No production RAM change.
- [ ] P1-C remains OPEN for **physical** full multi-member VOLUME
  controller failures, real concurrent swap-in/snapshot PI overlap,
  and exact recoverable owner/MM/BCACHE rollback. P1-D/P1-E and
  KCORE completion/freeze remain OPEN. 32K optimization HOLD.

## Current KCORE feature increment: v1581 (2026-11-14)

- [x] Actual full 128K PDP-6 Type-236 direct DEVICE failure during
  persistent MEMFS shutdown with live swap owners. Selected physical
  snapshot-header DR_DONE goes through original PI2 error and original
  fatal reason-2 HALT at linked PC `004011`. Post-HALT KCORE words
  confirm one real descriptor completion, original DRM driver error
  counter 1, `01374` octal swap-owned
  blocks, exact `0157670` group address, cleared target and arm.
  Physical drum has no committed snapshot header, retains 248
  nonzero backing sectors, and keeps all AUXSTORE partitions intact.
  Seven intentionally corrupted physical witnesses rejected; no
  production source/RAM/ABI changes. Injected PI2 is not natural
  magnetic media failure.
- [ ] P1-C OPEN: independently scheduled PI swap-in overlap with
  snapshot DMA, multi-member mapped VOLUME failure/return and exact
  nonfatal owner/BCACHE/MM rollback. P1-D/P1-E and final freeze OPEN.

## Current KCORE feature increment: v1580 (2026-11-14)

- [x] Physical 128K KCC/PDP-6 twice cold-booted on same real
  Type-236 AUXSTORE drum: four sleeping 24K-word process children
  existed at each privileged persistent MEMFS HALT. The disposable
  snapshot MRES refused zero actual KCORE `proc_swap_blocks_used`;
  both genuine physical HALTs passed. Independent raw-drum oracle
  verified the 35-sector v2 snapshot for the real 3,648-word pool,
  248 nonzero non-snapshot backing sectors, intact separate
  LOGSTORE range and byte-identical AUXSTORE descriptor. Boot two
  recovered exact MEMFS data/XIP and boot-one committed LSREC1.
  No production code or RAM change.
- [ ] P1-C remains OPEN for PI-driven simultaneous swap-in DURING
  snapshot DMA, mapped DEVICE/VOLUME failures, and ownership/
  BCACHE/PI rollback. This new physical owner witness cannot close
  those distinct acceptance requirements. P1-D/P1-E/freeze open.

## Current KCORE feature increment: v1579 (2026-11-14)

- [x] Source-linked original PDP-6 MEMFS snapshot and BACKSTORE
  allocator/free execute together. An existing process and the
  38-block persistent snapshot remain correctly owned when two more
  process runs are allocated across invalid-header and metadata I/O
  callbacks. All 512 bitmap bits, 147/39 sector accounting, final
  commit order and three executed unsafe mutants PASS/reject.
  Native physical I/O callbacks are synthetic, not real DMA.
- [ ] P1-C full real-guest swap/snapshot HALT concurrency and mapped
  DEVICE/VOLUME/PI controller failure and owner rollback are still
  open. P1-D alternate real inventory, P1-E uncompressed PURE and
  final cross-feature acceptance remain open.

## CURRENT KCORE FEATURES after v1578 (2026-11-14)

The dated entries below preserve historical acceptance and earlier
open questions. This top section supersedes their P0-A `OPEN`
claims; do not treat historical status lines as separate current
tasks. No KCORE optimization or 32K reduction is authorized.

- [x] P1-C partial v1578: leave one invalid LOGSTORE physical
  LSREC1 with a zero commit trailer *unretried* through MEMFS
  snapshot HALT. Host verifies exact 36-bit physical orphan before
  second cold boot; fresh production KINIT ignores it and reuses
  original physical slot/sequence, while MEMFS data/XIP restores.
  Consecutive physical committed sequences `3,4,5`; four negative
  altered physical images rejected; native LOGSTORE recovery PASS.
  Synthetic completion status after actual invalid Type-236 write,
  NOT a naturally failing controller. **P1-C remains OPEN** for
  concurrent process swap/HALT and real DEVICE/VOLUME/PI failures.
  Zero production code/RAM. `STATUS/KCORE-FEATURE-STATE-V1578.MD`.

- [x] P1-C partial v1577 LOGSTORE torn/failure rollback: original
  producer PDP-6 source and four rejected mutations verify cursor,
  slot and wakeup stability after failed physical block write.
  Two full 128K guest cold boots physically write an uncommitted
  LOGSTORE record on DRM236, simulate negative completion through
  test-only MRES, observe unchanged cursor and invalid physical
  readback, retry the same slot, then recover log and MEMFS persistent
  state across cold boot. This is NOT a natural Type-236 error.
  **P1-C remains OPEN** for unretried orphan recovery, active
  swap/snapshot overlap and real hardware DEVICE/VOLUME/PI errors.
  No production code/RAM. `STATUS/KCORE-FEATURE-STATE-V1577.MD`.

- [x] P1-C v1576 partial physical mixed-storage acceptance:
  one 128K PDP-6 full production guest, two cold boots on the same
  AUXSTORE drum with persistent MEMFS XIP/data and exact physical
  LOGSTORE record write/read/recovery across boot. v1536 KINIT
  snapshot-reserved swap bitmap, seven rejected mutants, and v1316
  source-linked swap-in/MM rollback, three rejected mutants, PASS
  independently. Descriptor geometry byte-identical. **P1-C OPEN**
  for active concurrent physical swap + snapshot commit, LOGSTORE
  partial failure and mixed DEVICE/VOLUME loss/return. Zero new
  production code/RAM. `STATUS/KCORE-FEATURE-STATE-V1576.MD`.

- [x] P0-B CLOSED v1575 for explicit **operator-trust DTVOL2**:
  full physical RW/DIRTY/CLEAN/reboot, torn DATA and sticky owner,
  destructive discard of an unowned poisoned VFS mount while the
  original DIRTY reel remains installed, RW/RO remount refusal,
  fresh CLEAN replacement mount and independent damaged-reel
  cold boot PASS. v1531/v1570 raw replacement verifier replaced by
  serialized cache-only provider drain. VFS owner/MM/PIPE/swap
  rechecks retained. Optional DTFS MRES -89 words; fixed RAM
  unchanged. No hardware interlock or auto-repair is claimed;
  legacy V1 abandoned-reel safety requires operator quarantine.
  See `STATUS/KCORE-FEATURE-STATE-V1575.MD`.

- [x] v1574 Type-136 reverse WRITE count overrun: the production
  driver's reversed final-data PI3 now leaves count zero to dispatch
  both normal write ACKs; reverse READ retains its count reset. Two
  full-physical original/unsafe mutant classes and original native
  stream/reverse regressions PASS. Full KCC image links, optional
  tape MRES +1 TEXT, zero fixed RAM. v1572 operator-trust changes
  remain an independent uncommitted integration and the extra
  DTVOL2 RBN0 read has not been removed from production.

- [x] **P0-A critical ROOT/BACKSTORE fail-stop COMPLETE.**
  Both full-guest orders on shared DRM236 root plus owned BLOCKSET
  VOLUME now PASS: BACKSTORE-first v1547 to HALT `003747`, and
  ROOT-first v1548 to HALT `003743` with octal `0320` swap
  blocks owned at arm and HALT. Native test, actual selected
  PI2 driver error path and 13 evidence mutations PASS;
  root tape/disk/drum and independent mounted secondary
  hardware negative tests remain accepted. An injected PI2
  branch is not the same as natural hardware disappearance.
- (Historical, superseded) **P0-B:** durable rejection/authorized repair or reformat
  of a previously discarded, genuinely partial WRITE_UNCERTAIN
  secondary DTFS volume after later reinsertion or cold boot;
  preserve owner/cwd/PIPE/swap lifetimes and replacement identity.
  v1549 checked DTVOL2 pre-write intent protocol and offline
  prefix-partial / real `mkdtfs` media models PASS. There is
  **no deployed KCORE durability implementation yet**; physical
  native Type-551 and cold-boot acceptance remain open.
  v1550 installed the independent PDP-6 DTVOL2 record parser
  and explicit experimental host formatter option; 13 native
  media cases, seven unsafe source mutants and legacy v1520
  regression PASS. Parser is NOT linked into production DTFS;
  no write-ahead or clean-unmount feature has been enabled.
  v1551 adds a source-linked raw Type-551 RBN0 dirty-intent
  arm/readback with stable volume ID and cache invalidation.
  Fourteen native PDP-6 cases and eight unsafe source mutants
  PASS. The service is still deliberately unlinked pending
  MM scratch, actual mandatory write gate and owner-safe CLEAN
  unmount; cold-boot/partial-media physical tests are open.
  v1552 adds a source-linked **MM-allocated, physically pinned**
  0200-word scratch owner 015 octal for v1551. Eleven strict-MM
  native scenarios, ten source mutants and a second test using
  production MM allocator/pin/free instructions all PASS.
  A failed unpin/free retains the caller's exact MM owner pointer.
  The wrapper is not yet wired into mounted DTFS and P0-B is OPEN.
  v1553 authenticates the VFS DTFS root/mount and stored volume ID
  for its first-write dirty barrier. Eighteen PDP-6 cases and
  13 unsafe source mutations PASS/rejected. It remains unlinked
  until physical already-armed media revalidation, real provider
  write routing and owner-excluding CLEAN/unmount are complete.
  v1554 adds physical DIRTY/identity revalidation for each later
  write, sharing the existing pinned MM scratch lifecycle. Native
  17 raw media tests/10 mutations, 23 gate tests/8 mutations,
  real production MM verify and 4 mutations all PASS.
  Still unlinked; mandatory provider mounting/writing and final
  CLEAN-unmount with cold-boot physical acceptance remain OPEN.
  v1555 integrates **real DTFS MRES DTVOL2/CLEAN mount admission,
  READ-ONLY only**. The central write veneer rejects any internal
  DATA/DIR write to a unit with a published V2 mount, including
  another mount of the same unit. Full KCC optional MRES link
  PASS; 15 mount cases/9 mutants, 12 write cases/7 mutants,
  12 C89 probe checks and v1519/v1520/v1522 regressions PASS.
  Writable V2, exclusive CLEAN/unmount and cold boot remain OPEN.
  v1556 completes actual production V2/CLEAN read-fault recovery:
  strict physical RBN0, saved 72-bit ID, unchanged media/root/
  read-only policy after sleeping I/O; 29 native PDP-6 cases,
  14 unsafe mutants and legacy v1522 acceptance all PASS.
  KCC full DTFS MRES assemble/fold/link PASS. Writable V2 remains
  DISABLED pending mandatory DIRTY write gate, retained scratch,
  exclusive CLEAN/unmount and physical cold-boot acceptance.
  v1557 adds the actual native production exclusive V2 physical-
  unit mount check: V2 conflicts with ANY existing DTFS on its
  DTC unit; V1 conflicts with an existing V2 on the same unit;
  V1/V1 behavior remains. Native 26 cases/8 mutants and 24
  real-image mount cases/11 mutants PASS; old v1519/v1520 and
  full KCC MRES link PASS. No writable V2 is enabled.
  v1558 fixes real physical short/false-successful admission:
  the shared RBN0144 workspace is zeroed before each native
  directory read; RBN0 gets 128 **nonzero all-one sentinels**
  so no-byte success cannot masquerade as valid zero legacy.
  Unlinked v1551 dirty arm also clears its MM scratch before
  initial read and final readback. Native 17 arm cases/10 mutants,
  28 real-media mount cases/13 mutants and 20 replacement cases/
  six mutants PASS, plus old v1519/v1520/v1555/v1556 and full
  KCC DTFS MRES link PASS. Writable V2 and CLEAN/unmount still OPEN.
  v1560 establishes the retained DTFS MM scratch `basep` lifetime
  in the existing per-slot VFS provider-private auxiliary word,
  shared with D6FS's reader slot on disjoint provider mounts.
  Native VFS refuses a stale owner before DTFS publication and
  after final owner recheck before unmount; poisoned discard
  refuses both before and after sleeping replacement I/O.
  No added fixed RAM or mount slab word; native 16 helper/5 mutants,
  14 VFS publish/unmount/5 mutants and 5 ordering mutants PASS,
  plus v1514/v1523 regressions and full KCC MRES link. Actual
  v1552 write gateway still UNLINKED; writable V2, CLEAN/unmount,
  physical Type-551 replacement and cold boot remain P0-B OPEN.
  v1561 links the real v1551-v1554 first-write physical RBN0
  marker and MM-pinned scratch helpers into optional production
  DTFS MRES. The native central DATA/DIR write veneer now calls
  a unique-unit-to-VFS-mounted V2 bridge, passing the exact v1560
  retained aux slot and preserving raw UNIT/BLOCK/BUFFER across
  sleeps. The existing v1555 unconditional RO veto still precedes
  the bridge; no actual V2 RBN0/DATA/DIR write is enabled.
  PDP-6 35 bridge cases/13 mutants, 12 central veneer cases/9
  mutants, previous v1554/real v1552 MM, v1559 source audit and
  full KCC KCORE/DTFS MRES build PASS. P0-B still needs exclusive
  CLEAN/unmount and physical partial-write/reinsertion/cold boot.
  v1562 adds the tested **transient V2 owner-quiescence prerequisite**:
  use an existing MM-backed per-mount media bit before sleeping SYNC,
  reject new FILE/cwd, stale mount and ordinary provider admissions,
  and release only the original VFS owner's barrier without yielding
  before retirement. Real V2 RO veto remains active; no physical
  CLEAN seal has been implemented. Native 41 helper cases/7 mutants,
  12 complete VFS unmount cases/3 mutants, earlier v1514/v1523/
  v1554/v1555/v1556/v1560/v1561 regressions and KCC image PASS.
  Still needed: PIPE/swap and scheduler admission proof, actual
  owner-safe physical CLEAN/write/readback, media exchange protection,
  partial writes/reinsertion/cold boot. **P0-B remains OPEN.**
  v1563 closes staged PURE executable owner exclusion in RUN/EXEC:
  the production C backing-attach path now checks DTFS quiescence
  before publication; native mount/backing scanners now count SIDL
  with or without its stable u-area, including mount IDs 17..26.
  No extra process/mount/RAM word. 11 native KCC attach and real
  VFS gate cases/two mutants; 26 actual PDP-6 SIDL/resident/swapped
  PURE owner cases/six mutants; host and v1382/v1523/v1562/PIPE
  regressions and full KCC image PASS. +34 fixed KCORE TEXT words.
  Still needed: full-guest scheduled FILE/PIPE/EXEC interleaving,
  physical exclusive CLEAN/readback and removable Type-551 identity
  protection/fault acceptance. **P0-B remains OPEN.**
  v1564: **two actual KCC PDP-6 physical cold boots PASS**.
  The full Type-551 DTC3 DTVOL2/CLEAN image rejects RW, mounts
  RO, denies unmount against separately scheduled PID2 cwd/FILE
  owners, retires, remounts RO and retires again; its whole-image
  SHA-256 is unchanged. A second physical boot with an independently
  valid WRITE_INTENT seal rejects RO and RW, publishes no slot and
  leaves tape bytes unchanged. Old DTVOL1 physical cross-owner test
  also PASS. No production code or RAM change.
  **P0-B OPEN:** real overlapping SYNC/new-owner scheduler race,
  physically checked exclusive CLEAN write/readback, media exchange
  protection and partial RW failure/reinsertion/cold-boot proof.
  v1565: **actual two-process PDP-6 scheduler crossover PASS**.
  A source-matched isolated KCORE copy sleeps with the genuine
  V2 mount quiescence bit acquired; the second real process
  witnesses sealed provider FSINFO refusal, real OPEN/CHDIR
  refusal, unchanged cwd, subsequent DTFS retirement and
  restored underlying TSFS mountpoint access. Whole physical
  DTC3 tape unchanged. No production code or RAM change.
  This does NOT yet establish simultaneous Type-551 controller
  I/O, exclusive physical CLEAN/readback or removable-media
  generation safety. **P0-B remains OPEN, DTVOL2 stays RO.**
  v1568 **real physical DTVOL2 DATA partial and cold boot PASS**:
  exact DTC3/mount2 test-only RW after provider publication,
  genuine first-write DIRTY, native directory and DATA physically
  verified. Real Type-551 `PARTIALWRITE=0145:1` changed exactly
  one 36-bit DATA word, retained the second old word and complete
  valid DIRTY RBN0. Poisoned ownership retained. The exact tape
  on a second unchanged production KCORE cold boot refused both
  RO/RW and remained byte-identical. Zero production code/RAM
  change. **P0-B OPEN:** interrupted V2 directory, reel exchange,
  reinsertion/replacement and RW cutover. Production V2 remains RO.
  v1569 **physical directory partial write and second cold boot PASS:**
  the test-only exact DTC3/mount2 writable path physically armed
  V2 WRITE_INTENT and wrote initial RBN0144. A subsequent CREATE
  encountered real Type-551 `PARTIALWRITE=0144:1`; a linked VFS
  breakpoint allowed full-image byte comparison proving exactly
  one changed directory word, unchanged valid DIRTY RBN0 and all
  other blocks. Guest retained the poisoned mount with healthy TSFS.
  An entirely unchanged production cold boot of the *exact* damaged
  tape refused both RW/RO and left full-tape SHA-256 identical.
  Zero production source/RAM change. **P0-B OPEN:** physical reel
  exchange/generation interlock and same/replacement reel policy
  during writable operation. Production DTVOL2 remains READ-ONLY.
  v1570 **real V2 poisoned-reel same/replacement and two production
  cold boots PASS:** private op026 now admits a different, fully
  CLEAN and salted DTVOL2 with valid native directory only for a
  poisoned V2, preserving V1-to-V1 and all prior owner/MM barriers.
  Native source-linked PDP-6 30 cases/eight unsafe mutants, v1558
  20/six and real full V1 v1531 admin regression PASS. Isolated
  exact DTC3/mount2 test physically wrote DIRTY, suffered a real
  Type-551 one-word partial DATA write, and rejected the same
  physically reinserted old reel even after explicit FD/cwd release.
  Independently stamped different V2/CLEAN physically retired old
  mount and mounted RO; original and new reel SHA-256 unchanged.
  Two further wholly unmodified production cold boots rejected old
  DIRTY RO/RW and mounted new CLEAN RO with no tape changes.
  **P0-B OPEN:** Type-551 has no atomic reel-generation/interlock
  guarantee during a subsequent physical WRITE. Specify/verify a
  physical or operating media-retention contract before allowing
  production V2 RW. No fixed KCORE or mount-slab RAM growth;
  optional DTFS MRES TEXT only; v1555 RO mandatory.
  v1571 **silent physical reel exchange NEGATIVE PASS (two complete
  unchanged production PDP-6 boots):** valid V2/CLEAN DTC3 mount
  was swapped to a distinct fully valid V2/CLEAN ID during blocked
  PID1 CTY with zero DTC3 I/O/error. Original FSINFO/STAT/CHDIR
  and UNMOUNT succeeded; old owner had NOT rechecked new RBN0.
  Both independent reel full SHA-256 digests were unchanged.
  v1517 present-unit CONI status was rerun PASS but has no sticky
  reel-change indication. **P0-B OPEN:** no hardware no-exchange
  guarantee binds RBN0 validation to a later WRITE. Physical
  locking/interlocked write path required; software-only status,
  renewed reads and an operator acknowledgment are insufficient.
  Production v1555 DTVOL2 READ-ONLY mandatory; ZERO production
  code, resident or MRES growth in v1571.
  v1566 **physical DIRTY->CLEAN prerequisite PASS:** real linked
  DTFS private MRES op027 uses exact owner/VFS quiescence and
  72-bit identity. The existing dynamic MM scratch is pinned,
  WR_UNCERTAIN published BEFORE raw I/O; after full physical
  DIRTY admission, CLEAN write and independent readback, the
  service frees MM and clears only volatile ARMED/uncertain.
  VFS rescans owners after sleeping I/O and retains poisoned mounts
  on partial failure. 84 source-linked native PDP-6 cases, 34
  rejected unsafe mutations; full KCC build PASS, +28 fixed TEXT
  words, no added fixed DATA/BSS. A disposable physically booted
  PDP-6 image performed two genuine Type-551 DIRTY->CLEAN cycles
  without altering final full-tape SHA-256. The production v1555
  READ-ONLY veto remains UNCHANGED. **P0-B OPEN:** media-generation/
  exchange, partial physical DATA/DIR/CLEAN, reinsertion and
  post-RW cold boot and rollback acceptance.
  v1567: **actual PDP-6 Type-551 interrupted physical RBN0 CLEAN and
  second unmodified production cold boot PASS.** Eleven physically
  booted DTVOL2 crash images (ten RO/RW refusals plus one valid CLEAN
  negative control showing DATA corruption undetected) and full
  SHA-256 immutability. A test-only source-matched image physically
  armed DIRTY with production v1551/v1552, then used SIMH's genuine
  `PARTIALWRITE=0000:1` to interrupt unmodified v1566 CLEAN after
  exactly one changed 36-bit word. Exact instruction breakpoints,
  flushed on-media record comparison and real sticky VFS refusal
  PASS. A fresh completely unmodified KCC boot refused both RO/RW
  of the same torn tape without publishing a mount. Original v1529
  physical partial DATA regression PASS. Production changes/RAM: 0.
  **P0-B OPEN**: no Type-551 mechanical reel-generation lock;
  partial V2 DATA/DIR and removable exchange/replacement/reinsertion
  after user RW still unproven; DTVOL2 remains RO.
- [ ] **P1-C:** real concurrent process swap/restore with MEMFS
  persistent snapshot and independent LOGSTORE fault activity;
  check backing bitmap, MM and cache ownership isolation.

Latest stage: `STATUS/KCORE-FEATURE-STATE-V1567.MD`.
Critical fail-stop acceptance: `STATUS/KCORE-FEATURE-STATE-V1548.MD`.

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

## 2026-11-13 v1545: real secondary D6FS DSK270/DRM236 outage

- [x] 256K PDP-6 selected DRM236 D6FS ROOT with independent
  mounted DSK270 D6FS; physically detach all DSK secondary
  members, observe nonfatal failed file READ, root READ survives.
- [x] Inverse DSK270 selected ROOT / DRM236 mounted secondary:
  after physical DR0..3 detach secondary read returns error and
  root read survives. Real `dsys_d6fs_mount`, `d6fsck`-validated
  14-word handoff and mount ID2/provider6; six corrupt-media
  and five wrong-ABI negative test controls rejected.
- [x] Correct transient KINIT DSK direct filesystem patch sites
  7/8/11/12 from raw low-level service to existing public
  Type-270 address-conversion wrappers; no new fixed words/RAM.
- [ ] Remaining P0-A shared ROOT/BACKSTORE DEVICE/VOLUME fault
  ordering and distinct critical owner HALT reason; retain
  authoritative ownership without fabricated release.

## 2026-11-13 v1544: D6FS/DSK/DRM source-linked isolation

- [x] Native 24-case D6FS read/write/physical DSK270/DRM236
  root-versus-secondary integration, including three-member
  mapped sets, six unsafe mutants rejected and normal full
  KCC KCORE/D6FS MRES link PASS. Zero executable/RAM change.
- [ ] Real full-guest selected TSFS or D6FS root with a separate
  **validated and mounted** D6FS secondary, physically failed
  DSK/DRM I/O, nonfatal error return and healthy root progress.
  Test-only error callbacks alone are insufficient.
- [ ] Mixed ROOT/BACKSTORE shared DEVICE/VOLUME fault
  qualification remains separate; no owner release on failure.

## 2026-11-13 v1547: shared physical DRM-root BACKSTORE-first

- [x] Full 256K KCC PDP-6 `ROOT=drum BOOT=drm`: four-member
  D6FS root and mapped swap share DR0..DR3. One selected
  Type-236 PI2 error for owned BACKSTORE WRITE HALTs at
  distinct swap PC `003747`; octal `0320` blocks retained,
  root/volume/device mapping and nine mutation guards PASS.
- [ ] P0-A final combined physical gate: ROOT-first failed
  actual transfer *while BACKSTORE still owns blocks*.
  Native v1546 proves both orders; physical v1547 proves swap
  first. No new KCORE code/RAM; 32K/optimization HOLD.

## 2026-11-13 v1546: shared critical-service native order

- [x] Source-linked PDP-6 D6FS root and BACKSTORE share one
  DSK270/DRM236 or mapped VOLUME physical unit. Twelve cases,
  five unsafe source mutants PASS; exact distinct HALT,
  unchanged owned blocks and correct no-second-operation order.
- [ ] Physical full guest with actual shared ROOT/BACKSTORE
  DEVICE/VOLUME failure and live swapped owners remains P0-A.
  No production code or RAM growth. Strict 32K HOLD.

## 2026-11-13 v1543: real DRM236 D6FS root loss accepted

- [x] The KCC `ROOT=drum BOOT=drm` PDP-6 guest mounts D6FS
  from DR0..DR3, detaches all four selected root members,
  and HALTs at the exact root PC 003743 on required root
  `/SYSTEM/INIT` physical data I/O. Type-270 v1540 root HALT
  also accepted. No production source/RAM change in v1543.
- [ ] Remaining P0-A: actual failed noncritical secondary
  DSK270/DRM236 I/O must not stop another selected root; a
  mere unused-drive DETACH is insufficient proof. Combine
  ROOT/BACKSTORE DEVICE/VOLUME physical faults and verify
  exact owner reason. Do not reopen the accepted policy.

## 2026-11-13 v1542: physical critical BACKSTORE acceptance

- [x] Full 256K KCC PDP-6 real Type-236 PI2 error completion for a
  valid owned BACKSTORE WRITE HALTs at v1515 reason-2 PC 003747.
  Error counter 1, enabled 1, used octal 0320; four clean
  pre-fault samples. Nine corrupt physical witness mutations
  rejected; no production KCORE source/RAM change in v1542.
- [ ] Remaining P0-A hardware profiles: alternate DRM-root,
  independent secondary DSK/DRM negative fault and combined
  root/BACKSTORE DEVICE/VOLUME fault. Native v1541 already
  covers direct/mapped source paths; physical profile still open.
- (Historical, superseded) P0-B and P1-C/D/E stay strictly separate; do not reopen
  the accepted selected-root or BACKSTORE policy.

## 2026-11-13 v1541: current critical storage P0-A policy

- [x] Required selected root filesystem and active/owned BACKSTORE
  now both use fatal service-level classification after an exhausted
  actual read/write, not speculative permanent-device-loss bits.
  Root owner/PC is distinct from BACKSTORE reason 2 HALT.
- [x] v1541 BACKSTORE direct DSK270/DRM236 and mapped VOLUME
  actual negative transfer routes to BACKSTORE HALT when enabled
  or still allocated. Inactive/unowned KINIT probe and invalid
  requests do not HALT. Source-linked PDP-6 17 cases/six mutants
  and KCC tape-root fixed+FS MRES link PASS, no new DATA/BSS.
- [x] v1538/v1539/v1540 root physical acceptance covers all-
  member TSFS, one-member TSFS, and D6FS disk-root loss at
  the exact v1515 root HALT PC; independent secondary DTC3
  physical failed write remains nonfatal (v1534).
- [ ] v1541 BACKSTORE complete full-guest Type-236/270 physical
  positive failure and exact **swap**, not root, HALT PC. Earlier
  v1511/v1512 returned-final-error scheduler retries are
  superseded intentionally, not silently counted as passing.
- [ ] Alternate DRM root, independent DSK/DRM noncritical loss,
  shared DEVICE/VOLUME failure and mixed root/BACKSTORE acceptance.
  P0-B and P1 tasks remain defined by DAIMOS current `TODO.MD`.

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
