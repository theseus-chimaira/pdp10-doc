## 2026-11-26 v1955: active disk-backed swap

Current KCC 64K D6FS lowmem disk root, v1954 SIMH. PID1
enables swap through STORAGECTL, six native non-PURE children
grow to 8192-word APRs and block on a coordination pipe.
Actual A/B/C MM9/20/8, free43663/1119/43677. Slot2 DISK
run0..63 exactly matches 64 live bitmap bits and counters.
Disk reads50/129/193, writes1/65/65, errors0. All six
children validate PID-specific canaries at 02500 and 017000,
release token and exact EXIT/WAIT status. Final bitmap0,
same parent APR/u-area and stable non-cache/non-pipe owners,
no child/pipe owner or reservation, idle-stack pin only.
Harness intentionally terminates sleeping PID1 after marker,
not a guest HALT. First run retained PID1 pipe; revised
fixture closes both parent FDs and passes the exact oracle.
No production change. Testkit a762a86, evidence
$TMPDIR/k09-disk-swap-v1955.MbLEZM/physical-final.
This accepts ordinary active DISK restoration, not provider
loss or interrupted WRITE recovery.

## 2026-11-26 v1953: organic descriptor exhaustion

48K KCC GENERAL noaux with v1950 SIMH. Four native children
each allocate three pipes and block on the coordination pipe.
PID1 fills remaining MM entries using ordinary PIPE, closes and
recreates one pipe, checks refusal again, releases children and
checks four exact WAIT results. Original physical MM A/B/C/D:
9/32/32/7 entries; free24247/14904/14904/24390. B/C largest14789.
The exhausted table has capacity32, with abundant contiguous
space and available FD pair capacity. Four additional parent
pipes were admitted; the fifth refused. All children resident.
No packed or child ownership remains after WAIT; exact baseline
owners preserved except closed 14-word coordination pipe and
reclaimed 129-word block cache. Baseline idle-stack pin remains.
Guest completion marker followed by deliberate harness stop,
not HALT. No production source change or instrumentation.
Testkit 77a2a9c, evidence $TMPDIR/k09-descriptors-v1953.XFiqba.

## 2026-11-26 v1953: 64K active KCC lifecycle acceptance

The unmodified v1951 64K nine-child mixed executable builds
were executed with the v1950 simulator. Both GENERAL and STRICT
complete actual RAW packing of nine children and refuse the
tenth RUN despite 5072 free words (largest extent 847).
MM26 at both stable pressure gates. Actual FULL entry 042435
has opcode 200000000004; slot0 operates on child3 PACKED
151000..160000 and PROCESS160000..170000, both pinned once.
FULL free4695, pinned7936 including the 256-word idle stack.
Nine exact PID/status WAITs, child and parent data checks,
no reservation or child/pipe owner, stable parent VM/u-area,
high slot2, MM8, free39589 in one extent and normal HALT.
Both traces pass the original and independent conservation
verifiers. Four modified-observation controls per trace reject
unretired slots, lost parent ownership, leaked pins and live
reservations. These controls are not live fault injection.

The unchanged native MM exhaustion regression also distinguishes
-1 total shortage, -2 fragmentation and -3 descriptor exhaustion,
recovers after freeing a descriptor, and rejects both wrong
error-handling implementations. No production kernel change.

Functional acceptance is separate from optimization. Per v1952,
the 16K target and size tuning are deferred; neither is a K09
closure gate. RAW/mode3 remain OFF by default.

## 2026-11-26 v1951: physical KCC resident budget

Matched 64K lowmem KCC TSFS boots without AUXSTORE, using the
v1950 simulator cb6a255, stop at actual INIT V1 and inspect the
original MM table, arena, service jumps and resident immediate.
No kernel instrumentation. Fixed KCORE bytes are identical.
The installed code/data are 16966 words OFF and 20845 RAW.
The runtime-selected 40-descriptor MM table consumes 80 words.
SYS_MEMINFO already includes that table (17046 OFF / 20925 RAW);
do not count it twice. The physical 060-word vector reserve is
separate. Actual non-process boot kernel dynamic allocations
are 614 words on both configurations; PID1 has 4096 VM words
and a 264-word u-area. Managed arena sizes 48442 / 44563,
allocated 4974 on both, free 43468 / 39589. The complete arena
and resident partition conserve all 65536 physical words.

The installed non-PURE modules total 5852 words: CTY116,
CLK41, PCLK21, MONITORFS1597, DCS62, GE96, TTY165, TAPE406,
DSK211, DRM149, DTFS2988. The linked software-clock fallback
398 words is not installed with PCLK. Disk SWAP is not installed
without BACKSTORE. PURE adds exactly 3879 words, zero separate
managed allocation at this gate. A fixed resident 16K ceiling
already fails by 662 words OFF and 4541 RAW at 64K, before
managed scratch. The 48K RAW resident immediate is 20909,
including the 64-word MM table. Neither endpoint nor guest
free words alone proves the target's resident budget.

Linked KINIT image at 030000 plus its 0400-word stack reserve:
OFF 27911, GENERAL RAW 32056, STRICT RAW 30172 words.
These are linked transient reservations, not observed peak live
RAM or restore high-water. STRICT excludes the linked disk
SWAP candidate but does not reduce installed noaux residency.

At 512-word RAW prefixes, five packed children save 2560 owner
words gross, less than PURE's 3879 installed words. Eight such
owners are required just to offset the module, excluding
transient restore allocation. The correctness tests are not
a demonstration of net RAM savings. Keep RAW/mode3 default OFF.

Evidence: $TMPDIR/k09-budget64-v1951.d5Qr6j. The independent
verifier reconciles actual physical MM with linked package sizes;
four altered snapshot controls reject bad root, resident count,
service target and overlapping extents. Not live injected faults.

## 2026-11-26 v1950: AUTO DSK AUXSTORE selection

The v1948 FULL AUTO/DSK root-selection failure is corrected
in the simulator, not in DAIMOS. See DSK270.MD for the DEC
H-270 octoflop contract and supersession of the speculative
v1949 CLEAR reset. With corrected simulator cb6a255, all six
FULL AUTO/explicit root cases pass on unchanged KCC images.
AUTO TSFS with DSK AUXSTORE now selects disk SWAP with 512
backing blocks, not PURE. Active swap/RAW pressure acceptance
remains separate. No permanent kernel change.

## 2026-11-26 v1948: FULL KINIT disposable-prefix sizing

KINIT must make a contiguous low region available to MINIT before
installing the resident packages. This region includes the already
copied KCORE source and the copied-out MINIT table, but must not
overlap still-live KINIT instructions. The linked resident footprint
rather than the encoded package footprint determines the bound.

The PDP-6 boot build now computes the required prefix from the
linked KCORE endpoint, the maximum MM extent table (two words per
descriptor), and the largest allowed resident package combination.
The alternatives are block-root with disk swap, tape-root with disk
swap, and tape-root with PURE. Common optional packages are counted
conservatively. Any deficit is emitted as disposable zero padding
before the embedded KCORE slot. The KCORE slot itself retains its
exact initialized size. KINIT reclaims the entire prefix through
the existing MINIT arena publication; no new permanent word,
runtime allocation or ABI is introduced.

In the KCC FULL AUTO RAW image the original contiguous low arena
was [047615,055172) at DTFS install; the package needed 005654
words, ending at 055471. All addresses are octal. The old image
correctly halted rather than installing across live KINIT.
The new prefix ends at 057445, adding 1195 decimal transient words.
The FULL RAW-disabled and 64K lowmem GENERAL/STRICT images need
no padding. The fixed KCORE endpoint remains 025632; KCORE,
PURE and SWAP binaries are unchanged. The corrected FULL image
boots TSFS without auxiliary backing and installs PURE, and boots
TSFS with valid DRM auxiliary backing and installs disk SWAP.
Disk and drum roots also install disk SWAP. These observations
verify root/service installation at INIT, not swap data traffic.
The overall K09 production acceptance is separate.

## 2026-11-26 v1947: integrated K07/K08 physical acceptance

K07 executable backing and K08 scheduler implementation are closed.
The full uninstrumented 48K PDP-6 TSFS/PURE replay completed in
284.728 seconds, with the same production images as v1945. At the
actual FULL entry, slot3 PACKED113600..120000 and PROCESS122000..130000
each have pin1. Real PCLK PI6 interrupts TAPE PC032060 without
changing either MM owner or any of the 2176 retained RAW words.
Actual DTC3 detach during tape search reaches the original 7200-tick
timeout, and FULL returns -4 with source pin0, no destination,
original backing and volume ID intact. The subsequent real PI6 and
slot4 execution while DTC3 is absent establish independent progress.
The foreign CLEAN DTVOL2 medium fails a new PREFIX restore at
PC034446/-4, 302 ticks after the missing-source FULL failure.
The original medium then permits READY_ONCE publication for slot5
and slot3. Slot3's actual service return publishes cursor01003,
sole unpinned PROCESS122000..130000 and stable u-area051274;
the original scheduler consumes the hint, selecting AC1=3/cursor3
without changing MM13. Exact five child data/EXIT/WAIT witnesses,
parent data, final UNMOUNT, MM8, high slot2, mode word0, healthy
retired medium0100003, root0, and normal HALT PC0 all pass.
Both reels are unchanged. Independent full and supplementary
oracles reject 14 altered observed traces (not additional guest
fault injections). The v1945 same-PC repeated-stop witness remains
withdrawn; the complete later v1947 replay replaces it. No code,
image, ABI or resident RAM change. PI6 observed inside FULL and
after rollback, not inside PI-disabled sections. Wider physical
root/controller matrices and KCC production acceptance remain K09,
KA10 remains K17. RAW PURE/mode3 remain OFF by default.

## 2026-11-26 v1946 acceptance correction

K07 code complete, integrated foreign-to-original recovery/WAIT
replay pending. The earlier v1945 same-PC foreign checkpoint
was invalid: actual SIMH repeats a cleared/rearmed breakpoint
without executing the stopped instruction. Genuine missing-medium
rollback and original-medium recovery/WAIT remain valid.
Independent v1946 physical verification establishes real pinned
FULL PI6 at TAPE PC032067, actual 7200-tick device-search timeout
and FULL -4, another child's execution with DTC3 absent, and
a new foreign PREFIX -4 299 ticks after the missing refusal.
All 2176 retained RAW words and original identity remain intact.
The complete v1946 run exceeded its 260-second diagnostic deadline
during recovery; no slot3 READY_ONCE dispatch or final WAIT claimed.
No production code or RAM change. The v1945 closure and foreign
witness statements below are historical and superseded by this note.

## 2026-11-26 v1945: K07 executable backing lifetime closed

K07 closes after the v1940 native matrix, v1942 physical FULL/WAIT,
v1944 real PI6 owner-admission orderings and v1945 physical DTC3
source loss, foreign-volume refusal and original-medium recovery.
Both failed FULL transactions retain the exact original unpinned
2176-word PACKED suffix and original backing, retire the temporary
target, and retain READ_SUSPECT. Original reel return permits five
exact WAITs and leak-free retirement. No additional resident RAM or
production image changes. See `PURE-K07-SOURCE-RECOVERY-V1945.MD`.
K08/K09 and native KA10 K17 remain open; RAW PURE and mode3 remain
OFF by default. Dated development-branch notes below are historical.

## 2026-11-19 v1684: two RAW PURE executable owners during FULL (C89 native PASS)

The source-linked daimos-testkit `pure-multiowner-source-v1684`
PDP-6 C89 test executes two original zero-copy `proc_pure_ram_drop_v1381`
transactions, original MM PURE rebase assembly, source-backed
`proc_swap_mount_busy` and `proc_swap_backing_busy` assembler, and the
bounded `proc_pure_ram_restore_v1391` transaction. Two independent
PACKED processes own executable identities on different mounts.
During the first process's FULL callback, the original MM contains
its PINNED RAW suffix, PINNED full APR target, and an independent
unpinned PACKED second suffix. The guards protect both executable
sources without changing any original MM owner. On returning from
FULL, the restored RESIDENT process **continues** to protect its
executable identity, because it can become PACKED again. The second
owner's descriptor and retained writable words remain unchanged.
When its mount root is absent, the original FULL source admission
returns `MM_ERR_INVAL` without consuming its RAW suffix or clearing
its TRANSITION. Mount-root return restores its original guarded
backing identity.

Two test-only mutations of the **original** mount and vnode/backing
guard assembler are rejected by the new-only C89 PDP-6 oracle, with
exact guest exit codes `000000004004` and `000000004200`. The
existing v1649 single-owner FULL guards and the v1648/v1395 rollback
prerequisites also pass. No production KCORE/MM/MRES source, ABI, RAM
or boot image was modified. Acceptance is **native component scope**;
this is not the nine-owner physical PI6/secondary-DTFS loss/unmount/
rename or intermediate high-water release gate. See DAIMOS
`design/kcore-pure-multiowner-source-lifetime-v1684.md` and
`daimos-testkit/tests/validation/pure-multiowner-source-v1684/README-v1684.md`.

## 2026-11-19 v1674: 48K PDP-6 physical RAW PURE high-water PASS

On the unmerged DAIMOS PURE development branch, the completed v1674
physical PDP-6 GENERAL/STRICT swapless 48K high-water test replays
**all 3122 original MM physical descriptor writes** from CTY A through
B to C against source-built kernel symbols. Both runs use byte-identical
KCORE and PURE MRES images; no DSK/DRM swap media is attached. Test-only
PID1 uses deterministic native PIPE-blocked children, not an expiring
SLEEP, and does not instrument KCORE or the production scheduler.

Both profiles pass the source-linked native RUN/EXIT/WAIT and original
MM process/owner/pin/SYS_MEMINFO tests. Nine PURE RAW 128-word prefixes
are dropped then restored. The tenth real RUN is refused under APR
fragmentation. Original free MM words are **27339 -> 2692 -> 27339**,
with a largest extent of 1539 at B. The lowest observed valid
*committed* MM free count anywhere between A and C is **1255 words**;
maximum nonfree physical RAM is **47897 of 49152 words**. Each profile
has 194 extent-count commits, five in-place BRK changes, and nine PURE
reclamation plus nine restoration span writes. The general/strict
comparison and both independent physical verifiers PASS; each original
physical recording rejects five physical mutations plus a forged
summary. Their accepted high-water and state reports are byte-identical.

This result is an **event-commit high-water** for the stated 48K
workload, not an execution-time-neutral scheduler worst-case proof.
SIMH write watches affect elapsed time; intermediate descriptor-array
shifts are replayed but not mistaken for actual committed allocations.
This does **not** close PI6 fault-injection, ROOT=auto/DSK/DRM, KCC
FULL-profile, compressed mode3, KA10, 32K boot or mainline integration.
See DAIMOS `design/kcore-pure-runtime-highwater-48k-v1674.md` and
daimos-testkit `tests/validation/pure-runtime-highwater-v1674/README-v1674.md`.

## 2026-11-19 v1673: development-branch physical RAW reclaim PASS (48K only)

The unmerged DAIMOS branch `kcore-pure-reclaim-pressure-v1673-20261118`
(commit `ae84bb37`) has source-matched original physical PDP-6
48K evidence for PURE RAW-suffix reclamation under APR fragmentation.
Two retained general/strict comparison pairs pass the original testkit
oracle; nine PACKED owners save 128 words each (1152 total). Original
MM free words are 27441 before RUN, 2794 under reclaim, and 27441
again after all EXIT/WAIT. The largest free extent at pressure is
only 1641 words. Stable physical MM owners, pins and native MEMINFO
are validated; five physical witness mutations and a forged summary
are executed and rejected. This is source-linked physical evidence,
not a mainline production acceptance, not a newly completed paired
simulator invocation, and not a continuous runtime high-water proof.
Bitmap mode3 is OFF. PI6 fault pressure, other root classes and KA10
remain unaccepted. See DAIMOS
`design/kcore-pure-raw-reclaim-48k-v1673.md` and testkit
`tests/validation/pure-runtime-reclaim-v1673/README-v1673.md`.

## 2026-11-17 v1645: physical PI6 partial wake under immutable PURE backing

Unmodified original 48K PDP-6 no-AUX TSFS/PURE kernel with C89
source-derived guest executing mode0777 compressed PURE from a
real native DTC3 READ/WRITE DTFS mount2. Six actual sleeping
PACKED child owners with exact retained `006400` octal words
each (this source's DXR2 TX2 text001454, omitted001400).

Before waking any child, the parent performs the v1644 UNMOUNT,
UNLINK, TRUNCATE, O_TRUNC and data WRITE refusal checks plus
the safe in-place native DTFS rename COMP->SAFE->COMP (`P`).
It publishes EXACTLY ONE real PIPE word (`H`) while the writer
stays OPEN; it repeats all source guards (`P`) during the ensuing
real PI6 clock progression. A true `dsys_sleep(0200)` gives the
awakened child scheduling time; that child reads the token,
validates private writable DATA and EXIT-057 (`w`) before the
parent's `T`. The other FIVE cannot see EOF because the PIPE
writer remains open; parent repeats the same guard checks (`P`)
while their original PACKED RAW sources remain immutable.

Physical original `clk_tick_count` and `proc_timer_clock`
both advance 1691 qualified PI6 ticks from H=8673 to T=10364.
The post-H guard at tick9098 still shows all SIX original
unchanged unpinned RAW owners. At T tick10364 and the next
guard tick11015, only child2 has retired; original RAW owners
3..7 remain unchanged and pin0. The original ROOT remains
TSFS mount1; native DTC3 mount2 RW (`vfs_mount_policy=1`).
Unmodified original KCORE `proc_swap_mount_busy` PC003431 and
`proc_swap_backing_busy` PC003322 are entered with correct
public mount2 and provider5/mount2/file slot0. The subsequent
final PIPE EOF lets the five other children run/EXIT-057;
all six exact parent PID/WAIT/status-057 checks pass. Parent
unmounts DTC3 successfully (`U`), then `Z` and normal HALT.
Source-linked U/Z physical MM zero child PROCESS/PACKED owners
or leaked pins. Full console ends `LHPwTPwwwwwUZ`.

The physical auditor reuses v1644's strict backing guard oracle,
adds real PI6/CLK/owner tests, and rejects forged CLK, owner pin
and RO-policy traces. Native v1086 PI6, v1563 PURE and v468
rename/PI6 tests PASS. No production or resident RAM changes.

This ACCEPTS genuine single-child user/PI6 wake with five still
PACKED immutable sources. It does not prove an instruction-
level in-flight WRITE/UNLINK or UNMOUNT vs NEW PURE RUN/EXEC
admission race, or simultaneous bounded FULL target at a
guard checkpoint. Those, uncertain MM rollback, 32K/64K
regressions and complete transient peak RAM remain OPEN.
The old native DTFS v1522 source fixture remains stale.
PURE OFF by default; strict 32K optimization HOLD. DAIMOS:
`design/kcore-p1e-pi6-staged-backing-v1645.md`.

## 2026-11-17 v1644: writable PURE backing guard and owner release

Original source-built PDP-6 48K PURE/TSFS/no-AUX guest, unmodified
KCORE, MM, VFS and DTFS. Unlike v1643's read-only secondary
backing, v1644 mounts real DTC3 native DTFS **READ/WRITE** with
the compiled compressed PURE source at `/MOUNT/COMP`, mode0777.
The fixed VFS policy word is `000000000001`: bit0 selected ROOT
RO, bit1 secondary DTC3/RW. Six actual RAW PACKED_PROCESS
children, each retained suffix 006400 octal words (different
source: original compiled TX2 text001420, omitted prefix001400).

Source-linked original KCORE physical breakpoints show
`proc_swap_mount_busy` PC003431 AC1=2, and
`proc_swap_backing_busy` PC003322 AC1=050202000000 (provider5,
public mount2, native file slot0). Original B/A/mount-entry/
vnode-entry/P physical MM snapshots retain six exactly matching
unpinned PACKED owners; the selected TSFS root remains unchanged.

Actual user syscalls refuse UNMOUNT, UNLINK of original or
renamed vnode, TRUNCATE, O_TRUNC, and WRITE_WORDS even through
an O_RDWR file descriptor. RENAME COMP->SAFE->COMP succeeds:
native DTFS preserves the existing file slot/content and merely
changes its directory name, so the backing record still refers
to the same immutable source. All six original child PIPE/private
DATA/EXIT-057 and exact parent PID/WAIT/status-057 checks pass.
Only AFTER those child owners retire does normal native DTFS
UNMOUNT succeed. Console ends `LwwwwwwUZ` and normal HALT;
final MM U/Z has no child PROCESS/PACKED owner or child pin.
The source-linked auditor PASSes and rejects forged RO-policy,
still-pinned source and wrong mount identity witnesses.

No production edit or additional resident KCORE RAM. This
accepts sequential PURE backing lifetime, not concurrent PI6
race freedom or uncertain MM rollback. General 32K/64K
regressions, matched transient peak RAM and old native DTFS
v1522 fixture modernization remain open. PURE OFF by default;
strict 32K optimization HOLD. Detailed DAIMOS report:
`design/kcore-p1e-backing-guards-physical-v1644.md`.

## 2026-11-17 v1643: real secondary DTFS FULL read fault recovered

The original physical PDP-6 48K no-AUX/TSFS/PURE guest is
source-derived from v1638 without modifying KCORE/MM/MRES/VFS.
PID1 copies the original compressed DXR2 from the selected TSFS
root to a real native DTC3 DTFS file `/MOUNT/COMP` on separate
public mount2, closes the file, CHMODs 0555, performs checked
CLEAN unmount and remounts the SAME reel read-only. The selected
TSFS ROOT (provider7/mount1, DTC0..2) remains physically online.
The first wrong test-only target `/MOUNT/DT0` was absent from
the immutable root; correcting the guest to the existing
`/MOUNT` fixed that mount failure. No kernel workaround.

Original B/A MM shows six genuinely PACKED unpinned children,
each RAW `006600` octal words. This different source includes
extra text: compiled DXR2 TX2 `001227`, actual original PDP-6
skip `(text+020)&~0177=001200`, retained `010000-001200=006600`.
Do not confuse this with earlier v1638 `007000` RAW words.

At source-linked original FULL loader PC 035252, slot3 owns
concurrent pinned source PACKED 111200..120000 and full PROCESS
APR 120000..130000. The test simulator physically detaches ONLY
DTC3. The original non-root backing read fails; v1391 returns
`-4` at v1612 service PC 032461, RAW suffix unchanged/pin0,
full destination freed. The unchanged TSFS root never HALTs.
The same DTC3 reel is reattached. A subsequent original bounded
FULL callback for compressed slot4 returns 0, and all six
original PIPE/private mutable sentinel/EXIT-057 and exact
PID/WAIT/status checks succeed; `LwwwwwwZ`, normal HALT and
final MM no child owner. Source-linked physical audit and
forged pin/fake success controls PASS/reject. No production edit.

Open: physical immutable source modification/unmount guards,
PI6/uncertain MM rollback, 32K/64K profiles and matched complete
transient peak RAM including both pinned FULL extents. Original
READ_SUSPECT media bit was not sampled. Older native DTFS v1522
testkit source extraction is stale after v1575/v1561; it has
NOT been accepted as a current-tree PASS. PURE disabled by
default; strict 32K optimization HOLD. DAIMOS report:
`design/kcore-p1e-secondary-backing-physical-v1643.md`.

## 2026-11-17 v1642: real selected-root read failure during PURE FULL

Source audit of current TSFS/VFS and the fixed v1538 root
classifier confirms that an otherwise valid failed I/O from
selected namespace ROOT must HALT rather than return a normal
filesystem error to the restore transaction. In the original
48K PDP-6 TSFS/no-AUX/PURE six-PACKED mixed executable guest,
original bounded FULL callback PC 035252 is reached with
compressed slot3 owning pinned RAW suffix 111000..120000 and
separately pinned complete PROCESS APR 120000..130000.

Testkit physically detaches all three actual Type-551 root
DECtapes DTC0, DTC1 and DTC2 at the suspended callback. Each
device reports no attachment. Next real TSFS read failure
executes fixed KCORE classifier PC 003761, AC1=7 TSFS and
AC2=1 public root mount, matching source-linked
`vfs_namespace_root`. Original MM extents/pins remain unchanged
until root-specific terminal HALT PC 003776. Neither normal
PURE rollback nor user EXIT/WAIT is permitted on this path.

Source-linked physical testkit auditor PASSes and rejects
forged non-root mount and altered root HALT witnesses. A
testkit detach-marker indentation bug was fixed and the full
physical run repeated before accepting. No production change.

Still OPEN: a recoverable physical FULL failure using a SECONDARY
non-root source; immutable source/unmount protections; concurrent
PI6/uncertain MM rollback; general 32K/64K cache layouts and
matched transient peak RAM. PURE OFF by default, strict 32K
optimization HOLD. DAIMOS report:
`design/kcore-p1e-tsfs-root-read-failstop-v1642.md`.

## 2026-11-17 v1641: physical rollback after one injected loader error

The original 48K PDP-6 no-AUX TSFS compressed PURE executable,
MRES and kernel remain unchanged. Read-only MM at the genuine
bounded FULL callback return PC 034361 confirms real loader
AC1=0, with slot3 simultaneously pinned PACKED source
111000..120000 and PROCESS full APR destination
120000..130000 octal words. Testkit deliberately injects
ONE error by setting ONLY simulator AC1=-1 before returning to
the existing v1391 transaction. This models an error returned
by a loader, not a real DECtape/VFS read failure or altered
immutable executable source.

The original transaction returns -4 to PURE service PC 032461.
At this breakpoint slot3's exact PACKED suffix remains owned
and unpinned, while the distinct PROCESS target no longer exists.
Other child extents and pins are unchanged. Another successful
FULL callback is later observed for slot4. All six original
child PIPE/private data/EXIT-057 and exact parent WAIT checks
finish with normal HALT and no remaining child MM owner at Z.
Source-linked physical auditor PASSes, rejecting a fake retained
source pin and a fake original loader error before injection.

This closes test-injected callback-failure owner cleanup,
in addition to v1640 natural FULL allocator NOMEM rollback.
Real VFS/source-loss/immutable-unmount failure, concurrent PI6/
uncertain MM rollback, 32K/64K cache regressions and matched
transient peak remain OPEN. PURE OFF by default; strict 32K
optimization HOLD. DAIMOS report:
`design/kcore-p1e-full-loader-error-rollback-v1641.md`.

## 2026-11-17 v1640: actual physical FULL NOMEM rollback and retry

The original 48K PDP-6 TSFS/no-AUX low-BCACHE PURE kernel/MRES
was not modified. One compiled C89 guest runs compressed PURE
children in slots 2,4..7 and plain PURE slot3, all six
actually PACKED before the seventh RUN is correctly refused.
The original v1391 full-APR MM allocation returns NOMEM -1 at
physical PC 034336. Only the slot2 PACKED source 121000..130000
has a pin; no destination was allocated. The original service
return PC 032461 preserves that -1, the identical six PACKED
owners, ALL pins zero, and no premature resident publication.

After the plain child restores and releases an APR, original
bounded FULL loader PC 035252 executes for SAME compressed
slot2. At the call, MM has separately pinned PACKED source
121000..130000 and PROCESS destination 110000..120000 octal,
with AC4=010000 trusted allocated APR words. Original six
child PIPE/private mutable sentinels/EXIT-057 and six parent
exact PID/WAIT/status-057 checks all pass, terminal LwwwwwwZ,
normal simulator HALT; final Z has NO child MM owner.

The source-linked physical audit PASSes and rejects a forged
still-pinned RAW owner and fabricated success at the original
NOMEM instruction. Natural FULL refusal/unpin/retry/cleanup
is now accepted. Physical VFS failure/source-loss/unmount and
concurrent PI6/MM uncertain rollback remain OPEN, as do 32K/
64K cache regressions and transient matched peak RAM. PURE OFF
by default, strict 32K optimization HOLD. DAIMOS report:
`design/kcore-p1e-full-nomem-physical-v1640.md`.

## 2026-11-17 v1639: matched six-owner physical RAM savings verified

Original 48K PDP-6 TSFS/no-AUX low-BCACHE C89 guest, compiled
INIT DXR and fixed KCORE linker map are byte-identical between
PURE-on and PURE-off builds. Both admit six children, refuse
seventh RUN, and complete all six exact original PIPE/mutable
sentinel/EXIT-057 and parent PID/WAIT/status checks, normal HALT.

Read-only source-linked complete MM snapshots at B/A: ON has
six 007000-word PACKED suffixes, occupied/free 066433/006564
octal; OFF has six complete 010000-word PROCESS APRs, occupied/
free 074433/004122. ON has +1314 decimal physical free RAM
words at equal six-child state, including installed 1758-decimal
PURE MRES. The 3072-decimal gross omission of six 512-word
prefixes less the 1758-word MRES gives exact +1314 physical gain.

After all six child WAITs, Z snapshots are child-owner clean,
but ON free 063461 vs OFF 067220 octal (-1887 decimal) reflects
1758 still-resident PURE words and the additional valid 0201-
octal/129-word BCACHE slab in the ON-only terminal snapshot.
Original MM/guest audit and two falsified owner/WAIT witnesses
PASS/reject. These are physical sampled free words, NOT matched
time-resolved peak RAM while the v1391 FULL loader pins both
source and destination. No production code changed in v1639.

OPEN: actual transient memory peak, source lifetime/unmount
failures, PI6/MM loader rollback, 32K/64K cache profiles and
final feature acceptance. PURE OFF by default; strict 32K
optimization HOLD. DAIMOS report:
`design/kcore-p1e-six-owner-physical-ram-v1639.md`.

## 2026-11-17 v1638: original physical FULL v1391 relocation accepted

Testkit uses the unchanged low-BCACHE PDP-6 kernel and its
optional PURE MRES in 48K no-AUX TSFS simulation. A plain first
PURE child and five compressed PURE copies of the SAME linked C89
executable are admitted. Seventh RUN refusal forces all six into
actual PACKED storage. The original parent then verifies all six
PIPE/private sentinel/EXIT-057 and PID/WAIT/status results,
`LwwwwwwZ`, and normal HALT.

At parent B and A there are six unpinned 007000-word PACKED
suffixes. The original MRES code at physical PC 035252 is
authenticated as the source-linked `exec_image_full_bounded_v1391`
loader (opcode 200000000004). The slot-zero scheduler calls it
with AC2=120000 full APR target and AC4=010000 trusted capacity.
At the SAME physical breakpoint, compressed child slot 3 owns
both PINNED original PACKED suffix 111000..120000 and distinct
PINNED PROCESS destination 120000..130000. No owner overlap or
fake prefix reconstruction; all children later EXIT/WAIT and
are released in final MM Z. The original physical testkit auditor
and forged FULL-owner/WAIT controls PASS/reject. v1630 had already
proven the FAST v1384 path. No new production code in v1638.

Still OPEN: interrupted/failed FULL and PI6/MM owner rollback,
immutable source loss/unmount, broad 32K/64K cache regression,
matched peak/net physical RAM and full P1-E feature freeze.
PURE default OFF; strict 32K KCORE optimization HOLD.
DAIMOS report `design/kcore-p1e-physical-full-relocation-v1638.md`.

## 2026-11-16 v1637: six physical 48K PACKED child restorations

With v1636 low BCACHE, the original PDP-6 48K no-AUX TSFS/PURE
guest admits six sleeping children, refuses a seventh RUN and
keeps the parent alive to check all six child PIPE/private data,
EXIT-057 and exact PID/WAIT/status checks (`LwwwwwwZ`, HALT).
Same compiled guest replayed with original source-linked MM B/A
tables contains six unpinned PACKED_PROCESS suffixes, each 007000
octal words, owners 7..2. Parent APR stays 130000..140000.
Final Z contains no child PACKED or PROCESS owner and no pin leak.
BCACHE is reclaimed under pressure and can return at low memory.
Original physical testkit auditor PASSes and rejects forged
PACKED ownership and false WAIT success. No new production code.
FULL relocated restore, source failure/unmount, concurrent PI6/
MM rollback, 32K/64K cache regression and matched net/peak RAM
remain OPEN. PURE remains OFF by default; strict 32K HOLD.
DAIMOS report `design/kcore-p1e-bcache-pressure-restoration-v1637.md`.

## 2026-11-16 v1636: physically recovered six-child 48K capacity

BCACHE now allocates its unchanged owner6, 0201-word private
clean slab using low instead of high MM preference. A general
small-cache placement fix leaves complete aligned high user APRs
free and retains identical source-key/cache invalidation/reclaim
behavior. An independent source-built 48K PDP-6 TSFS/no-AUX/PURE
guest admits SIX child RUNs, completes parent no-growth BRK,
all original child PIPE/private sentinels/EXIT-057 plus six exact
parent PID/EXITED/status WAITs, terminal `LwwwwwwZ`, HALT.

Original live B/A/Z MM owner snapshots prove the parent moved
from former 126000..136000 to `130000..140000` octal, BCACHE
resides low and six complete PROCESS owners are present at B/A,
none remain after WAIT. Source-linked physical audit and forged
parent/success mutants PASS/reject; native BCACHE reclaim/I-O
and multisource tests PASS. No child was PACKED in this fitting
six-child workload, so subsequent seventh-RUN RAW pressure/
restoration and source/PI6/FULL, 32K/64K, physical net/peak
RAM gates remain OPEN. PURE OFF by default, strict 32K HOLD.
See DAIMOS `design/kcore-p1e-bcache-low-physical-v1636.md`.

## 2026-11-16 v1635: physical BCACHE high slab displaces first parent

The original v1629 48K PDP-6 TSFS/no-AUX/PURE first-VM
breakpoint stops at `vm_space_create` PC 022472 for parent slot1.
At that exact moment the source-authentic MM table has the
un-pinned KERNEL_DYNAMIC owner6 `137577..140000`, the 0201-word
private BCACHE tag+data slab. BCACHE source requests MM_ALLOC_HIGH.
The parent's 007067-word VM needs 010000 APR-aligned words,
forcing highest legal base 126000 while that slab occupies the
physical end 140000; later MM snapshot confirms parent at 126000.
The same arena with no high slab could instead place the parent
at 130000. This identifies physical placement rather than a
necessary 389-word PURE MRES reduction as a possible remedy
for the fixed-parent admission regression. Two forged physical
cache owner/span controls rejected by testkit v1635. No production
change is made here: validate any general-purpose low placement
on complete original 48K/64K/32K process and cache workloads.
PURE default OFF; strict 32K optimization HOLD. DAIMOS report:
`design/kcore-p1e-bcache-parent-placement-v1635.md`.

## 2026-11-16 v1634: original PURE MRES compiler-only size test

The original PDP-6 GCC 3.2 -Os PURE MRES can be reproduced
byte-for-byte in testkit using the original kernel map, `p10fold`
and assembler `das -F -C -O`. It is 003336 octal/1758 decimal
resident words. The initial separate diagnostic missed `-F` and
reported a spurious +4-word result; this was corrected in the
test script, with no production bug. Seven alternative compiler
settings were measured: O1/O2 made the image larger and the
best `-Os -fno-strength-reduce` saved only ONE word. None reaches
the required >=389 decimal word low-core footprint reduction
under the v1633 fixed parent APR geometry. No production flags
or code were changed. Next consider validated source helper
consolidation, with all restoration/rollback behavior preserved.
PURE OFF by default, strict 32K HOLD. DAIMOS report:
`design/kcore-p1e-mres-compiler-size-v1634.md`.

## 2026-11-16 v1633: physical APR corridor threshold under PURE

The original source-linked 48K MM table has a PURE-enabled low-
core KERNEL_DYNAMIC high of 046605 octal, vs 043247 with the
same guest and KCORE but no PURE module. The gap 003336 octal
is exactly 1758 decimal resident PURE MRES words. The parent
remains a full APR at 126000..136000 with a 002000 upper tail.
Six other simultaneously runnable full 010000-word APRs require
the dynamic boundary at or below 046000. The installed PURE
low-core boundary is 000605 octal/389 decimal words too high,
even if presently PACKED suffixes could be moved safely.
Original MM/source auditor and two forged boundary/span tests
PASS/reject in daimos-testkit v1633. A module-size improvement
is only a hypothesis pending complete physical guest validation;
no MM owner or process allocation workaround was introduced.
PURE OFF by default; strict 32K HOLD. See DAIMOS
`design/kcore-p1e-apr-corridor-bound-v1633.md`.

## 2026-11-16 v1632: physical equal-five-owner RAM comparison

Source-identical 48K PDP-6 original INIT DXR and KCORE map are
compared with five live children and parent in both modes. ON has
five unpinned PACKED suffixes, occupied/free MM=057023/016173
octal; OFF has five full PROCESS owners, occupied/free
=064224/014330. The physical sampled free-memory difference ON
minus OFF = +001643 octal / 931 decimal words, accounting for the
installed PURE MRES as a 003336 octal arena-start difference.
The largest aligned free gap is ON 007000, OFF 012000; a sixth
RUN needs 010000 and physically succeeds only in the OFF guest.
Prior history differs (ON after a refused sixth RUN, OFF before
the sixth), so these are sampled, not complete physical peaks.
The source-linked MM audit and fake-owner mutants PASS/reject.
No production changes. Design report in DAIMOS:
`design/kcore-p1e-equal-five-memory-v1632.md`. PURE remains OFF
by default; strict 32K size optimization HOLD.

## 2026-11-16 v1631: PURE increases aggregate free but reduces admission

In an original physical PDP-6 48K no-AUX TSFS experiment the same
byte-identical C89 INIT DXR and KCORE map were booted with the
optional PURE MRES on versus off. With PURE on, five children
retain PACKED mutable suffixes and the sixth RUN cannot find a
010000 contiguous aligned APR. With PURE off, the SAME source
prints `aaaaa!uwwwwww`: sixth RUN unexpectedly succeeds, parent
exits via the test's `!u` assertion, six children subsequently
complete their original `w` markers and simulator HALT.

The physical OFF MM snapshot at `!u` shows seven unpinned full
PROCESS APRs, owner slots 7..1 occupying consecutive 010000-word
regions `046000..136000`. There is no PACKED owner; the arena is
037224..140000, occupied/free 074634/003720 octal. The source-
matched ON five-PACKED case has arena 042562..140000, free 016173
but the largest aligned hole only 007000, too small for another
complete 010000 APR. The arena start delta is precisely 003336
octal, matching the 1758-decimal-word installed PURE MRES image.

Thus RAW PURE's individual 001000-word prefix holes fail to form
useful contiguous physical VM space, and for this 48K guest the
optional module worsens RUN admission capacity. This is a real
feature-performance limitation, NOT demonstrated compiler/MM
corruption. The compared snapshots have different child counts;
do not claim equal-load net or peak savings until measured.
Default PURE OFF, strict 32K optimization HOLD. See DAIMOS
`design/kcore-p1e-matched-capacity-regression-v1631.md`.

## 2026-11-16 v1630: physical slot-2 in-place PURE claim and commit

The original v1629 five-PACKED-child 48K PDP-6 TSFS/no-AUX guest
was replayed with debugger breakpoints at original installed PURE
MRES code, source-matched to the MRES map and DXR instruction bytes.
The first restored slot 2 enters v1384 MM CLAIM at PC 034610 with
AC1..4 = 117000, 2, 001000, 010000 (octal suffix/owner/prefix/
complete APR). It then reaches genuine v1384 MM COMMIT PC 034614
with AC1..4 = 116000, 2, 001000, 010000 (original APR/owner/
prefix/complete APR). No FULL v1391 call preceded commit. After
the debugger resumes, all five original child PIPE/data checks
and EXIT 057, five parent WAIT statuses, parent Z and simulator
HALT succeed. Source-backed physical auditor rejects forged AC1
and false-FULL-PC witnesses; original v1395/v1397 native rollback
and v1612 service tests PASS. Thus the FIRST restored child's
fast in-place path is accepted; full-APR fallback and failure
atomicity still require independent physical acceptance. No
production code or MRES layout change, PURE default OFF.
See DAIMOS `design/kcore-p1e-first-inplace-restore-v1630.md`.

## 2026-11-16 v1629: real 48K PACKED to user execution/WAIT accepted

The new actual 48K PDP-6 TSFS/no-AUX PURE guest keeps the parent
alive after a sixth RUN's expected VM allocation refusal. Its
first five sleeping children have source-authentic PIPE input,
private mutable sentinels and EXIT status 057. The parent uses a
no-growth BRK and preserves all five exact PID/EXITED/status-057
WAIT checks. Full physical console:
`aaaaax[B013060/042502]<010000+010000>[A013060/042502]{013060-013060}LwwwwwZ`
followed by normal HALT. The `x` is expected sixth RUN refusal,
and L is lifecycle-only, not a RAM-size savings claim.

Same compiled guest replayed with actual source-linked MM at B/A:
022 octal extents and five distinct unpinned PACKED suffixes,
each 007000 words (owners 6..2), occupied/free 057023/016173.
At parent Z after five WAITs there are 010 extents, no PACKED
and no child PROCESS owners, occupied/free 011536/063460. The
physical original owner audit and false-owner/WAIT mutations
PASS/reject. This is real PACKED -> restored user code -> EXIT/WAIT
and physical MM cleanup, not a 64K resident-only test.

Five omitted 001000-word prefixes total 2560 decimal gross words.
Installed GCC PURE MRES costs 1758 words; positive whole-system
net/peak saving is NOT yet established. Exact fast/full restore
and concurrent immutable source/failure rollback remain OPEN.
Default PURE OFF, strict 32K optimization HOLD. DAIMOS report:
`design/kcore-p1e-48k-packed-lifecycle-v1629.md`.

## 2026-11-16 v1628: no RAW omissions during accepted 64K lifecycle

The same genuine v1627 full-guest 64K source/DXR was replayed
under SIMH, taking original MM ownership snapshots at parent B,
parent A and final Z. The MM arena stays `042602..200000` octal;
extents B/A/Z number 025/025/010, with occupied/free respectively
`074654/040322`, `104654/030322`, `021556/113420`. All three
stages have **zero physical PACKED process extents**. The full
six-child WAIT/EXIT output remains accepted as an ordinary resident
process lifecycle, not a proof of RAW PURE packing/restoration or
net/peak savings. Testkit validates all physical descriptors and
rejects an overlapping extent and a missing final snapshot.

To accept real RAW restore under a live parent, use the previously
observed 48K five-PACKED-child resource-pressure case. Keep the
parent alive after the sixth RUN's expected VM creation refusal,
then prove every child restore/WAIT/EXIT and physical MM cleanup.
Default PURE OFF, 32K optimization HOLD. See DAIMOS
`design/kcore-p1e-64k-physical-owner-matrix-v1628.md`.

## 2026-11-16 v1627: six-child 64K physical process lifecycle passes

A separate source-backed physical PDP-6 64K TSFS/no-AUX PURE
guest retains the original child PIPE read, two mutable sentinels
and EXIT status 057, and the parent's six exact WAIT/EXITED/PID/
status-057 checks. The only new parent success marker `L` means
the lifecycle is tested independently of *additional* PURE drops
specifically during its BRK. Genuine console:
`aaaaaa[B073470/042522]<010000+017000>[A103470/042522]{073470-103470}LwwwwwwZ`
followed by normal simulator HALT. Final Z is after all WAIT
statuses and parent sentinels. Source/physical testkit checks and
mutated success/word-count negative controls PASS. The full
functional lifecycle is accepted on this 64K profile, but
physical PACKED MM owner snapshots, concurrent backing loss,
full/relocated restore failures and net/peak RAM remain OPEN.
No production kernel/MM/MRES/ABI code change, PURE OFF by default.
See DAIMOS `design/kcore-p1e-64k-full-lifecycle-v1627.md`.

## 2026-11-16 v1626: physical PURE MM fit and six-child 64K BRK

The actual v1625 48K physical source-matched MM snapshot has a
single arena `042562..140000`, five unpinned `007000`-word PACKED
owners and six separated free gaps totalling `016173` octal words.
The largest `02000`-aligned free run is `007000`; the exact
booted PURE DXR image `001043` + BSS `004001` + user stack/origin
`002020` needs a rounded complete VM of `010000` octal words.
The sixth RUN therefore cannot fit the current 48K physical owner
layout. This is not a compiler error or a source admission failure.
The testkit proves these physical facts and rejects changed owner
and overlapping-extent mutants; it does not claim a new PACKED
relocation/compaction feature or positive net RAM.

An independently rebuilt physical 64K no-AUX TSFS/GCC PURE guest
admits all six children and completes the parent BRK 010000 ->
017000. Process-word counters grow only by the `010000`-word APR
allocation: `B073470/042522` to `A103470/042522`. The original
additional-BRK-reclaim assertion correctly reports `!R`; the six
child completions follow failed parent cleanup, not full WAIT.
The next independent lifecycle gate must check parent WAIT/EXIT
without confusing *when* PURE omission happened with correct
process termination. Whole-system RAM and 32K remain unaccepted.
See DAIMOS `design/kcore-p1e-mm-geometry-v1626.md`.

## 2026-11-16 v1625: physical RUN VM-creation refusal, not scheduler timeout

Current opt-in PDP-6 RAW PURE has physically restored a child and
executed its first original user-mode PIPE/data checks (v1621),
but the full parent BRK/WAIT/EXIT and net RAM gate remain OPEN.
Source-built 48K TSFS/no-AUX six-child v1624 guest isolates the
sixth refusal as failed RUN (`aaaaa!rwwwww`), before BRK and before
any child sleep-poll check. A separate five-child BRK 017000 run
does not return successfully; neither is a passing pressure case.

The v1625 original PDP-6 RUN error-stage replay breaks at linked
`proc_run_claimed_bad` PC 017725 with AC1 = -2
(`EXEC_LOAD_NOMEM`). This is the genuine `exec_load.s` return from
failed `vm_space_create`, not a later u-area/startup error or full
process-slot table. Source-linked physical/negative auditors PASS.
Exact free-run/MM admission geometry remains to be captured. Do
not change VM or scheduler code based solely on this expected
resource error, and do not claim a positive RAM saving before
counting the installed 1758-word GCC PURE MRES plus scratch.
Default PURE OFF; strict 32K size work remains deferred.
DAIMOS design reports:
`design/kcore-p1e-physical-run-admission-v1624.md` and
`design/kcore-p1e-exec-load-nomem-v1625.md`.

## 2026-11-15 v1605: PURE owner contract for KA10/KI10 portability

P1-E v1604 staged preflight now isolates common stopped-process
TRANSITION/immutable-backing eligibility from the PDP-6-only
sector-rounded omitted PURE prefix calculation. No production
scheduler PURE reclaim/restore is linked yet; v1451 mutable-RAM
compression remains test-only. A KA10 with optional protected
high segment must independently own shared immutable code and
private low mutable state, not drop/reload another process's high
segment. KI10 uses 512-word noncontiguous physical page frames:
the same immutable backing and lifecycle contract applies, but
PDP-6 physical extent and prefix algorithms do not. No
runtime VM vtable is added on PDP-6. See
`KERNEL/MULTI-CPU-PORTABILITY-V1605.MD` and
`DAIMOS/design/kcore-p1e-ka10-portable-boundary-v1604.md`.

## 2026-11-16 v1621: restored child executes user code on PDP-6

The previously source-proven physical 48K TSFS/no-AUX
PDP-6 PURE restoration was continued beyond the v1620
MM commit/pin release, READY_ONCE publication and
selection of child slot 2. After that scheduler write,
the guest printed an isolated lower-case 'w' before
the first of two bounded CPU continuation intervals.

The saved pressure-test C source and exact /SYSTEM/INIT
image named in the TSFS manifest were checked against
the installed executable, which has DXR2 purity=PURE.
There is exactly one user-code 'w' path: the child
returns from its PIPE read, verifies BOTH private
writable-data sentinels and prints the marker, then
requests exit status 057 octal. The marker did not
appear before selecting the restored slot.

This confirms actual successful user-code execution
after PURE prefix restoration. The testkit rejects a
missing signal and a wrong-source mutation and also
rechecks the entire v1620 physical MM/scheduler
write sequence. It does NOT yet prove that the
parent has finished WAIT/EXIT for all children.

Separately, a real 48K four-child physical build
with a smaller, fitting BRK target 016000 octal
succeeded in BRK but did not provide evidence of
RAW physical savings: the logical process-word
count changed 052450 -> 060450, exactly the
006000-octal parent BRK growth from 010000
to 016000. The original test correctly emitted
!R and refused to credit any net mapped-process
reduction. Four child completion markers followed
failure cleanup, NOT a successful parent WAIT.
The dedicated testkit negative-control auditor
rejects an unsafe altered memory-growth word.

A second actual 48K/no-AUX guest requested
six children with the same target. Five
children reached the original ready/sleep
check, but the sixth produced !c before
BRK was reached. The source uses !c both
for an invalid RUN pid and for a child
that fails to enter SLEEP within bounded
polling. The available console cannot
classify which occurred; do not attribute
it to a compiler or memory allocator
defect. The source-linked negative-control
auditor accepts this early refusal and
rejects a mutated refusal marker.

At the original restore boundary, three remaining
omitted 001000-word prefixes total 1536 decimal
gross words, below the installed 1758-decimal-word
GCC PURE MRES footprint. Even four simultaneously
packed prefixes offer only 2048 - 1758 = 290
decimal gross words before other resident costs;
this is not a measured positive physical net.

Full PIPE/WAIT/EXIT, immutable source lifetime,
truly pressure-inducing but fitting BRK, and
48K/64K exact peak/net physical RAM acceptance
remain OPEN. No permanent KCORE, removable
module, MM, driver or user ABI was changed.
PURE stays OFF by default.

DAIMOS source:
design/kcore-p1e-physical-user-resume-v1621.md.

## 2026-11-16 v1620: real PURE MM commit, unpin, READY_ONCE and selection

P1-E's experimental RAW PURE restoration was followed through
the original code in a full 48K PDP-6 TSFS/no-AUX guest. After
the v1619 Type-551 RBN/DCT transfer, the original VFS prefix
loader returns successfully and the CPU enters
mm_pure_commit_prefix_v1384 at 034614 octal. The matching
rollback entry at 034612 was not the selected path.

The real commit arguments are owner slot 2, original APR
base 116000, immutable prefix length 001000 and full
process length 010000 octal words. The current executing
slot is zero. Source-linked v1384 MM descriptor word
024307 then changes in physical memory:

    000014000002   PACKED_PROCESS, pin 1, owner 2
    000011000002   PROCESS,        pin 1, owner 2
    000001000002   PROCESS,        pin 0, owner 2

Both writes are observed by native PDP-6 memory write
watchpoints, not inferred from a function entry or C
unit test. The physical 116000..126000 APR remains
in place with no second mutable-suffix copy.

The scheduler then makes two separately verified
physical writes:

    PC 032477  proc_sched_cursor (024235) = 001002
               READY_ONCE | restored owner slot 2
    PC 016770  proc_current_slot (024233) = 000002
               slot 2 selected for context restoration

At both stops, slot 2 is still PROCESS, pin zero.
The original READY_ONCE path therefore works with the
new PURE restoration mode and fixed KCORE without
additional resident metadata. This is stronger
physical evidence than merely entering the v1384
DROP/restore code or finding an RBN block.

The testkit v1620 auditor checks the exact PURE MRES
linked map/DXR bytes and physical MM/process context.
It rejects executed wrong-owner, missing-pin,
retained-pin and incorrect scheduler cursor/current
mutations. No production KCORE, MRES, MM, driver or
boot-selection code was changed.

The native watchpoints stop before full user-context
return, so successful PIPE/WAIT/EXIT, stable executable
source lifetime and physical peak/net RAM benefit
remain OPEN. At this snapshot the three other omitted
prefixes reclaim 1536 decimal gross words compared
with 1758 resident words for the installed GCC PURE
MRES; a positive net RAM gain is NOT established.
P1-E remains disabled by default.

Canonical DAIMOS report:
design/kcore-p1e-physical-prefix-commit-v1620.md.

## 2026-11-16 v1619: genuine Type-551 RBN and DCT block read

The 48K PDP-6 TSFS/no-AUX PURE pressure guest
had been observed searching backward for
physical DECtape block 000004. Two source-
authentic direct controller tests now use a
private read-only copy of its actual root
DTC0 tape. Both a near block-4 read and
a long seek from block 0341 followed by
reverse read of block 4 PASS in the
original Type-551 and Type-136 drivers.
The 0200-word destination contains data,
and the test media remain byte-identical.

The same real full PURE guest now proves
that its own PI3 handler eventually finds
the requested block. A physical breakpoint
at PC 031046 catches
dtc_search_pi_found with unit 0,
reverse direction, target 000004 and
last RBN mark 000004. The next breakpoint
at 031106 catches the original PI3 read
setup, and PC 030555 catches the
Type-136 DCT reverse-read completion.
The latter has storage_state=-1 and
storage_count=0200 octal, with the
completion event still unposted.

The original DECtape read completes
on the DCT path, not on the general
PI5 tape_pi_done label. An initial
PI5-only test was therefore corrected;
there was no production driver bug.
Temporary extra assembler symbol exports
were shown to leave every word of the
linked production tape MRES unchanged.
The testkit audits these physical
registers and rejects deliberate
mutations of both the matched RBN
and data-completion state.

This closes the question of whether
the target block can be found and
transferred. It does NOT prove VFS
prefix reload completion, the MM
PACKED-to-PROCESS commit/rollback,
pin release, slot-0 READY_ONCE,
PIPE/WAIT/EXIT or positive net RAM.
The next physical breakpoints are
the original PURE MRES
mm_pure_release_prefix_v1384 at
034612 and mm_pure_commit_prefix_v1384
at 034614 octal. No permanent
kernel, driver, MRES or process
storage code changed in v1619.
PURE remains disabled by default.

Canonical DAIMOS design record:
design/kcore-p1e-physical-dct-read-v1619.md.

## 2026-11-16 v1618: 120-second Type-551 search/reversal timeout

The original Type-551 DECtape search timeout was
only 45 seconds (05214 octal ticks at 60 Hz).
A real rewind or direction reversal can take
as long as a minute, so the original deadline
could reject a healthy physical transport.

v1618 increases the search/reversal timeout to
**120 seconds** (016040 octal ticks), allowing
a full minute plus another full minute of
headroom. This is a per-search timeout aged by
PI6; successful RBN matching and data transfer
are not delayed by the larger failure bound.
The PI3 block-mark handler still restarts
the timeout on search progress. The Type-516
MTC rewind/spacing bound remains 15 minutes,
and both DTC/MTC data watchdogs are unchanged.

Compiling the historical v1617 and new v1618
Type-551 driver against one KCORE map changes
exactly two linked MRES instruction words:
the original search command and PI3 mark rearm
at image offsets 000343 and 000405. Native
image/BSS/relocation word counts remain
000607/000017/000013 (octal). Fixed 48K GCC
KCORE DXR remains byte-identical to v1617,
low end 024254 octal; no additional process
fields or permanent memory allocation.

The completely rebuilt v1618 48K no-AUXST1
PDP-6 TSFS guest confirms three real physical
Type-551 search samples:

    clk_tick_count: 015435 -> 015441 -> 015446
    dtc_last_mark:  0341   -> 0133   -> 0126
    storage_count:  777777761740 at all samples

All values are octal. The 60-Hz clock advanced
4 and 5 ticks, block marks changed and PI3
rearmed the new negative 120-second timeout.
The testkit also rejects the obsolete compiled
45-second module and a mutation of the live
physical countdown. Historical v1617 evidence
remains tied to its original 45-second DXR.

The measurement confirms that the search timer
and controller progress remain active, but
does not demonstrate successful RBN/data
completion, PURE prefix commit or rollback,
slot-0 READY_ONCE, PIPE/WAIT/EXIT or positive
net physical RAM savings. P1-E is still
disabled by default. See DAIMOS
design/kcore-p1e-dtc-timeout-v1618.md.

## 2026-11-16 v1617: physical DECtape search during PURE restore

The physical PDP-6 module origin of the previously
interrupted PC 030762 has now been resolved.
Actual scheduler trampoline words show PURE
MRES installed at base 032027 octal: original
reclaim and service offsets 000210 and 000346
lead to installed targets 032237 and 032375.
Therefore PC 030762 is outside the PURE module.

The unmodified linked TAPE MRES image contains
SKIPN 0,storage_count at offset 000331,
matching the same exact physical opcode at
030762. It gives tape MRES base 030431.
The original tape_io.s source places this
instruction in dtc_search_wait, after the
Type-551 controller status checks and before
block-search timeout or retry handling.

A second real 48K no-AUX physical PDP-6
breakpoint at PC 030762 stopped with
storage_state=0200 and storage_count equal
signed -05214 octal ticks: the exact
initialized 45-second RBN search timeout
at 60 Hz. The idle current slot was zero.
Thus restoration was waiting for a real
DECtape search. A single sample does
not prove either a deadlock or completed
diskless prefix reload.

The accompanying physical MM table showed
three 007000-word, unpinned PACKED suffixes
(slots 5, 4, 3) and a full 010000-word
PACKED prefix claim with one pin (slot 2).
The latter is a legal intermediate stage
while immutable text is loaded from VFS,
not proof of a permanent pin leak.

All measurements are source-map-backed and
cross-checked against the original tape
DXR and guest memory; the testkit rejects
mutations shifting the PURE service export
or the original initialized tape countdown.
No production KCORE, module, driver,
MM or process format has changed.

The next gate is to observe line-clock
aging and Type-551 PI3 mark/data progress
until the prefix read completes or returns
a bounded error. This must precede any
claim of successful slot-0 RUN/WAIT,
net/peak RAM savings or a driver bug.
The experimental PURE MRES remains OFF
by default. Full findings in DAIMOS:
design/kcore-p1e-physical-dtc-wait-v1617.md.

## 2026-11-16 v1616: genuine physical RAW PACKED owners

The experimental 48K PDP-6 TSFS no-AUX guest now
demonstrates actual RAW PACKED MM extents in physical
core after BRK pressure. Source-linked DROP entry
evidence alone had not established this in v1614.

An executable CPU breakpoint at the exact linked
vm_space_brk_current address 022104 stopped the real
initial BRK(0) query. A less intrusive SIMH console
EXPECT trigger then stopped directly before the
020000 BRK with five resident PROCESS owners and
zero PACKED. The parent image was 016000 octal
words; each of four sleeping children was 010000.
The largest APR-aligned physical gap was 011577,
insufficient to allocate a second 020000-word image
without reclamation or compaction.

The source-DXR-checked, deferred BRK return breakpoint
020761 was not reached before the second guest was
interrupted at its controlled timeout. At that later
instant the MM table showed child 5 as PACKED at
067000..076000, 007000 words, pin count zero;
child 4 was PACKED at 076000..106000, 010000 words,
pin count one. There were no resident PROCESS
extents and slot zero was active. These are actual
physical allocations; they are not a reading of
logical mapped-process MEMINFO.

The full-size pinned PACKED representation is
intentional in original v1384 restoration: while
the immutable prefix is being reloaded from VFS,
MM claims the expanded full image but keeps its
PACKED type and one pin until commit or rollback.
The interrupted snapshot matches that intermediate
shape. It does not prove either a successful restore
or a persistent MM pin leak.

The v1616 testkit replay wrappers and read-only
audit verify the exact before/after owner geometry
against the matching physical KCORE map. The
snapshots came from separate executions of the
same preserved image; the late one was stopped
asynchronously. They are not a complete atomic
transaction trace. BRK return/AC2, restored RUN,
PIPE/WAIT/EXIT, executable-source lifetime,
peak RAM and positive net savings remain open.

DAIMOS design and physical evidence:
design/kcore-p1e-physical-packed-v1616.md.
No production KCORE/MRES changes. PURE default OFF.

## 2026-11-16 v1615: physical BRK geometry and OOM observability

A controlled 48K PDP-6 no-AUX PURE guest BRK to
`014000` halted before cleanup. The real MM table
contained five PROCESS owners, no PACKED owners,
and an APR-aligned adjacent lower gap of `02000`
words. That BRK succeeded without RAW reclaim.

The stepped guest then requested `014000`, `016000`
and `020000`. The parent disappeared, four sleeping
pipe readers printed `wwww`, and the eventual
simulator halt occurred after process cleanup.
The final MM table contained no process owners;
it is not a live failure snapshot.

The existing BRK dispatcher invokes fatal process
close on `MM_ERR_NOMEM` instead of returning to
user mode. Therefore the test-only BRK cause veneer
cannot print AC2 on that path. Fatal OOM is
consistent with the physical trace, but AC2 has
not yet been independently sampled at failure.
The next diagnostic must stop at the fatal branch
before cleanup to capture AC2 and the exact MM
owner state. No production ABI or KCORE/MRES code
changed, and PURE remains default OFF.

See DAIMOS `design/kcore-p1e-brk-geometry-v1615.md`.

## 2026-11-16 v1614: fix host/native DLINK PURE address proof

The first physical pressure experiments uncovered a
linker defect and an invalid test setup. The old
pressure test's DXR had `purity=UNKNOWN`, because
the test did not link with `dlink --pure`; such an
image is ineligible for P1-E immutable-prefix
reclamation. In the actual host linker, direct
accumulator stores such as `POP 17,10` were
misidentified as TEXT writes, while direct writes
to the last relocated TEXT words could evade the
check: the predicate compared absolute EA against
TEXT length rather than the absolute TEXT interval.
The native linker also needed the AC0..17 exception.

Both source linkers now reject definite direct
TEXT writes using the absolute linked TEXT range,
while recognizing the accumulator register bank.
Strict C89 source-built host tests and native
PDP-6 execution tests PASS, including relocated
targets and an unsafe native mutant.

The corrected physical 48K no-AUX TSFS guest has
DXR `purity=PURE`. SIMH breakpoint evidence shows
two calls into production reclaim and four entries
each into v1381 RAW DROP and MM rebase. The BRK
allocation still fails, however: entering these
routines does **not** prove successful owner
publication, slot-0 restoration or net RAM saved.
The next step is to diagnose exact physical extent
geometry and allocator refusal before modifying
production MM. No kernel or module memory growth.
P1-E remains disabled by default.

See DAIMOS
`design/kcore-p1e-linker-pure-address-v1614.md`.

## 2026-11-16 v1613: physical 64K no-AUXST1 PURE boot

The source-selected opt-in PDP-6 PURE service now installs
in a genuinely diskless dedicated TSFS boot. The KINIT
selection previously returned immediately when AUXST1
discovery returned -1, so the earlier zero-BACKSTORE
branch could never handle absent media. v1613 selects
PURE whenever there is no positively validated
BACKSTORE: absent descriptor (-1) or a valid descriptor
with zero backing blocks. A validated nonzero BACKSTORE
continues to select the original DISK swap module.
The default `DAIMOS_PURE_MRES_ENABLED=0` is unchanged.

The actual KINIT source selection is tested under C89
for absent, zero and positive AUX capacity, both boot
families and the disabled flag. A mutant restoring the
old early return is rejected by the executed test.
KINIT now prints `PURE LOADED` only after the new
module is installed and its two original scheduler
trampolines are patched. This is transient boot
diagnostics, not resident kernel memory.

The first full 64K PDP-6 SIMH TSFS root with all DSK/DRM
media **unattached** has successfully booted to
`DTC OK`, `PURE LOADED`, and `INIT V1`, remaining
alive until controlled test termination. The test
is reproducible in daimos-testkit
`tests/validation/pure-boot-v1613/run-v1613.sh`.
GCC fixed KCORE is byte-identical to v1612,
`__kcore_low_end=024254` octal. The installed PURE
MRES remains 1758 words with no BSS.

This verifies physical boot/install, **not** a successful
RAW pressure, PI6 restoration or positive net RAM gain.
Two test-only adaptations of the historical whole-image
DISK pressure guest could not allocate their third
oversized process; they failed before actual RAW
reclaim/wakeup validation. Their octal MEMINFO
snapshots cannot measure exact physical RAM savings:
`MEMINFO.process_words` omits retained PACKED raw
suffixes whenever the process's APR base is zero.
The failing experiment remains isolated under
`daimos-testkit/tests/diagnostics/pure-pressure-v1613`,
not in the required passing suite.

Next is a smaller BRK/MM allocation after sleeping
owners are established, with exact physical
MM extent accounting, real slot-0 RUN/WAIT/EXEC
acceptance and immutable VFS-source lifetime.
PURE remains OFF by default. DAIMOS implementation
and results: `design/kcore-p1e-noaux-boot-v1613.md`.

## 2026-11-16 v1612: measured smaller experimental PURE module

The separate opt-in PDP-6 PURE MRES has been reduced from
**2275 to 2196 KCC words** (79 words saved, 3.47%),
and **1864 to 1758 GCC lowmem words** (106 saved, 5.69%).
The v1612 CPU-specific service removes one duplicate
MM extent traversal from the retained-PACKED failure
path, while still enforcing exactly one nonzero physical
owner of correct type, word count and pin state.
Wrong/duplicate/zero-base/pinned owners cannot release
TRANSITION or publish READY_ONCE.

The existing `p10fold` identifies a 49-word relocation-
safe zero-cost equivalence in the original v1381 DROP
and v1391 FULL transaction objects. The generated v1612
fold plan merges that shared code without an extra JRST
and without changing fixed KCORE, swap formats or
the historical disk service. The service policy itself
also saves 30 KCC words (412 -> 382).

A source-level common helper was tested but resulted in
**2356** total MRES words because of compiler-generated
call overhead. The experiment was discarded after
measuring the complete linked MRES, not because the
common-preflight idea was invalid. No extra process
table, CPU vtable, RAM registry or helper survives.

The v1612 native PDP-6 tests execute actual DROP,
fast-prefix and bounded FULL restoration and refused
backing/source/loader cases, including duplicate MM
owners, pinned suffix, and unsafe READY_ONCE mutation.
The updated OFF -> ON -> OFF KINIT test requires the
v1612 package, 2196-word folded size and fixed KCORE
binary identity. `__kcore_low_end` remains octal
`024534` (GCC `024254`), with no additional BSS or
per-process words. The fixed GCC lowmem KCORE also
remains byte-identical to its v1611 image.

P1-E remains **experimental and default OFF**. No
complete physical no-AUXST1 PDP-6 guest, real PI6
schedule/EXEC/WAIT/EXIT lifecycle or successful
net/peak physical RAM acceptance has yet been
demonstrated. Compression remains deferred. The
source/measurement record is DAIMOS
`design/kcore-p1e-pure-mres-minimize-v1612.md`.

## 2026-11-16 v1611: experimental no-BACKSTORE PURE MRES (default OFF)

DAIMOS now builds a separate PDP-6 swapless PURE MRES containing
the real v1381 zero-copy inactive RAM DROP, v1384 same-base prefix
restore, v1391 capacity-bounded FULL fallback, its two physical
MM primitives and the shared v1386 executable image reader.
KINIT can install this MRES on a dedicated TSFS root with **no**
AUXST1 backing, or the existing DISK SWAP MRES when AUXST1
does exist. Both install through the existing two resident
scheduler trampolines, with no new persistent registry or
process storage mode.

**Production activation is deliberately disabled:** the
compile-time switch `DAIMOS_PURE_MRES_ENABLED` defaults to
zero. It requires explicit value 1 to link and offer the
new module. All source-linked native PDP-6 DROP/slot-0
prefix/FULL, source refusal and retry tests PASS, and
both opt-in KCC KINIT and PURE MRES link successfully.
The default KINIT omits PURE entirely; default KCORE DXR
is byte-identical to v1610 with low end `024534` octal.

The experimental PURE MRES costs **2275 KCC or 1864 GCC
words**, BSS zero, no extra per-process storage. This
must be subtracted from gross PURE prefixes reclaimed.
The preliminary strict 16K single-root placement report,
which sums the mutually exclusive DISK and PURE packages,
FAILS for experimental KCC full
(`PERMANENT_LAST=055660` octal, 7089 words over)
and GCC lowmem (`046702`, 3523 over). The matched
default KCC full is itself 4814 words over. Thus no
physical installed-runtime fit or net RAM benefit is
established by these aggregate link reports.

The independent KINIT Makefile stack-watermark mode stamp
definition-order issue was corrected, and the historical
TSFS swapless MINIT v490 GCC test passes again. Full
no-AUX hardware/simulator physical-guest acceptance,
immutable executable source lifetime, peak allocator
memory, EXEC/EXIT/WAIT/BRK and owner-failure tests
remain open. Compression and other CPU backends are
not enabled.

Source, size and acceptance detail:
`design/kcore-p1e-swapless-service-v1611.md` in DAIMOS.

## 2026-11-16 v1610: authoritative slot-0 DISK/PACKED selection

Production PDP-6 `proc_swap_mres_service_v481` now checks
the original two-bit per-slot storage map before calling
the disk swap-in transaction. DISK retains the original
I/O, pressure, fatal error and READY_ONCE handling.
PACKED **cannot** be passed to DISK: the service instead
keeps the inactive raw MM suffix and executable backing
record intact, raises the existing 0100-octal cooldown
and clears the pending request. It issues neither a disk
read/free nor TERM or READY_ONCE. Invalid modes refuse.
This is a safe interim service state; it does NOT restore
RAW memory or authorize an un-restored APR for execution.
No production path creates PACKED at v1610.

The staged PURE common lifecycle preflight now has three
explicit actions: DROP (0), TERMINAL (1), RESTORE (2).
RUN is admissible only for RESTORE when the processor is
executing in slot 0 with the target TRANSITION-held; DROP
remains restricted to SLEEP/STOP. Both the v1384 fast
prefix and v1391 bounded FULL source-linked transactions
use this common check, followed by their existing PACKED
mode/executable/MM identity verification.

Real PDP-6 compiled MRES mode test, unsafe DISK-routing
mutation, slot-0 RUN prefix/FULL tests, and historical
v1316/v1317 disk-swap/rollback tests PASS. The full KCC
ROOT=tape KCORE and SWAP MRES package build/link PASS.
Permanent KCORE DXR/map remain byte-identical to v1609,
`__kcore_low_end = 024534` octal. The mode selection
costs 11 instructions in the removable SWAP MRES only.

The next work remains a correctly retained immutable
executable, a small no-BACKSTORE PURE service, live
zero-copy inactive DROP and complete prefix/FULL slot-0
wakeup with PI6/EXEC/WAIT/failure tests. Compression
is still deferred. DAIMOS source/evidence:
`design/kcore-p1e-slot0-mode-v1610.md`.

## 2026-11-16 v1609: production terminal PACKED/RAW owner cleanup

KCORE now has a correctly owned PDP-6 EXIT/VM destroy path
for a future inactive RAW PURE process, before enabling real
reclamation. `vm_space_destroy` admits storage mode PACKED
only with zero APR base; `proc_swap_detach` dispatches the
new PDP-6-only `pdp6_pure_detach_v1609` physical MM leaf
when mode 2 is authoritative. It checks TRANSITION, stable
u-area, saved executable identity, original PURE geometry
and exactly one unpinned MM_TYPE_PACKED_PROCESS extent with
matching process owner and suffix word count. It never
interprets a saved PURE executable record as a DISK run.

On refusal the inactive owner, mode and record survive for
PREVM retry. On success the exact MM extent is freed,
PACKED->RESIDENT is published, the saved backing is cleared
and VM_SPACE_NONE is committed. Terminal release does not
require the executable mount to survive. The existing disk
swap and resident XIP paths retain their previous behavior.
The physical leaf is source-selected via the existing PDP-6
architecture manifest, without new process or RAM state.

The first correct C scan grew the linked KCORE by 169 words.
The accepted compact PDP-6 assembly scan costs 104 words:
`__kcore_low_end` `024364` -> `024534` octal, zero new
DATA/BSS or per-process words. Full KCC ROOT=tape build and
native PDP-6 actual VM destroy / proc_swap_detach / MM leaf
positive and negative tests PASS, including an intentionally
unsafe saved-backing mutant. Historical v880/v957 DISK and
v1129 VM_NONE tests PASS. Staged in-place v1384 restore
now also checks v1608 live executable-source admissibility
before any MM prefix claim; v1384/v1385/v1395 tests PASS.

This does **not** create PACKED processes in production.
P1-E1 real immutable VFS lifetime, E2 slot-0 RAW DROP,
E3 complete prefix/FULL restoration, E4 scheduler/PI6/
RUN/EXEC/EXIT/WAIT and E5 full physical PDP-6 tests
remain OPEN. No RAM codec or paging is enabled.
See DAIMOS `design/kcore-p1e-packed-terminal-v1609.md`.

## 2026-11-15 v1608: staged P1-E1 source admission

The first P1-E implementation increment now adds a non-sleeping
swapless and live executable-mount check to the already staged
v1381 RAW DROP and v1391 bounded FULL RESTORE. It rejects
enabled or still-owned BACKSTORE, missing mount slab/identity,
unsupported provider and mismatched high mount-ID bit before
any MM pin, rebase or destination allocation. Terminal
PACKED teardown remains possible when backing is missing.

Native PDP-6 source-linked v1381, v1384, v1385 and v1395/v1397
and v1604 boundary tests PASS, plus the v1608 refused
source/admission cases and a deliberate bypass mutant rejected
by the testkit. Full KCC PDP-6 ROOT=tape image and map are
byte-identical to the v1606 baseline. No production scheduler
calls the staged transactions; no physical swapless boot,
VFS provider lifetime or PI6 process reclamation is accepted
by this test. P1-E1 production and E2-E5 remain OPEN.
DAIMOS design and exact evidence:
`design/kcore-p1e-source-admission-v1608.md`.

## 2026-11-15 v1607: source-matched KCORE completion checklist

The active task is **production uncompressed P1-E RAW PURE**
and its native/real PDP-6 scheduler, owner, PI6, full APR
restore, OOM, source identity and rollback acceptance.
The v1604 lifecycle/geometry split remains staged only.
Compression is a later measured yes/no decision; generic
segmentation/paging, strict 32K optimization and CPU ports
are not active. See
`STATUS/KCORE-REMAINING-WORK-V1607.MD`.

## 2026-11-15 v1606: PDP-6-first PURE lifecycle and memory mapping

The P1-E staged v1604 implementation now separates common
quiescent-process lifecycle admission from PDP-6-only APR PURE
prefix geometry. PDP-6 remains the ONLY implemented CPU target.
Finish and physically validate its RAW swapless PURE reclamation,
source immutability, full and fast restore, PI-safe publication,
EXEC/EXIT/WAIT, owner rollback and peak RAM before attempting
compression. Production scheduler linkage is not yet accepted.

A noncontiguous/segmented mapping is a future backend concern:
common policy retains writable state and executable ownership,
not a presumed physical APR prefix or single contiguous user
pointer. No future CPU port is planned, and no paging or shared
segment code is part of the current P1-E work.
See `KERNEL/MULTI-CPU-PORTABILITY-V1606.MD`.

## 2026-11-11 v1488: active swapless PURE integration boundary

Production raw disk swap is unchanged. The v1381 in-place PURE drop,
v1384 prefix restore, and v1395/v1397 fault rollback tests PASS on
native PDP-6 in isolation, but NONE of the staged C PURE RAM
transactions nor bounded FULL restore v1391 is scheduler-linked.
There is no proven net whole-system RAM saving from these components.
The v1451 30-word trusted bitmap codec is also test-only, not
activated RAM compaction or production process compression.

The v1487 updated common-DXR EXEC/PURE candidate carries over the
original strict XIP admission path and passes standalone DXR tests.
Same-profile isolated TEXT counts: production EXEC 218, candidate
EXEC 163, shared engine 235: +180 combined TEXT words. This is NOT
a production change or accepted RAM optimization; the copied XIP
branch has not been given a fresh real guest lifecycle test.
Avoid duplicating resident loader parsers to reach the PURE milestone.
Audit existing loader/restore reuse and total process savings first.

Native 48K LOWMEM v1486 verifies ROOT/DSH and five stable
DATE/MEMSTAT samples, not swapless PURE; 32K is explicitly deferred
and must not be run. MEMFS/XIP requires at least 96K. Complete safe
inactive process drop/restore under actual scheduler/PI, fallback,
immutability and APR ownership before enabling optional in-memory
mutable-state compression. Future runtime MRES unload/reload is
POST-1.0 and not part of this integration. Refer to
`STATUS/KCORE-RELEASE-AUDIT-V1488.MD` for current gate inventory.

## 2026-11-10 v1397: fail-closed PURE DROP MM rebase cleanup

The staged `proc_pure_ram_drop_v1381` formerly ignored failure to unpin the
original PROCESS extent after `mm_pure_rebase_pinned_v1381` rejected the
conversion. It could return a less severe error while leaving an unresolved
MM pin and a process still owning TRANSITION. The revised transaction checks
the cleanup result: a failed unpin returns `MM_ERR_BUSY`, preserving
TRANSITION and the original descriptor for explicit recovery. A successful
unpin still returns the original rebase error. The native PDP-6 v1395/v1397
test now injects both cleanup cases, in addition to four existing FULL
restore rollback/finish faults; all PASS. Existing native v1381 in-place and
v1384 prefix tests also PASS. The separate KCC target object changes from
**518 to 521 TEXT words** (+3), with zero DATA/BSS. It is **not production
linked**, so no KCORE increase or proven whole-system RAM gain exists yet.

The grouped D6 codec remains deferred until the outstanding AUXSTORE/
BACKSTORE, restricted MEMFS XIP and swapless PURE RAM feature integration is
finished. Production D6LZ36 and raw disk/drum swapping are unchanged.

# Swapless process-memory reclamation

**v1451 finalized test-only bitmap codec:** the independent PDP-6
RAM-only encoder+decoder now totals **30 TEXT words, zero BSS**,
preserves AC10..AC15 without a stack frame, and packages as a
**36-word transient / 30-word resident MRES**. It deliberately
contains *no untrusted-data validation* because MM alone owns
both source and packed images. Exact preflight, buffer lifetime,
process state/restore and root-profile gating remain separate
work. See KERNEL/D6CODEC-BITMAP-TRUSTED-OPTIMIZATION-V1451.MD.

**2026-11-10 v1432 evidence:** A native 55-word PDP-6
zero-presence-bitmap encoder/decoder compresses real cold
DSH mutable suffixes to 314/524/677 words (from
3200/3200/5248) using 24,658-41,400 encoder instructions.
This is test-only capacity evidence, NOT a production feature:
finish PURE-only reclaim and real MM/32K/48K acceptance first.
Never encode on every SLEEP. The same bitmap **expands
typical nonzero-heavy compiled DXRs by ~2.7%**, so it does
not replace TSFS's general-purpose executable/file codec.
See KERNEL/SWAPLESS-BITMAP-COMPRESSION-V1432.MD.

Status: accepted KCORE direction, 2026-11-09 v1375. **Staged PURE-only
and zero-copy transactions pass through v1382; no production scheduler or full
swapless guest acceptance.** This document supersedes the earlier proposal
for executable-opt-in *compressed disk swap* (old decision 5/P0-D). Raw
disk/drum process swapping is unchanged. Compression of disk-swap records is
not part of the release plan.
The old P0-D *feature-list* label must not be confused with DAIMOS's
separate v1298 **P0-D OOM fault/regression matrix**, which stays accepted.

## v1391/v1392: APR-bounded FULL DXR restoration and KCC correctness

**2026-11-10. Staged host/native test acceptance, NOT a production
KCORE or disk-swap change.**

The previously staged FULL DXR callback accepted a backing record,
destination APR base and stable process u-area, but did not receive
the extent's *allocated length*. Validating only the executable
text count was insufficient: a DXR file with the same PURE text
but an oversized initialized image could write past the allocated
APR. This also affected compressed FULL decode.

`proc_pure_ram_restore_v1391.[ch]` now supplies a four-word C ABI
callback argument list: saved backing identity, pinned physical
destination APR base, stable u-area and **trusted allocated APR
word count**. The shared assembler
`exec_image_full_bounded_v1391` first obtains the immutable source
vnode, VFS stat and DXR2/TX2 metadata using the same parser as EXEC
and PURE_PREFIX. Before invoking any payload reader or decompressor,
FULL mode rejects zero/unaligned APR capacity, unaligned APR base,
physical address overflow, or an initialized image where
`image_words + 020 > allocated_APR_words`. An exactly fitting
initialized image is valid. The former unbounded staged FULL entry
is removed, not retained as a compatibility backdoor. The current
production `proc_swap_reload_pure` remains unchanged.

Native KCC/DAS/PDP-6 tests in `daimos-testkit` verify successful
plain/compressed FULL restoration, exactly filled APR, mount 18
and canonical vnode identity, short read, invalid metadata,
compressed decoder failure and rollback. Crucially, both plain
and compressed *oversized* cases observe **zero payload-reader and
decoder invocations**. Independent negative cases cover zero,
misaligned and out-of-range allocations and physical base
misalignment; sentinel words immediately beyond the target APR
remain unchanged. The only valid mutable suffix is copied from
its original owner-pinned raw RAM after the FULL reload succeeds.
These are native component tests with controlled VFS/MM, not
concurrent scheduler/PI or 32K/48K guest acceptance.

### Optimizer integrity and size

Testing uncovered a separate real KCC code-generation defect:
under a larger diagnostic layout, optimized KCC emitted
`MOVEM 7,swap_slots(1)` for the constant slot
`swap_slots[1]`, leaving a dynamic AC1 index without a valid
reaching definition. `kcc/ccopt.c` now embeds a constant in the
symbolic address only after a straight-line backward register
definition scan proves it, stopping at skips, branches, calls and
register modifications. This is a compiler fix, not a DAIMOS
array-access workaround. All 21 focused native/host PURE and
EXEC regressions pass on the corrected installed KCC; its fresh
optimized release build matches the tested binary. The cleaned
v1392 test also passes on the old compiler, so a smaller
independent old-failing repro and full compiler matrix remain open.
See `TOOLCHAIN/KCC-SYMBOLIC-FOLD-20261110-V1392.MD`.

| Isolated PDP-6 source | TEXT words | Static DATA/BSS |
| --- | ---: | ---: |
| Existing production EXEC loader | 183 | 0 |
| Disposable refactored EXEC caller | 119 | 0 |
| Shared DXR inspector/payload core | 161 | 0 |
| Shared capacity-bounded FULL/PREFIX adapter | 74 | 0 |
| Previous production PURE FULL reload leaf | 50 | 0 |
| Separate v1391 FULL restore C transaction | 359 | 0 |

The hypothetical combined EXEC+FULL replacement costs
`119+161+74=354` TEXT words versus `183+50=233`, i.e.
**+121 words before p10fold** (and without counting the new C
transaction). Thus the safety correction is staged but not a
resident-RAM saving. Do not link the v1391 C body alongside the
older restore transaction as a permanent duplicate. Net physical
memory benefit still requires actual folded KCORE measurements and
multiple inactive-process workloads at 32K/48K. Grouped compressed
PURE_PREFIX, PI/VFS ownership concurrency and booted lifecycle
regressions remain open.

## v1389/v1390: smaller shared loader and one FULL backing entry

**Staged and native PDP-6 tested, not linked into production
(2026-11-10).** The v1386 shared DXR inspector and FULL/PURE_PREFIX
payload engine now use contiguous BLT register-save frames rather
than seven individual AC PUSH/POP pairs. This preserves the established
PDP-6 AC10..AC16 calling convention and reduces the standalone
shared assembler object from 225 to **212 TEXT words**, zero static
DATA/BSS. Native v1386 parser, v1387 in-place restoration and the
v1388 disposable EXEC caller tests still PASS.

The `exec_image_full_backing_v1389` entry consists of a zero-prefix
selection and branch into the *same* backing adapter. It uses the
existing `proc_swap_unpack_backing` vnode/mount reconstruction,
VFS stat, validated DXR inspector and FULL payload reader; no
duplicate DXR parsing or decompressor code is added. The same
adapter's `exec_image_prefix_backing_v1386` entry invokes bounded
plain PURE_PREFIX instead. With FULL, the fresh, separate APR
destination receives the entire initialized DXR image and the
restoration transaction subsequently overlays the saved writable
suffix. Thus even compressed FULL DXR uses the original decoder
without overwriting the only authoritative source suffix.

The native PDP-6 `daimos-testkit/tests/validation/
exec-image-full-backing-v1390` test links the **actual staged**
v1381 RAM drop/restore transaction and real backing-unpack assembly
to this common FULL path. Controlled immutable providers exercise
plain and compressed DXR2, mount 1 and mount 18, failed short reads,
invalid TX2 and corrupt compressed input. The source suffix is
retained on failure and overlaid after successful FULL reload.
This PASS does not prove the production disk swap or 32K guest
behavior. The existing `proc_swap_reload_pure` is unchanged.

The final native test sweep also found an **out-of-contract
test-provider write**: the v1387/v1390 mock VFS header reader wrote
three words when the shared inspector requested only two. That
extra write could corrupt adjacent frame scratch and made a mount-18
case appear sensitive to an unrelated, wider compression-mode
predicate. Correcting the fixture to write TX2 only on a
three-word request (while DXR2 inspector reads TX2 separately)
restored reliable PASS in both predicate configurations. This
finding does not demonstrate any KCC, assembler or production VFS
regression; the corrected regression now respects actual VFS read
bounds.

### Size and capacity gate

| Independently assembled PDP-6 source | TEXT words |
| --- | ---: |
| Existing production EXEC loader | 183 |
| Alternate shared-engine EXEC caller | 119 |
| Shared DXR parser and payload engine only | **161** |
| Shared FULL/PREFIX backing adapter | **51** |
| Existing independent PURE FULL reload leaf | 50 |
| **Combined alternate EXEC + FULL replacement, before folding** | **331** |
| **Current EXEC + FULL leaf baseline, before folding** | **233** |

The hypothetical replacement is therefore **+98 words**, before
p10fold and before all new swapless MM/scheduler transaction code.
There are no new static data words, but this does NOT establish net
system RAM savings. In particular, measuring only a 256-word PURE
prefix saved from one suspended process while ignoring the resident
transaction and allocator code would be misleading. Net physical
RAM must be evaluated for real 32K/48K workloads, including all
inactive processes, actual KCORE linked text/BSS, heap/stack, and
pressure/retry costs. The executable v1390 size regression records
these isolated measures and rejects unexpected growth.

**Mandatory safety gate before connecting FULL to live swapping:**
the older three-argument PURE reload callback does not expose the
physical destination's allocated word count to the common DXR
reader. Even with a pinned immutable vnode and matching TX2 text
count, a larger initialized-image length in a malformed/backing
file could overrun the allocated APR extent. A final callback ABI
must carry explicit destination capacity and reject
`initialized_words + EXEC_USER_ORIGIN` greater than that capacity
*before* any read/decompress/write operation. The native tests use
valid controlled images and do not close this hostile-header bound.
The separate compressed PURE_PREFIX output-boundary constraint and
full 32K/48K tape-root integration remain open.

## v1386-v1388: actual shared DXR loader staging and cost gate

**2026-11-10: native tested, deliberately NOT production linked.**

`system/kernel/proc/exec_image_common_v1386.s` is the first executable
format/loading component shared *in executable native tests* between
ordinary EXEC and swapless inactive-PURE restoration. It defines a
single DXR1/DXR2 inspector and a single payload loader. The parsed
metadata occupies exactly four caller-owned temporary words:

| Word | Left 18 bits | Right 18 bits |
| --- | --- | --- |
| 0 | Entry offset | Uncompressed initialized image words |
| 1 | DXR flags | BSS words |
| 2 | Uncompressed PURE text words | DXR header words (2 or 3) |
| 3 | Relocation-map words | Compressed payload words |

The inspector validates the executable magic, entry and image limits,
DXR1/DXR2 file shape, BSS/flag bounds, compressed payload length and
text metadata/TX2 tag before publishing that four-word description.
The loader interprets it in `FULL` mode (DXR1/2 plain, DXR2 compressed
via the existing D6LZ36 decoder), or `PURE_PREFIX` mode (DXR2 PURE,
copy exactly the missing prefix and write **no** mutable word). The
current compressed-prefix case returns unsupported and triggers the
already proven full-image restore fallback. Neither mode allocates
kernel process tables or additional permanent state.

`exec_image_prefix_backing_v1386` is a bridge within the same staged
source, not a new parser: it uses `proc_swap_unpack_backing` to
reconstruct the original vnode and high mount ID, obtains file size
and regular-file type via VFS, checks the parsed TX2 text size
against the saved PURE record, and uses the common prefix loader.
It does not own MM, alter the process's execution state or bypass
existing immutable-vnode/unmount exclusion. Its caller is the
previously staged v1384 MM restore transaction. The original v1385
standalone loader is kept **only in daimos-testkit** as historical
validation evidence; it is not in DAIMOS source.

The native PDP-6 testkit results are:

- v1386: shared parser and image reader, DXR1/DXR2 FULL, compressed
  FULL delegation, STRICT prefix output boundaries, invalid shapes
  and short read rollback: PASS.
- v1387: real v1381 zero-copy PURE drop, v1384 MM claim/rollback/commit
  and the same v1386 shared loader, using the existing genuine backing
  unpacker, including mount 18, provider validation and safe
  compressed-prefix refusal: PASS.
- v1388: generate a throwaway EXEC caller from the current production
  `exec_load.s` with explicit source-anchor checks. Real PDP-6
  execution of this alternate caller together with the v1386 common
  engine passes plain DXR1/DXR2 and compressed FULL, RT root-only
  admission, malformed headers, VM NOMEM, read/decode failure and
  PURE-attachment rollback. The generated caller is **not** the
  production EXEC implementation: PASS.

The native tests caught a caller-clobbered register bug in v1386
header parsing (a DXR2 header count lost across the VFS callback).
The corrected routine preserves the value across VFS. The v1386
negative suite also corrected a test fixture which had failed to
generate the intended malformed file length. These fixes are in
the staged/test code; production source remains unchanged.

### KCORE size gate

| Comparable isolated PDP-6 source | TEXT words |
| --- | ---: |
| Existing production `exec_load.s` | 183 |
| Throwaway v1388 EXEC caller | 119 |
| Shared v1386 inspector + payload core, no bridge | 169 |
| **Combined alternate EXEC baseline** | **288 (+105)** |
| Full v1386 module with restoration bridge | 225 |

All these stand-alone sources have zero static DATA/BSS. The
combined sizes are **pre-fold component measurements**, not a final
KCORE link comparison. Adding the shared parser and loader to the
currently mandatory resident EXEC path would increase text words
before folding despite reducing the caller from 183 to 119. The
extra bridge adds 56 TEXT words relative to the 169-word parser/
payload core. This must be reduced, folded safely, or selected only
for a genuinely swapless configuration with measured net RAM saved
before any production link acceptance. The existing three fixed
resident C translation units and raw disk/drum swapping are unchanged.

**Still open:** actual source-to-production EXEC cutover and booted
32K/48K acceptance, `proc_swap_reload_pure` FULL path consolidation,
grouped D6codec bounded compressed PURE_PREFIX, hardware PI/VFS
concurrency and full MM/scheduler/EXIT/WAIT lifecycle. The v1388
native PASS does not close these conditions or establish a positive
production link-size delta.

## v1384/v1385: prefix restore proof and ONE shared EXEC loader

**Staged and native tested, not production linked (2026-11-10).**

The latest raw-PURE prototype can restore into its *original* physical
APR extent if the released prefix remains wholly free. A new
`mm_pure_expand_v1384.s` three-operation MM interface claims the
original lower prefix, reverses that claim after a read failure or
retypes the fully reconstructed image as PROCESS. Preflight checks
the exact slot owner, PACKED_PROCESS type, suffix length, exactly one
pin, proper APR alignment, physical 18-bit limits, preceding MM
neighbor and one continuous managed physical arena. No data is
copied or allocated by these MM operations, and the descriptor
remains PACKED_PROCESS throughout I/O, so the intermediate image
cannot be scheduled.

The staging transaction `proc_pure_restore_v1384.c` keeps
TRANSITION and executable backing immutability across the operation.
It briefly disables PI for the claim and for the MM/VM/storage-mode
commit, but executes potentially slow filesystem I/O with normal
interrupts. Failed partial reads reverse the MM claim and leave all
original mutable suffix words intact; an occupied or unmanaged gap
returns a distinct result permitting the already tested full-image
restore fallback. A compressed DXR not known to support bounded
prefix decoding also takes that safe fallback. The claimed memory
must not become a runnable PROCESS extent until the complete PURE
prefix has been validated and loaded.

`daimos-testkit/tests/validation/pure-ram-prefix-v1385/exec_prefix_fixture_v1385.s`
verifies the actual native DXR2 plain-prefix operation, including
canonical vnode reconstruction
for mounts 1 and 18, fifth-bit mount ownership, exact TX2 text size,
full-sector prefix bounds, malformed headers, short reads and
compressed-format rejection. It directly reads only
`[APR_base + 020, APR_base + pure_prefix)`; it never writes the
already-owned mutable suffix. These tests PASS in
`daimos-testkit/tests/system/pure-ram-prefix-v1384` and native
`tests/validation/pure-ram-prefix-v1384`/`pure-ram-prefix-v1385`.
An inverted PDP-6 arena comparison was found by native tests and
corrected. No active KCORE linking is altered.

### Loader consolidation (required before production)

The v1385 plain-prefix function exists **only in daimos-testkit**, not
as a second DAIMOS production DXR loader. Existing
`exec_load_process` handles initial
EXEC image construction, while `proc_swap_reload_pure` is used for
full-image PURE reconstruction. A future single DXR image-loader
engine must replace duplicated header parsing and payload handling
and expose an explicit output mode:

| Shared loader mode | Destination contract |
| --- | --- |
| `FULL` | Initialize the complete executable image in a new APR allocation; the EXEC caller owns VM setup and process publication |
| `PURE_PREFIX` | Write only the missing immutable prefix to an already-claimed MM extent, never user ACs or the existing mutable suffix; RESTORE owns MM rollback and process publication |

One DXR parser, one plain payload reader and one accepted grouped
compressed decoder should implement both modes. Do **not** collapse
EXEC's pathname/privilege/runtime admission or RESTORE's existing
process ownership/MM rollback into the low-level image loader.
Those lifecycle operations are distinct and should remain outside
the shared core. The present EXEC plain payload path already makes
a direct VFS read into the allocated image; there is no separate
runtime relocation pass to duplicate. A future grouped compressed
decoder must enforce a hard output boundary, including MATCH tokens
crossing it, or provide independently framed text. Existing
unbounded/full-image compressed decoding is never permitted to
write over the inactive mutable suffix.

### Measurements and remaining limits

| Isolated staged PDP-6 object | Text words | Static DATA/BSS |
| --- | ---: | ---: |
| v1384 prefix transaction, C89-compatible source | 323 | 0 |
| v1384 MM claim/release/commit, assembler | 130 | 0 |
| v1385 plain-prefix testkit fixture, **not production source** | 78 | 0 |

These 531 pre-fold words are **not** production linked-text growth.
The v1385 test-only 78-word fixture must be replaced by the existing
EXEC image-loading engine rather than shipped beside it. Full
linked text and shared-code savings have not been measured.

For a 1024-word process whose first 256 words are reconstructible
PURE, in-place restore holds only the original 768-word suffix plus
the reclaimed 256-word prefix (1024 process words total), instead of
temporarily holding the 768-word suffix plus another complete
1024-word allocation. This assumes the gap remains free and excludes
kernel, device and VFS workspaces. When the gap has been reused,
full-image restore still needs an APR-aligned allocation elsewhere.
Actual scheduler/PI integration, grouped-codec bounded output, the
consolidated loader and a 32K/48K swapless guest remain open.

## v1383: one-word size geometry and safe prefix-only restoration

**Design reconciliation, 2026-11-10. No new production code or test
acceptance.** A PDP-6 word can encode the two uncompressed lengths
needed for swapless PURE restoration:

| Left half (18 bits) | Right half (18 bits) |
| --- | --- |
| Exact immutable text words from the DXR2/TX2 executable | Current retained mutable user-memory suffix words |

These are **uncompressed** counts, not the number of stored compressed
stream words. The correct current suffix count is NOT merely the
original writable DATA section of the executable: runtime BSS,
stack, heap, other writable words and APR padding must be retained
exactly. The low 020-octal user AC words are saved in the stable
u-area and are not copied from the executable during text reload.
For the present full-sector PURE-drop convention:

`released_prefix = (text_words + 020) & ~0177`

`mutable_suffix = current_VM_allocated_words - released_prefix`.

Validate that `released_prefix` is nonzero, less than the current VM
allocation, exactly 0200-word-sector aligned, and the sum is an
APR-legal allocated size. The existing DXR2 header already contains
18-bit *uncompressed initialized image* words and the TX2
uncompressed text count. The process `vm_state` holds its full
current 18-bit VM allocation; the already resident/saved PURE
backing word contains its validated text count. That backing word
currently encodes only **14** text bits alongside vnode and mount
identity. Therefore the two proposed sizes can be reconstructed at
quiescence with **zero extra permanent per-process words** for the
existing supported PURE range. Changing the 14-bit backing limit is
an independent ABI decision; merely defining a larger 18-bit field
elsewhere does not extend it. A packed `text,,suffix` word is an
optional *transient representation*, not automatically an additional
KCORE table or slot.

### In-place RESTORE and compressed executable caveat

The v1381 DROP releases only the lower contiguous
`released_prefix` words, retaining the raw suffix at its original
physical address. A matching in-place RESTORE can reclaim that gap
and reverse the MM descriptor rebase if, and only if, the whole
prefix remains free, the reconstructed full base satisfies APR
alignment, and a transactional MM/PI/PROCESS owner transition is
validated. Reconstruct **only** the missing words at addresses
`[base + 020, base + released_prefix)`, preserving the suffix
bit-for-bit. A plain DXR2 loader can request the shorter count;
neither user ACs nor any mutable DATA may be overwritten.

The current `proc_swap_reload_pure` compressed DXR path is NOT safe
for this operation because it invokes the existing decoder on the
entire initialized image. Furthermore, the trusted-backing
`d6lz36_decode_vfs` ABI can stop when a requested output count is
reached, but the fixed `decompressor.inc` MATCH path subtracts the
whole token length and rejects a match if it runs beyond the
requested prefix end. It cannot simply be passed the smaller
prefix length on every existing compressed DXR. Metadata alone does
not change token boundaries or guarantee partial decoding.

At the atomic grouped-D6codec production cutover, choose either
an explicitly bounded-prefix decoder that safely clips the final
match without touching writable memory, or an independently framed
immutable-text segment guaranteed by the producer. Preserve the
global **no permanent legacy decoder** policy, and test match
crossings, exact lengths, short files, corrupt streams and
restoration rollback against actual native PDP-6 code. A single
36-bit geometry word is adequate for the sizes; stream framing and
data safety require separate proof.

If another MM allocation has occupied the released prefix, size
knowledge cannot make it free. The already staged safe fallback
allocates a complete, aligned APR extent elsewhere, decodes there
and then overlays the exact retained suffix. If memory is
insufficient, the process stays inactive until the scheduler can
retry without violating source ownership; never overwrite the
only valid mutable suffix. This is why in-place RESTORE is a fast
path with a rigorous unavailable-gap fallback, not a guarantee
that any suspended process can resume immediately.

This v1383 section corrects the earlier framing of full compressed
image reload as the only possible design. It records a feasible
prefix-only alternative, **not** its completed implementation.
The v1381/v1382 native component acceptance and production linking
state remain unchanged.

## v1381/v1382: zero-copy PURE release by an atomic MM rebase

**Preferred staged PURE-only design, 2026-11-10. Not production linked.**

Earlier v1379 DROP needed a second physical allocation for the raw
mutable suffix and copied every word into it before releasing the old
APR extent. This was unnecessary: the suffix already occupies the
correct high-address part of the original physical extent. Retain it
without a physical copy by changing the original sorted MM extent
descriptor from:

`PROCESS [base, base+words)`

to

`PACKED_PROCESS [base+pure_skip, base+words)`.

This releases the lower PURE prefix immediately as a free gap in the
implicit MM allocator. The descriptor stays in the correct sorted
position: the preceding extent still ends no later than the old base,
and the next extent starts no earlier than the original end. The
owner and physical pin count are preserved. No extra extent or
per-process metadata is allocated, and no user memory word moves.
The retained suffix is NOT an APR mapping: it is explicitly
nonrunnable until full restore.

`system/kernel/mm/mm_pure_rebase_v1381.s` is a new standalone assembler
primitive, with its own header, requiring one pin, an owner-matched
MM_TYPE_PROCESS extent, exact old size, 02000-octal APR alignment,
nonzero 0200-octal-word-multiple PURE prefix shorter than the original
process, and a range within the full 18-bit physical address space
(exclusive upper boundary `01000000` octal). It checks ALL failure
conditions before two descriptor stores. Rejected requests change
neither MM metadata nor data. No call to the old
`mm_process_shrink_pinned` is made, because that API must continue
to enforce the 02000-octal APR process size granularity.

The staged `system/kernel/proc/proc_pure_ram_v1381.[ch]` replaces v1379
DROP while retaining registry-free RESTORE and DISCARD behavior.
It validates the resident state and exclusive process transition,
pins the original extent, disables PI for a short critical section,
calls the rebase primitive, and publishes stable PURE backing,
nonresident VM base and the existing packed mode **before restoring
PI**. Inactive suffix pin removal uses the new physical base.
An MM validation failure leaves the original image complete,
restores PI and removes its temporary pin. If post-commit unpin fails,
the caller retains TRANSITION; neither a partial process image nor
spurious disk swap record becomes runnable.

The primitive itself does not disable PI: the caller must exclude
interrupt-time MM mutations until the MM record and process storage
mode are consistent. The critical section contains no data copy,
filesystem I/O, memory allocation or retry loop. The descriptor
search is bounded by MM extent count. The only fallible step before
publication is the exact preflight and it has no side effects.

The v1381 host test exercises zero additional physical allocations
or frees during DROP, repeated drop/restore, exact suffix integrity,
MM pin/type/owner validation, PI pairing, allocation and reload
failures, and ZOMB DISCARD. The native PDP-6 v1381 regression
links the actual KCC transaction C, new MM assembler rebase and
existing assembler mode handlers. Positive, negative and physical
18-bit boundary cases PASS. The v1382 native regression additionally
links actual resident PURE vnode/mount busy scanning: source guard
stays effective through the zero-copy descriptor rebase, return to
resident state, second drop and terminal release. High mount 18 and
low-slot alias rejection PASS.

During this work, the older host MM fixture was corrected to reject
**overlapping physical intervals**, not merely equal allocation
starting addresses. The previous mock could incorrectly allocate a
full restored process image over its still-authoritative in-place
suffix after a failed reload. With the correct check, v1377 and
v1381 host regressions PASS. This is a test-fixture correction, not a
production allocator change.

### Memory and text cost

| Example, decimal physical words | v1379 copy/drop | v1381 zero-copy |
| --- | ---: | ---: |
| Original APR process | 1,024 | 1,024 |
| Additional suffix allocation during DROP | 768 | **0** |
| Peak during DROP | 1,792 | **1,024** |
| Final inactive suffix | 768 | 768 |

For the same source/object compilation mode, v1379 DROP/RESTORE/
DISCARD C was 567 PDP-6 KCC TEXT words; v1381 C is 518 and
`mm_pure_rebase_v1381.s` adds 70, total **588 standalone
pre-fold words**, zero static DATA/BSS. The increase is 21 TEXT
words for zero temporary RAM allocation on DROP. These are
component measurements, NOT a permanent KCORE linked-size delta.
Neither source is in the production Makefile.

**Still open:** RESTORE must presently retain the raw suffix while
allocating a complete aligned APR destination and reloading PURE.
For compressed executable formats the reload may overwrite the
entire destination; it cannot simply reconstruct a missing prefix
in place and overwrite the only valid writable data. Therefore
RESTORE peak and no-partner pressure tests remain blockers, as do
hardware-PI allocator concurrency, live scheduling, the real
VFS/provider lifetime, EXEC/RUN/WAIT/EXIT and a full 32K/48K
swapless tape-root guest. Native tests use controlled MM/PI/provider
fixtures; they do not certify any of these missing integrations.

## v1379/v1380: registry-free RAW PURE staging (not production)

The v1377 dense/sparse inactive-process layout registry is **not
necessary for the raw PURE-only case**. In the new, still unlinked
`system/kernel/proc/proc_pure_ram_v1379.[ch]`, the already authoritative
MM_TYPE_PACKED_PROCESS extent contains exactly the saved mutable
suffix and its process-slot owner. The original logical process size
is preserved in `vm_state`; the PURE sector-rounded omission is
derived from the validated executable backing record. Exact MM
type/owner/count checks reject duplicate or mismatched extents.
This avoids both extra per-slot metadata and lookup through an
independently allocated registry. The existing resident
`proc_storage_expect_v30` and `proc_storage_publish_v30` assembler
operations still own the two-bit process mode transition.

The source is compiled only with `DAIMOS_SWAPLESS_PURE_RAM_V1379=1`;
no production KCORE build selects it, and the v1377/v1376 prototypes
remain unlinked. Both host and native KCC/PDP-6 v1379 tests PASS
repeated drop/restore, original mutable memory preservation, raw
fallback/no-codec behavior, source/destination MM ownership and
failure rollback, and ZOMB discard. The PDP-6 test links the real
assembler process mode service rather than the staged C map helper.

The v1380 native test also links the *actual* existing
`proc_swap_backing_busy` and `proc_swap_mount_busy` assembler
code. It proves that a PURE process's vnode and mount remain busy
while resident, across the old APR free, while stored as an inactive
raw RAM suffix, during the full reload, across inactive suffix free,
and after the second drop. The saved backing record in the stable
u-area supplies nonresident immutability; after terminal discard
the guards no longer report busy. The high public mount-id 18 is
tested with the separate credential extension bit and negative
checks for aliasing lower mount IDs. Ordinary resident RAW/COMPRESSED
swap and previously accepted OOM guards are unchanged.

### Resource measurement and remaining risk

| Isolated PDP-6 KCC text | Words |
| --- | ---: |
| Previous v1377 transaction alone | 672 |
| Revised v1379 transaction alone | **567** |
| Staged C storage-map/layout/owner dependencies removed | 121+310+273 |

The **704 removed words** are only an isolated test-link difference:
the production kernel already owns assembler mode handling, and the
old C registry never shipped. Neither number is a production
linked-size delta. v1379 has zero static data/BSS. Source-protection
acceptance is limited to the tested VFS busy scanners and controlled
MM/provider transitions, not arbitrary concurrent provider mutation
or a booted guest process.

The present drop still needs a **separate temporary copy of the raw
suffix** before freeing the old APR extent. This makes peak memory
pressure an explicit unsolved blocker for a 32K swapless machine.
The current `mm_process_shrink_pinned` demands 02000-octal-word
process extent granularity and cannot retag a live PROCESS extent
as a compact PACKED_PROCESS extent. Investigate a properly serialized
in-place MM conversion with a validated no-fallible-step-after-overwrite
contract. Never simply copy into the original extent and shrink
afterward without rollback, or publish released tail words while the
copy still uses them.

This work does not complete KINIT capacity setup, actual scheduler
admission, full PROCESS/EXEC/EXIT/WAIT teardown, physical 32K/48K
tape-root boot, or the required peak-RAM/high-water measurements.
Those remain open before production activation.
Fault tests currently cover allocation refusal, failed executable
reload and MM free refusal, not every ambiguous MM pin/unpin outcome.
If unpin fails with uncertain ownership, TRANSITION remains held and
an explicit owner-recovery path is required before production use.

## v1377: staged PURE-only inactive process lifetime

DAIMOS adds `system/kernel/proc/proc_pure_ram_v1377.[ch]`. This is
new, standalone, **unlinked** transaction code. It is not a production
KCORE or scheduler integration.

The three entry points are `proc_pure_ram_drop_v1377(slot)`,
`proc_pure_ram_restore_v1377(slot, reload)`, and
`proc_pure_ram_discard_v1377(slot)`. They reuse the existing
`proc_storage_runtime_v29` two-bit mode map, the inactive dense/sparse
`proc_packed_runtime_v38` descriptor layout and MM-authoritative
slot/type/owner checks. These are only storage primitives: their
historical PACKED names do not mean the new payload is compressed.
The retained image is **raw**, containing exactly the writable suffix
after the omitted complete 0200-octal-word PURE sectors. An initially
running process must first be quiesced, have its user AC state saved in
the stable u-area and hold its lifecycle transition.

Drop requires valid PURE text backing, a sleeping/stopped process,
transient source exclusion, a nonpinned process extent, and an available
registry slot. It pins the old APR mapping, reserves and pins a new
unaligned managed-RAM extent, copies every mutable suffix word, reserves
the registry layout, and frees the old process extent before publishing
the inactive storage state. Only then may TRANSITION be cleared. An
owner-checked failed operation before the MM free does not publish a
partial image. The stable u-area stores the exact backing key to
support the existing nonresident vnode/mount exclusion contract.

Restore verifies the same owner, backing, original word count, PURE
boundary and raw saved suffix length before allocating the complete
APR-aligned process extent without recursive reclaim. It initializes
omitted words, calls the validated executable reloader (which may
decode an entire compressed DXR payload), then overlays every saved
mutable word, before freeing the inactive MM extent. Failed image
reconstruction preserves the saved RAM extent. A successful restore
publishes one complete resident process extent. Discard releases
validated inactive RAM on terminal teardown (including ZOMB), clears
the backing record and the high mount-slot credential bit, and keeps
the process's broader EXIT transition under caller control.

**Build isolation:** the translation unit deliberately refuses
compilation without `DAIMOS_SWAPLESS_PURE_RAM_V1377=1`. It references
no BACKSTORE variables or services and cannot be accidentally pulled
into a normal swap-enabled build. The flag is only enabled by focused
tests today. Future minimal `TAPE_AUX=none` wiring must explicitly
select the source and its proven runtime dependencies.

The testkit host suite executes the actual transaction C and
storage-layout/owner C against instrumented MM and immutable-provider
stubs. Dense and sparse layouts, three consecutive cycles, allocation
refusals, provider reload failure, destination/source MM release
failure and ZOMB discard pass. It revealed a missing source unpin on
failed restore allocation; the production candidate now releases it
before returning. The same source compiled with PDP-6 KCC executes
on SIMH; the target test covers raw drop, a failed full-image allocation
and retry, correct mutable-data overlay after mock DXR reloading, second
drop and terminal discard. Both suites PASS.

The isolated KCC object is 672 text words and zero DATA/BSS.
The previously staged C support objects linked in the focused test are
121 words (storage map), 310 (layout) and 273 (ownership): **1376 text
words pre-fold total**, not the live KCORE increase. The test's own
533 words and fixture data are not production costs. Some layout/planning
code belongs only in KINIT, and the real kernel already has assembler
storage-mode operations; a future production split must account for that
before declaring these prototype costs irreducible. The historical
header interfaces emit nonfatal KCC identifier-truncation notes.

**Still open:** actual VFS executable write/unlink/truncate/unmount
serialization across drop publication, exact 32K/48K boot-time registry
allocation, real MM and scheduler wake/retry, process EXIT/EXEC/WAIT/PID
reuse, saved AC restoration and provider failure tests. No production
activation, full boot, RAM peak, size benefit or time benefit has been
measured or claimed. With a single typical 02000-octal process and
0400-octal omitted prefix, the data reduction is only 256 words before
descriptor/codec/code costs; this alone cannot justify adding 1376
resident text words. The v1376 RAM compressor remains optional and
unlinked. No disk swap behavior was changed.

## v1376: first staged codec component accepted (not a process feature)

The DAIMOS source now contains `system/kernel/core/kpack_ram_rle_v1376.c`
and its header, **unlinked from production**. It is a no-allocation,
lossless 36-bit-word codec for zero runs, repeated nonzero words and raw
literals. It does not require aligned output, disk sectors, an MM table,
BACKSTORE or any dynamic scratch buffer. A measurement pass computes the
exact output length; the caller must reject allocations that would not
produce net physical-RAM savings, including metadata and restore headroom.
This is intentionally a simple memory-first staging baseline, not a claim
that its compressed size or CPU cost beats grouped D6codec.

`daimos-testkit/tests/system/swapless-ram-codec-v1376` verifies host
roundtrips, exact-capacity refusal, malformed control/length rejection
and the maximum 18-bit word count. The native regression in
`tests/validation/swapless-ram-codec-v1376` compiles the same C source
with KCC and runs its encode/decode and failure ABI on a real PDP-6
SIMH instruction path. Both PASS. KCC/DAS/DLINK independently accept
the implementation (449 text words, zero static data/BSS in the isolated
KCC-linked object). **449 words is too expensive to accept without a
complete swapless KCORE/MRES savings comparison**; neither compression
nor PURE-only inactive-state publication is integrated into production.
The initial PURE-only no-compression lifecycle must still be established
before any scheduler admission or RAM-compression integration.

## Applicability and exclusions

This mechanism is for configurations with **no process backing device**,
especially the build-time minimal `ROOT=tape TAPE_AUX=none TAPE_TMPFS=0`
profile. Discovering a disk or drum at runtime must not implicitly enable
disk swap or alter this mode. The initial target is stopped, sleeping or
otherwise schedulable-but-inactive user processes; the current running
process, active executive continuations, real-time pinned owners and any
unresolved VM/MM/provider transaction cannot be discarded or compressed.

Two independently enabled techniques are permitted:

1. **PURE executable-text dropping.** On a process with validated immutable
   executable backing and a software-immutable PURE prefix, retain all
   changeable state in RAM and omit the reconstructible PURE prefix from its
   inactive representation. At restoration, reload that prefix from the
   original vnode, *then* restore saved writable words exactly.
2. **Opt-in RAM compression of changeable process memory.** An approved
   process may retain a lossless compressed copy of its mutable user image
   **in managed RAM** while inactive. If compression is not worthwhile or
   cannot be completed atomically, preserve its original resident image.
   There is no disk record, disk fallback or BACKSTORE allocation.

PURE dropping is not synonymous with MEMFS execute-in-place (XIP): a normal
PURE DXR is still copied to a private APR extent while executing, whereas
restricted XIP executes from a pinned MEMFS-owned extent under a different
trusted no-write policy. Neither feature adds general shared text mappings.

## 2026-11-16 v1622: optional mutable bitmap compression contract (TODO)

The native v1451 zero-presence bitmap encoder/decoder (30 TEXT
words, zero BSS) is tested independently, not production-linked.
Only a validated executable explicitly marked
COMPRESSIBLE_MUTABLE may compress its mutable suffix. PURE
prefix omission is independent of this opt-in. EXEC replaces
eligibility, fork inherits it, and the stored RAW/BITMAP format
is tracked separately. No such executable flag or process
compression transition has been implemented. The existing DXR
COMPRESSED flag identifies the file payload, not RAM eligibility;
the three occupied flag bits above the 15-bit BSS field require
an audited backward-compatible header extension.

Prefer an ownership-preserving, overlap-safe in-place encoder
instead of allocating another full mutable image. Preflight
exact bitmap size and scratch needs read-only; incompressible
images remain RAW. Before any in-place source overwrite,
prove non-failing completion under an exclusive process lease
or exact recovery of a partially transformed stream. No
intermediate representation may become runnable or published.

The preferred temporary spill location during encoding is
the process's own still-owned and pinned reconstructible PURE
prefix, released only after the last scratch read. Restore
obtains an independent lease, because the original prefix and
compressed tail may already be partially reused by other
processes. The resident fs_block_workspace is 0200 octal words
and also serves serialized VFS/BCACHE: it is NOT free scratch.
Borrow it only with a verified exclusive, cache-invalidating
lease that excludes nested filesystem I/O, callbacks and
deadlock. A pinned no-reclaim dynamic MM extent is a fallback;
avoid a new permanent scratch slab by default. Prove the
precise forward/reverse/shifted overlap bounds rather than
assuming one 36-word-group bitmap per group suffices.

Every reconstruction destination must be fully MM-owned and
pinned before writing. Test forward and reverse decode (including
restoration from the end), partially reused prefix or tail,
another owner occupying a gap, and safe disjoint full-APR
relocation when overlap cannot be proved. Preserve the only
valid encoded image and immutable backing on every refusal.
Prioritize native and physical functional/rollback tests in
daimos-testkit; do not impose a standalone codec efficiency
benchmark during this integration phase. This does not change
the later production linked-size and peak-RAM acceptance gates.
Authoritative checklist: DAIMOS KCORE.MD and TODO.MD v1622.

## Hardware and layout invariants

The PDP-6 APR provides one contiguous process mapping, one relocation base
and 02000-octal (1024 decimal)-word alignment, **not** independent
text/data mappings or a text write-protection bit. Thus PURE cannot be
freed independently while a process is executing with private mutable data.
The first implementation must quiesce the process, save its CPU/u-area
context, and transactionally create an **inactive representation** before
releasing the original APR process extent. Restore the full contiguous APR
extent before that process can run again. Do not emulate page faults or
weaken APR isolation.

The existing single-word PURE backing record and
`proc_swap_reload_pure` are useful starting points; they do **not** already
provide an accepted swapless representation. In particular, the current
disk-swap omission rounds the PURE prefix down to full 0200-octal sectors.
An in-RAM representation may use exact word boundaries if that reduces
RAM, but it must retain the entire mutable boundary, including its
partially shared text/data word or block, and account for relocation,
startup state, zero-fill/BSS, heap and stack. Never infer immutability
solely from a compressed-executable header.

Persist the original logical image word count, PURE boundary, executable
vnode/mount generation and source integrity contract, retained mutable
word count, actual representation type and exact encoded word count.
Reuse existing authoritative process/u-area/storage-map fields where
possible; measure any extra per-slot state against actual RAM savings.
A process must never be marked resident while its full APR image is absent.

## Transaction and safety policy

- **Admission:** restrict PURE dropping to validated immutable image
  sources and valid PURE metadata. Reject self-modifying or unknown images.
  Obtain and retain the executable/provider lifetime guard *before*
  publishing an inactive representation. Reject write, truncate, unlink
  and unmount while an inactive process depends on that source.
- **Opt-in:** compression eligibility belongs to the validated current
  executable or explicit privileged policy, not an untrusted user bit.
  Default is uncompressed. EXEC resets admission; any supported fork must
  define eligibility inheritance. The actual RAM representation is recorded
  independently from eligibility.
- **Reclaim:** quiesce the candidate with valid scheduler, PI6 and MM owner
  state. Either retain an independent MM destination as previously specified,
  or use the v1622 overlap-safe in-place design only after proving exclusive
  ownership, sufficient spill and non-failing/recoverable source transformation.
  Validate complete encoding/count before publishing the shorter extent or
  releasing any original process words. Never publish a partial stream,
  overwrite another owner's memory, write hidden disk backing or release
  pinned scratch while it is live.
- **Benefit test:** reject non-shrinking compression *after* accounting for
  encoded words, metadata, preserved context and temporary peak RAM. Try
  PURE omission without compression when beneficial. If there is no
  provable net gain, keep the original resident image.
- **Restore:** reserve a complete APR-aligned process extent (including
  writable words), including the overlap source only under an explicitly
  proven safe MM transaction. Choose forward/reverse reconstruction or
  disjoint relocation based on present ownership, not the former address
  range. Reconstruct immutable PURE from the guarded vnode,
  restore/decompress mutable words, validate exact counts and ownership,
  then atomically publish runnable process state. On failure, retain the
  original inactive representation and its backing guards; do not run a
  partial image. A nonrecoverable lost backing source is a typed failure,
  not permission to invent replacement text.
- **Progress:** a compact RAM representation does not guarantee enough
  memory for its later expansion. Reserve/measure restoration headroom,
  apply bounded reclaim/compaction attempts, and fail safely on genuine
  insufficient physical RAM. Prevent cyclic waits on a filesystem provider
  owned by a process that must first be restored. Restore never allocates
  disk backing or forces a provider lock free.
- **Teardown:** EXIT, TERM, fatal event, EXEC replacement, PID reuse and
  MM failure rollback must release exactly the owned representation and
  any executable hold once, even if the image is inactive. Preserve
  per-process status and parent WAIT semantics.

## Implementation order and acceptance

1. Establish an exact current KCC/KCORE size, call-graph and physical-memory
   baseline for a **swapless** 32K/48K tape-root boot. Confirm which SWAP
   gateways/bookkeeping the no-backing link still truly needs; do not
   reintroduce removed disk-swap services in that profile.
2. Specify the minimal inactive-state descriptor and validation contract
   against real `proc_storage_map_v28`, u-area, MM owner/pin and scheduler
   fields. Reuse validated PURE source/loader functions without adding a
   new duplicate decoder.
3. Implement PURE-only stop/drop/restore first, with original writable
   bytes retained verbatim in RAM and **no** compression requirement.
   Verify native PDP-6 resume, repeated cycles, executable protection,
   low-memory and failure rollback.
4. Only then add explicitly marked mutable-RAM compression using the tested
   v1451 bitmap codec where feasible. Establish header admission, read-only
   size/spill preflight, source ownership, overlap-safe encode/decode,
   direction selection, interrupted-transaction recovery and safe fallback.
   Prioritize functional native PDP-6 and real-guest tests rather than a
   standalone codec speed/efficiency benchmark. Do not link duplicate codecs.
   Keep the whole-system linked-size/peak-RAM gate for production activation.
5. Integrate scheduler reclamation and VM/BRK/EXEC teardown, prove bounded
   memory-pressure progress, then run mixed PURE/non-PURE,
   compressible/incompressible, multiple inactive processes, provider
   failure, parent WAIT and interrupt/preemption tests. All new tests live
   in `daimos-testkit`, including SIMH and PDP6-AAP where applicable.
6. Repeat the full KCC linked permanent-size and real physical high-water
   checks. Only production-enable after accepted 32K swapless native
   tests; keep the disk/drum raw-swap build behavior unchanged.

The inactive `proc_packed_*` v1-v46 sources are **historical prototypes**:
they are not linked into the production process lifecycle. Their MM
ownership and rollback ideas may be reviewed, but their old compressed
disk-swap eligibility/representation plan is retired. Do not reactivate
them by renaming or linking them without source-level integration and
independent native acceptance.

## Status of production source at this decision

`proc_swap_pdp6.s` currently omits PURE sectors when writing a process
to disk swap and reloads text through `proc_swap_reload_pure`. It does
not drop PURE text in a swapless system. `vm_pdp6_swap.c` still owns
ordinary raw swap bookkeeping. No swapless inactive RAM compression,
restore state or scheduler admission has been enabled. This document is
an implementation specification and not a claim of completed code.
