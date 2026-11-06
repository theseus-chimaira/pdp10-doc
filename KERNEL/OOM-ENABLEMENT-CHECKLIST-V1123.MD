## v1252: software PI6 must service deferred process timers (F07 OPEN)

Under real 256K PDP-6 RUN/swap load, slot 0 can spend many clock
ticks executing synchronous disk transactions. Clock PI6 still advances
the timer epoch and marks an expired frontier DUE, but it cannot switch
the slot-0 process while that transaction is active. The subsequent
software PI6 reschedule used to select another process without calling
`proc_timer_service`, so a due sleep could remain blocked across repeated
swap requests. This is a scheduler correctness defect, not an OOM or
memory-size optimization.

DAIMOS `00fb8ecb` makes `proc_sched_resched_choose` service the timer
frontier **before** selection, only when the DUE bit is set. The normal
clock-quantum path already services the frontier. The production fix
does not alter the timer state format, resource ownership or OOM policy.
The native PDP-6 testkit `b07202f`/`8959bfd` executes the real
software-reschedule, timer and run-queue leaves: an expired sleeper
becomes RUN and is queued before selection, an undued timer remains
unchanged, and an old-path negative mutant leaves the sleeper blocked.
The standalone timer-frontier and reaper-scheduler tests also PASS.

**This is not F07 acceptance.** The v1252-r1 256K stress guest admitted
all 18 additional RUN children and reached `^@%*`, including the timed
sleep stages. After about 14.5 minutes it had still not reached the
requester WAIT checkpoint or any child reap. A controlled stop preserved
the snapshot: the timer frontier was clear, with 32 successful swap-outs,
20 ordinary swap-ins, 32 exchanges, no recorded swap transaction failure
and no genuine BRK-OOM sample. The remaining issue is to distinguish
heavy swap traffic from requester scheduling starvation. This diagnostic
guest included the separate, unaccepted v1208/v1211 reclaim experiments
and was built before the final DUE-bit fast guard, so it does not certify
the exact committed candidate. Full simultaneous RUN/swap/WAIT/MM
lifecycle evidence is still required. E13/F07/F11 remain OPEN,
F12/F13 deferred, 59/64 OOM checks, production requester OOM OFF.

## v1205: real concurrent process/APR/BRK accounting under OOM (partial F07)

The DAIMOS `684d4504` and testkit `f2222e2` exact-frozen 256K
KCC DSK/D6FS PDP-6 release-media boots PASS both disposable OOM
policies. Unlike v1204's MM_TYPE_KERNEL_DYNAMIC-only stress, v1205
RUNs six extra independent user processes (twelve resident users)
before the foreign filesystem-provider lease is held, then captures
their actual process descriptors, stable u-area logical BRKs and
MM_TYPE_PROCESS owners at genuine physical BRK OOM.

Every live process's physical base and APR-rounded VM length matches
the MM table. Actual summed logical BRK/APR padding is **5499 words**,
not an assumed 1023-word maximum per process. Both policies show
231012 managed physical words allocated (95.3 percent) and 31/136
MM descriptors; enabled/disabled have 11415/11429 words free (the
14-word delta is the test-enabled OOM policy code). Child TERM/WAIT,
independent scheduler bystanders and original foreign FS owner
and private u-area teardown all PASS. Evidence is recorded in
DAIMOS `design/F07-REAL-PROCESS-BRK-ACCEPTANCE-V1205.md`.

The whole F07 gate remains OPEN: 12 real process images do not
exercise all 64 configured process slots or near-full descriptor
occupancy simultaneously. v1206 will combine the 12 real processes
with high descriptor occupancy; wider swap and stack concurrency
remain separately required. E13/F11 OPEN; F12/F13 deferred;
59/64 completed and production requester OOM OFF.

## v1204: high concurrent physical-MM/descriptor F07 pressure (partial)

The DAIMOS `75ccef85` / daimos-testkit `aa7cb86` independent
256K PDP-6 KCC DSK/D6FS guests PASS real physical BRK OOM with
a held foreign FS provider under **both** disposable enabled and
production-disabled OOM policies. A test-only, owner-tagged allocator
creates twelve APR-02000-word-aligned 8192-word blocks plus 96
single-word blocks after all six real processes are active and
the provider is leased; all 108 are later released exactly once.

At physical OOM, 211092/242518 managed words (87 percent) and
127/136 descriptors (93 percent) are occupied in the enabled guest;
the disabled build differs by 14 permanent test-code words only.
Six actual APR process extents, 121 kernel-dynamic extents, three
free gaps, 128-word shared filesystem workspace, and the original
foreign gateway owner are all accounted for. Protected processes,
exact FILE/u-area/MM cleanup and the provider waiter's identity
survive. Test-only host checks reject 17 invalid-snapshot mutations
across v1203/v1204. No production KCORE or ABI change is involved.

The earlier v1203 F07 verifier used the incorrect 511-word APR
alignment-loss bound. APR quantum is `02000` octal = 1024 words;
the corrected maximal per-gap and per-process rounding bound is
1023 words. Old v1203 observed gap lengths are unchanged, but old
**derived** upper-bound fields are superseded by v1204. Full exact
logs, checksums and method:
DAIMOS `design/F07-HIGH-PRESSURE-MM-ACCEPTANCE-V1204.md`.

This does **not** close F07. Pressure extents are genuine allocator
objects but are not actual concurrent user process slots; the full
64-slot profile and individual process logical BRK values/rounding
remain to be measured under high concurrency. E13 and F11 are also
OPEN, F12/F13 deferred, **59/64 completed**, production OOM OFF.

## v1202: A04/F02 exact-revision P0-D release matrix accepted

DAIMOS `5db15ecf` with daimos-testkit `122c61f` completed one
uninterrupted `P0D_FULL=1` v1096 run (exit 0) on the supported
256K KCC DSK/D6FS PDP-6 profile. All 18 component/source-linked
release checks, v1083 direct fatal-release scope, v1087 transitive
no-new-MM/backing-allocation graph, and all 17 full booted guest
invocations PASS. This covers physical TTY/PIPE/DPY/VM/u-area
MM_BUSY, owner-invalid -4 quarantine, held FS leases, genuine
swap-out/in and bitmap rollback, PID reuse and stale WAIT/event
rejection, and both disposable OOM policy variants. See
DAIMOS `design/A04-F02-FULL-MATRIX-ACCEPTANCE-V1202.md` for full
log provenance and SHA256 evidence.

A04 and F02 are now CLOSED, leaving **59/64 completed**, with
E13/F07/F11 pre-enable and F12/F13 deferred. This closure proves
the defined resource-release graph on this exact candidate;
it does not certify remaining cross-simulator/media coverage,
combined dynamic peak MM/APR/provider buffer budgets or F11
integration. Production requester OOM is still disabled.

## v1190: F10 four-profile build diagnostic audit accepted

The testkit v1190 release-build log audit PASSes Android/aarch64
and Linux/x86-64 full DSK/D6FS release media under both disposable
OOM policies: the same 13 precisely cataloged diagnostic classes,
zero unexplained compiler/assembler warnings or errors, and three
injected unexpected-diagnostic mutants rejected. Known KCC notes
and the legacy D6FSR2 warning remain visible, with actual D6FS
superblock CLEAN verification. F10 is CLOSED for this evidence;
the exact final-candidate recheck remains part of A04/F02/F11.

After retiring F05 as a non-OOM memory-size goal, **57/64 OOM
gates complete, 7 open**: A04, E13, F02, F07, F11 pre-enable;
F12/F13 deferred. Production requester OOM remains disabled.

## v1189: F05 removed from requester-only OOM acceptance

F05 (static 16K permanent KCORE/MRES fit) is **not an OOM target**
and is retired from this checklist, not marked as passed. The
measured instrumented 256K full-profile 2717-word permanent
address-budget overflow remains a separate KCORE/MRES optimization
problem, not a reason to block requester-only OOM signoff. Historical
v1188 and earlier F05 references below record the prior scope and
are superseded by this explicit decision.

OOM acceptance now has **64 gates: 56 complete, 8 open**. The six
pre-enable gates are A04, E13, F02, F07, F10 and F11; F12 and F13
remain deferred activation/post-activation steps. In particular,
F07 still requires real *dynamic* peak MM/descriptor, APR/alignment,
stack and provider-buffer accounting and cannot be waived with F05.
The selected 256K KCC/full DSK/D6FS profile and all failure-safety
tests remain unchanged. Production requester OOM remains disabled.

## v1188: F06 boot-installed MRES footprint reconciled; F05 still fails

The Android aarch64 and Linux x86-64 v1174-v1176 256K full
DSK/D6FS physical-OOM guests, both built from DAIMOS
`96e6c695` with corrected KCC `b9cd764`, each reconcile
the live SYS_MEMINFO resident words, a separate SIMH inspection
of KINIT's patched resident immediate, the KCORE map boundaries,
actual 136-slot MM descriptor capacity (272 permanent words), and
all 22 MRES objects installed by the MINIT boot sequence. The
installed MRES sum is 8837 Android or 8839 Linux words. A linked
but uninstalled package is NOT resident, but an *installed* MRES
is permanently reserved: runtime OOM never loads, unloads or
reclaims MRES.

The Android disabled instrumented guest occupies 19053 words
(9944 KCORE + 272 MM table + 8837 MRES). Including the low-core
start at octal `060`, its exclusive end is `045235`, which
exceeds octal `040000` by 2717 words. The enabled test guest
is 14 KCORE words larger and exceeds by 2731. These are direct,
boot-observed *instrumented* figures; they cannot be relabelled
an uninstrumented production-size certificate.

**F06 is closed only as the required live-vs-packaged inventory
and overflow investigation. F05 remains OPEN and the mandatory
16K permanent-memory acceptance FAILS for the measured profile.**
The booted 256K system works but does not meet this independent
memory-budget condition. With F03/F04/F06/F08/F09 closed,
**56/65 checks complete, 9 open**, and production requester OOM
is still disabled.

## v1187: F08 strict 32K profile remains a separate deferred goal

F08 is CLOSED as a scope/documentation boundary, not because a
32K GCC/DSH build has passed. The initial requester-only OOM
acceptance uses only 256K PDP-6 KCC/full DSK/D6FS as explicitly
frozen in `OOM-RELEASE-PROFILE-V1160.MD`. A successful 256K boot
cannot be counted as 32K resident-kernel/DSH acceptance. The
strict 32K KCORE/DSH image optimization remains deferred until
after OOM release work. The independent permanent executive/MRES
16K address budget is *not* waived or deferred by this decision;
F05/F06 must still resolve it. With F03/F04/F08/F09 closed,
**55/65 mandatory tasks complete, 10 open**, production OOM OFF.

## v1185: F04 negative/error-cause and F09 recovery policy CLOSED

On DAIMOS `96e6c695`, KCC `b9cd764`, Linux/x86-64 PDP-6,
the completed F03 two-policy matrix plus eight further booted or
source-linked tests establish the bounded *writable-process*
cause/owner contract: v1113 real MM -2/-3 with unsafe mutations
rejected, v1156 real backing bitmap exhaustion in both OOM policies,
v1158 separate failed RUN parent and OOM BRK requester in both
policies, v1157 APR/postcommit EXEC while an independent FS provider
retains its original lease, v1106 physical pinned VM/u-area retry,
and v1077 physical held D6FS provider with MM_BUSY. All PASS.
The eight new logs are archived on Linux as
`$HOME/tmp/daimos-f04-cause-evidence-v1184.tar.gz`, SHA256
`5c12dce02fe1877a47c19afa9e9339e1a466a39f79526d1960617a048ee69096`.
F04 is CLOSED for the first release scope, not optional raw XIP.

The completed operator policy `OOM-FAIL-CLOSED-RECOVERY-V1185.MD`
enumerates known permanent or retryable teardown refusals and
permitted operator interventions. A retained original provider
or invalid MM owner must never be force-unlocked, force-freed or
reported as successfully reaped merely to finish an OOM kill.
KINIT MRES installation failures are boot failures, not runtime OOM.
F09 is CLOSED as a documented noncompletion/restart policy, not
as a bounded-liveness guarantee.

**54/65 mandatory gates complete, 11 open**, including F05/F06
resident-memory requirements, F02 corrected-toolchain P0-D final
run, E13 additional platform/media coverage, F07/F08/F10/F11,
and post-enablement F12/F13. The production policy remains OFF.

## v1181: F03 exact-candidate positive/negative matrix CLOSED

Linux/x86-64 Open SIMH V4.1-0 built 14 independent full 256K PDP-6
disposable guests for physical requester OOM (v1075), invalid/address
BRK (v1110), failed RUN rollback (v1112), real MM descriptor-table
exhaustion (v1114), physical fragmentation (v1115), original MM cause
propagation from failed RUN (v1116), and 24 PID reuse cycles with
stale WAIT/TERM rejection (v1120). All seven scenarios PASS in
production-disabled and isolated test-enabled variants on DAIMOS
`96e6c695`, corrected KCC `b9cd764` and Open SIMH
`edd19b17`. No production kernel source or OOM policy modification
was necessary. After the KCC AC10 preservation repair, v1114's
test-only 32-entry descriptor-base limit was corrected to the actual
136-descriptor full 256K capacity and both variants PASS.

The complete 14-log archive and compiler/simulator/source hashes are
preserved on the Linux test host in
`$HOME/tmp/daimos-f03-verified-evidence-v1181.tar.gz`,
SHA256 `dab45fce94cd6350bfb32e7b1683010f85bb7d93e4a6fd60b4996d17d49bb6bc`.
**F03 CLOSED; 52/65 mandatory checks complete, 13 open.** This does
not close F04-F11 or authorize production OOM. A full Android P0-D
candidate run is still pending; earlier fresh Linux P0-D boot matrix
passed. Separate same-source Android/Linux E13 release-media counter
verification passes, but the remaining required platform/media
coverage remains outstanding.

The F05/F06 live physical memory audit proves that MINIT-installed
MRES images are permanent, not runtime-loadable/unloadable OOM
resources. The current instrumented full DSK/D6FS profile occupies
19053 words (OOM off) and 19067 (test-enabled); this exceeds the
separate 16K permanent address budget. The release must not silently
waive that budget, confuse packaged with installed MRES, or activate
OOM before F05/F06 are resolved.

## v1163: E13 second-platform evidence PASS, final signoff still open

Linux/x86_64 Open SIMH PDP-6 build `a6b511e5` ran independent
256K KCC/DSK/D6FS `v1160` actual-release-media guests with physical
BRK OOM default-disabled and test-enabled. Both PASS held foreign
provider lease, requester-only victim selection, protected INIT and
bystanders, correct original-owner return, 160 runnable spinner
advances and exact WAIT/MM/FILE cleanup with zero integrity errors.
The Linux evidence archive SHA256 is
`b80eb98991845b89fb56344cb621c22eb3e8fd79aff185912163c5406f3e86fd`.
New testkit `v1163` cross-platform checker confirms both Linux and
Android/aarch64 archives, distinct simulator binaries, actual
process/lease counters and matching instrumented KCORE low ends
`023410`/`023426` for disabled/enabled.

Original Android v1160 used DAIMOS `27acb950`, Linux used `90dfc8d0`
(strict-C99 host-tool repair), so final E13 same-revision coverage
remains outstanding. The Linux toolchain's older AUXSTORE formatter
was updated and v636/v638 safety tests passed on both systems.
Other required simulator/media targets and F05/F06/F10 budgets and
diagnostics remain open. **51/65 checks complete, 14 open**, OOM OFF.

## v1160: first release profile frozen; E13 DSK/D6FS evidence PASS

S07/S08 freeze the first bounded requester-OOM profile, described
in `OOM-RELEASE-PROFILE-V1160.MD`: 256K full KCC PDP-6, DSK270
Stage 1, four-member D6FS disk root, DRM236 AUXSTORE available,
60 Hz PCLK, CTY headless. Normal RUN/EXEC uses writable VM;
root-only raw MEMFS XIP admission and 32K/alternate-media/GUI
profiles are excluded from *this* OOM activation scope, not removed
from DAIMOS. v1123 source/mutation scope guard PASSes again.

Two fresh 256K full-system v1160 guests PASS genuine physical BRK
OOM while a foreign gateway holds its original lease: enabled policy
selects only the requester and disabled returns -1 without killing.
Each has 160 independent spinner advances, exactly one original
provider return, correct WAIT/process/MM counters and zero integrity
errors. Actual generated DSK/D6FS media were attached; full build
logs, SIMH/counter transcripts, boot.ini, maps and hashes are archived
in `$TMPDIR/daimos-e13-release-media-v1160.qel3Zm.tar.gz`.
This is **one local Android aarch64 Open SIMH PDP-6 target**. E13
remains OPEN pending other required simulator/release-media variants.
The guest uses a test-only INIT, so ordinary userland swap activation
is not asserted. No production kernel change or OOM activation.

S07/S08 CLOSED for this explicitly bounded profile: 51/65 mandatory
checks complete, 14 open (12 pre-enable plus F12/F13). E13, A04,
and F02-F11 remain. Production requester OOM OFF.

## v1159: E12 protected live critical/nonrequester processes PASS

Fresh full 256K PDP-6 testkit v1159 regressions PASS in both
isolated requester-OOM-enabled and production-disabled modes. A
disposable guest extends the original real v1150 held-provider
physical BRK OOM scenario with read-only actual process-table
snapshots and fatal-branch counters at the genuine -1 MM decision.
Requesting slot 3 is the only fatal selection in the enabled guest
(one call); the disabled guest returns BRK -1 and records zero fatal
selections. INIT slot 1, blocked witness slot 2, provider waiter
slot 4, runnable/sleeping spinner slot 5, and original in-flight
gateway owner slot 6 all remain active at the shortage. The original
provider lease `7,,0` remains held through the OOM and is released
later on the original owner's private continuation. Both guests
PASS 160 independent spinner advances, exact existing WAIT/MM/FILE
cleanup and zero integrity errors; the enabled mode records seven
fatal FILE close BUSY retries.

E12 proves *nonrequester* protection: it does not establish an INIT
exemption if INIT itself invokes failing BRK. No permanent code,
data, process state or ABI changed and production OOM remains OFF.
E12 CLOSED for supported writable-process requester-only scope:
49/65 mandatory checks complete, 16 open (14 pre-enable plus
F12/F13). E13, S07/S08, A04 and F02-F11 remain outstanding.

## v1158: E11 distinct RUN parent survives genuine BRK OOM

Fresh full 256K PDP-6 guests PASS
`test-daimos-oom-run-parent-v1158` in both default-off and
disposable requester-OOM-on configurations. A separate live RUN
parent fills the production MM descriptor pool using a test-only
probe, encounters genuine -3 while admitting a child, verifies
no published child process slot or leaked process RAM, restores
the original MM descriptors, and acknowledges INIT through a pipe.
INIT then starts a distinct child which experiences actual physical
BRK exhaustion. The RUN parent remains alive until the latter's
terminal WAIT: requester-only fatal retirement occurs only with OOM
enabled, while default-off returns -1. The original RUN parent is
then released and reaped with exact process accounting. No new
production KCORE, MRES, permanent data or ABI changes.

E11 CLOSED: 48/65 mandatory checks complete, 17 open (15 pre-enable
plus F12/F13). E12/E13, S07/S08, A04 and F02-F11 remain open;
the production OOM policy is still disabled.

## v1157: E10 APR/EXEC/RUN during held foreign provider accepted

Fresh full-system 256K PDP-6 `test-daimos-fatal-exec-provider-v1157`
PASS against DAIMOS `a7b721b4`, testkit `436552f`. The disposable
guest holds a real foreign filesystem gateway lease while an
asynchronous APR fault occurs in post-commit EXEC FILE cleanup and
an unpublished RUN rolls back its physically pinned u-area.
Provider-backed FD3 survives staging until real foreign-lock refusal.
The original owner later resumes/unlocks and final private-owner
FILE, VM/u-area and WAIT accounting PASS. Counters: six VFS calls,
five refusals, seven FILE passes, six sleeps, one physical
pin/busy/unpin cycle, zero integrity errors. This is a real held
provider and actual physical MM refusal in a test-only copied guest,
not a change to the production OOM policy or permanent memory.

E10 CLOSED for supported writable-process scope: 47/65 mandatory
checks complete, 18 open (16 pre-enable plus F12/F13). E11-E13,
S07/S08, A04 and F02-F11 remain. Production requester OOM OFF.

## v1156: E09 real physical MM versus backing exhaustion accepted

The frozen `test-daimos-oom-error-matrix-v1156` nine-case suite
PASSes actual 256K PDP-6 and native cases against DAIMOS
`9f51326e`, testkit `81f40eb` and frozen host-tool checksums;
logs are preserved under
`$TMPDIR/daimos-oom-error-matrix-v1156.gzw27g`.
The tests distinguish true physical BRK shortage (-1), fragmented
free extents (-2), MM descriptor exhaustion (-3), invalid or
wrong-owner (-4), and pinned MM_BUSY (-5). A new disposable guest
exhausts the real production backing bitmap and checks the next
allocation fails atomically, without killing the process; it then
successfully grows and shrinks ordinary BRK and releases backing.
Production-disabled and disposable requester-OOM-enabled tests
both PASS, and native backing allocator mutants are rejected.

E09 is CLOSED: 46/65 mandatory checks complete, 19 open (17
pre-enable plus F12/F13). E10-E13, S07/S08, A04 and F02-F11
remain mandatory. No production KCORE/MRES/RAM or ABI change and
`PROC_OOM_BRK_ENABLED_V1000` remains zero.

## v1155: E08 multiple sessions, stopped groups and PID reuse accepted

Two complete full 256K PDP-6 v1155 guests PASS with two distinct
live foreign session leaders and two simultaneous independent
two-member process groups. Each group receives actual TSTP/CONT,
then terminal TERM/INT; each member has exact STOPPED, CONTINUED
and EXITED parent WAIT statuses and CHLD notification. The guest
rejects duplicate status and stale PID event delivery and checks
that neither foreign session is killed by unrelated group events.
With both sessions still live, a separate child is RUN/reaped for
64 additional numeric PID lifetimes. Each has a real ALRM and
terminal WAIT, then rejects stale WAIT/TERM; the two sessions are
finally terminated and reaped one at a time. Process slots, process
words and FILE slots return to baseline. Original v979 parent/group
regression also passes on a separately rebuilt full guest.

The first in-development v1155 guest timed out before reaching its
group phase. The completed v1155 r2 and independently rebuilt r3
both PASS the full sequence; the testkit source is recorded in
commit `0ac6c83`. No production source or RAM changed.

E08 is CLOSED, with **45/65 mandatory checks complete, 20 open**
(18 pre-enable plus F12/F13). E09-E13, S07/S08, A04, F02-F11
and final actual RAM measurements remain open. Production OOM is OFF.

## v1153-v1154: E07 qualified PI6 and independent user progress accepted

The v1153 full 256K PDP-6 mixed release guest PASSes all original
v1144 physical PIPE, Type-340 DPY, VM, u-area and foreign-provider
ownership assertions, plus additional real PI6 and unrelated-user
witnesses. From the first genuine PIPE MM_BUSY to final ZOMB
publication, PI6 advances 1297 -> 1352 (55 ticks) and the bystander
executes three real sleep/wakeup operations, with one exact release
per resource and zero errors.

The v1154 independently built guest extends the v1122 physical PIPE
and checked DISK bitmap refusal/restoration scenario with a distinct
user session. From the first failed real bitmap verification to the
second successful verification, PI6 advances 1015 -> 1032 (17 ticks)
and that unrelated user executes eight real sleep/wakeup syscalls.
The two physical bitmap calls, one corruption/restore, original
DISK backing record, exact PIPE release, parent WAIT and TTY ownership
all PASS without errors. Both tests change only disposable guests;
there is no synthetic clock, allocation result or production code.

The independently booted v1148 TTY/HUP/VM/u-area case additionally
PASSes 32 PI6 ticks and 48 bystander sleeps through pinned TTY and
deferred owner continuation. Together these accept E07 for all
currently supported writable-process resource classes. Mandatory
checks: **44/65 complete, 21 open** (19 pre-enable plus F12/F13).
E08-E13, S07/S08, A04, F02-F11 and actual permanent/peak RAM proof
remain mandatory. Production requester OOM remains disabled.

## v1151r1: E05/E06 sustained physical-OOM endurance accepted

Both independent 256K PDP-6 full disk-root v1151 guests PASS with
production-requester OOM disabled and test-only requester OOM
enabled. Each completes the initial real physical BRK exhaustion,
32 additional physical-OOM waves, 32 subsequent successful RUN and
BRK growth/shrink operations, 256 numerical PID-reuse lifetimes,
512 rejected stale WAIT/event requests and a minimum 1024 real
qualified PI6 clock ticks. Every wave checks exact process slots,
process words, FILE slots, non-cache MM descriptors and disk swap
backing, including the correct victim and real MM shortage cause.

The initial v1151 raw MM extent-count equality failed after the
first successful recovery RUN because the real BCACHE allocated its
normal single clean `0201`-word, kernel-dynamic owner-6 slab at
physical `0777577`. Inspection of the preserved halted guest shows
this is intentional reclaimable cache residency, NOT a stranded
process/u-area allocation. The corrected disposable kernel exposes
the real cache base read-only. All non-cache MM extents must match
the baseline exactly, the cache is bounded to one slab and a later
true physical OOM must reclaim it. The disabled guest correctly
prints a `D` on each normal BRK `-1` return; the corrected wrapper
requires exactly 32 extra such markers. Both final full guests PASS.

Mandatory checklist: **43/65 complete, 22 open** (20 pre-enable
plus F12/F13). E05/E06 are CLOSED, but E07-E13, the final release
configuration, full F02-F11 matrix, warnings and real RAM budgets
remain open. The production requester-OOM switch is still OFF;
there is no production KCORE, MRES, process ABI or static-RAM change.

## v1150: B06 physical OOM under live foreign provider

Both `test-daimos-oom-provider-held-v1150` and
`test-daimos-oom-provider-held-disabled-v1150` PASS independently
rebuilt disposable full 256K PDP-6 guests. The requester owns a
genuine open FILE before an unrelated process obtains the serialized
filesystem provider lease (slot 6, encoded `7,,0`). A third process
sleeps on the real pre-dispatch gateway wait event while the holder
remains suspended on its own in-flight private executive stack.
An unrelated user remains runnable.

At the genuine BRK failure with physical-MM cause -1, the holder
lease still belongs to slot 6, the pre-dispatch waiter is still
sleeping, and no wrong-owner release has occurred. With requester
OOM opt-in enabled only in the disposable guest, the requester alone
is terminated; seven real foreign-provider FILE-close attempts retain
the provider and the owner's FILE until it resumes and unlocks from
the original continuation. In the default-off guest, BRK returns -1
and ordinary EXIT instead sleeps on the provider event once. Both
variants record one in-flight holder entry/return, one physical
exhaustion event, 160 independent spinner advances, final WAIT and
surviving witness, zero instrumentation errors and recovered MM/slot
accounting. No synthetic MM allocation return was used.

B06 is CLOSED for the current writable-process scope. Mandatory
checklist: **41/65 complete, 24 open** (22 pre-enable blockers plus
F12/F13). S07/S08, A04, E05-E13 and F02-F11 remain open. The
production OOM switch remains disabled and no KCORE/MRES/RAM/ABI
change was made.

## v1149: B07 capped backoff does not authorize forced reclamation

The new `test-daimos-oom-unbounded-owner-v1149` source/negative
contract PASSes against the actual production PDP-6 owner FILE/HUP,
foreign provider-lock and slot-0 finalization assembly. All eight
unsafe mutations are rejected, including stopping an owner at the
01000-tick delay cap, directly freeing its MM, stealing/unlocking a
foreign provider lease and treating an unsuccessful slot-0 retry as
committed. Native v885 executes 14 failed FILE close requests with
multiple capped delays before one legitimate successful EXIT; native
v1102 covers shared reaper backoff and four negative mutations.

B07 is CLOSED as a fail-closed ownership contract, not a bounded
provider-liveness guarantee. A provider that never returns may keep
its owner indefinitely and requires a separate recovery policy.
Mandatory checklist: **40/65 complete, 25 open** (23 pre-enable plus
F12/F13). B06 real OOM with held/in-flight providers, E05-E13 and
F02-F11 including RAM/diagnostic signoff remain open. Production
requester OOM is OFF and no resident KCORE/MRES/ABI change occurred.

## v1148: C04 integrated private-owner HUP and real PI6 closure

Two independently rebuilt 256K PDP-6 v1148 guests PASS the combined
C04 scenario. The original session leader's first EXIT broadcasts a
remote HUP while the attached canonical TTY MM extent is physically
pinned and returns genuine MM_BUSY. One child records exactly one HUP
deferral on its private executive continuation and exactly one HUP
delivery at its user-return boundary; parent WAIT checks the actual
HUP exit status. The original session is retained until the retry.
The second leader EXIT then encounters actual pinned process VM and
u-area MM_BUSY and completes through the normal slot-0 retry clock.
Each guest observes one pin/refusal/unpin for TTY, VM and u-area, one
final commit and no hook errors. PI6 advances 32 actual ticks after
the refusal phases, with 48 sleeps of an unrelated session process.
The existing v1098 clock/backoff source contract passes all three
negative mutations. No fake timer tick or allocator return is used.

C04 is CLOSED for the current writable-process OOM scope. Mandatory
checks: **39/65 complete, 26 open** (24 pre-enable plus F12/F13).
B06/B07, E05-E13, F02-F11 and real RAM/diagnostics signoff remain
open. Production requester OOM remains disabled; no kernel, MRES,
persistent RAM or ABI change was made.

## v1146r3: D16 exact-once matrix closed on frozen candidate

The nine-stage `daimos-testkit` target
`test-daimos-oom-exact-once-matrix-v1146` PASSes completely against
DAIMOS `2e12c1da` and testkit `253a813`, using the fixed installed
P10FOLD, DLINK, DAS and PDP-6 simulator checksums recorded alongside
the retained test logs. Its cases cover the transitive fatal-release
graph and mutants, physical PIPE/DPY/VM/u-area pin/BUSY/unpin/free,
canonical TTY, FILE/HUP refusal and foreign provider-lock retention,
swap-out/in and bitmap-corruption/refusal recovery, the nested
APR/EXEC/unpublished-RUN rollback, exact owner/ZOMB/WAIT publication,
bystander progress, and stable PID/MM/slot accounting. Both normal
OOM-disabled and disposable requester-OOM-enabled variants complete
24 PID-reuse lifetimes and distinguish the original requester from
unrelated processes. D16 is CLOSED for the current writable-process
scope; scheduled XIP launch is still explicitly excluded.

Mandatory OOM checklist: **38/65 complete, 27 open** (25 pre-enable
blockers plus F12/F13). The remaining C04, B06/B07, E05-E13, F02-F11
and full actual RAM/release signoff remain mandatory. A full-image KCC
build emits known identifier-truncation and unused-static diagnostic
notes; F10 has not been accepted. The production OOM flag remains
disabled, with no resident kernel/MRES code/data or ABI change.

## v1144: D14 combined real MM/provider acceptance

The 256K PDP-6 `daimos-testkit` integrated v1144 guest PASSes one
real physical pin/MM_BUSY/unpin/free each for four distinct owned
extents: PIPE, Type-340 DPY raw list, process VM and executive
u-area. The original fatal owner retains its real filesystem-provider
lock and sleeping waiter through FILE/HUP retries. A pre-commit
WAIT(NO-HANG) returns zero, and a publication witness checks that
PIPE/DPY/FILE release precedes VM destruction and VM/u-area release
precedes zombie publication. The final parent WAIT/CHLD is correct;
one swap detach, 48 stale WAIT/PID/event checks and independent
spinner progress PASS. All witness errors are zero.

The original separately rebuilt v1118 mixed PIPE/VM/FILE/HUP guest
remains PASS after this testkit extension. The v1087 transitive
release suite and its native regression/mutation controls also
PASS; an indefinitely in-flight foreign provider remains subject
to the existing fail-closed lifetime policy, not bounded completion.

D14 is CLOSED for this release scope. Mandatory OOM checklist:
**37/65 complete; 28 open** (26 pre-enable blockers plus deferred
F12/F13). D16, C04, sustained E/F and size/signoff remain pending.
Only the disposable test image contains the new hooks and counters;
there is no production kernel, MRES, ABI or OOM switch change.

## v1143: D15 completed by explicit unscheduled-XIP exclusion

For the current requester-only BRK OOM activation scope, ordinary
production EXEC and RUN use writable process startup, not MEMFS XIP
process admission. The current source-linked v1123 scope/mutant
verifier PASSes with six unsafe variants rejected. Current PDP-6
v1108 and v1109 MEMFS MRES XIP physical release/retry tests PASS,
including unsafe mutation rejection. The kernel supports XIP-owned
VM/provider release but not scheduled XIP process launch. D15 is
therefore CLOSED only for the present OOM scope, which EXCLUDES
scheduled XIP processes. The separate future trusted-XIP process
launch work X03-X08 is still required before DAIMOS release and
would require requalifying OOM if subsequently enabled.

Mandatory OOM checklist: **36/65 complete, 29 open** (27 pre-enable
blockers plus deferred F12/F13). D14/D16, C04, E/F remain open.
Production `PROC_OOM_BRK_ENABLED_V1000` is still zero; no production
kernel/resident/MRES/ABI change was made.

## v1142: physical MM_BUSY completes the D13 nested RUN subcase

The new full-system PDP-6 `daimos-testkit` target
`test-daimos-fatal-exec-physical-uarea-v1142` PASSes a combined
APR fault, EXEC postcommit FILE refusal/retry, and unpublished
post-VM RUN rollback. The test pins the child's actual allocated
u-area immediately before its real `mm_free`. The first free returns
MM_BUSY (-5); the original rollback defers the owner, then a checked
unpin permits a second real MM free in the slot-zero finalizer.
Two independently rebuilt guests each report exactly two frees,
one pin, one BUSY refusal, one unpin and zero errors, with parent recovery,
process-word/slot accounting, and the remaining EXEC child's
terminal WAIT/ZOMB all confirmed. No fake MM error replaces the
production allocator; all hooks live only in the disposable guest.

D13's physical unpublished-RUN requirement is now met after the
v1141 cross-object local relocation folding fix. Mandatory status
is **35 of 65 complete; 30 open**, of which 28 precede the later
activation/signoff actions F12/F13. D14-D16 and C/E/F remain open.
Production requester BRK OOM termination remains disabled.

## v1141: toolchain correction restores uninstrumented D13 subcase

The earlier v1130-v1140 D13 invalid-UUO diagnostics were generated by
an unsafe P10FOLD/DLINK cross-object local TEXT fold. Two equal numeric
LOCAL RH18 addends named different object-local destinations; the old
tools merged `proc_discard_ok` with unrelated syscall-dispatch text.
The corrected tools reject this fold and are tested by
`daimos-testkit` v1141 (both old-tool negative controls fail).

An independently rebuilt **uninstrumented** 256K PDP-6 nested guest
now PASSes the combined APR fatal, postcommit EXEC FILE refusals,
unpublished failed RUN rollback, parent continuation and WAIT;
`finish_zombie=1` and `error=0`. Historical full v890 EXEC/FILE/WAIT
and native v1129 VM_NONE regressions also PASS. No DAIMOS production
kernel source or OOM policy was modified. A separate v197 full guest
also PASSes 16 post-VM failed RUN mappings under three live children;
the nested uninstrumented v1126 guest PASSes a second fresh rebuild.
The native folding fix and
its relocation rules are documented in
`TOOLCHAIN/P10FOLD-LOCAL-RELOCATION-SAFETY-V1141.MD`.

This resolves the uninstrumented D13 return regression but does not
complete the required integrated *physical MM_BUSY* unpublished RUN
rollback or other D/E/F gates. Keep D13's mandatory checkbox open
until that evidence is accepted; requester OOM remains OFF.

## v1130-v1133: D13 remains blocked by nested syscall-return fault

After the v1129 full-VM_NONE destructor repair, the unchanged v890
EXEC/FILE/WAIT guest PASSes. Separately, v197 inherited-group and
v1130 NEW-group full PDP-6 guests each PASS sixteen real post-VM
fd-map RUN rollbacks with three blocked children and stable process
accounting. The nested v1126 APR+fatal-EXEC+failed-RUN guest still
stops before its parent R marker at a simulator UUO; the EXEC child
is not published as a ZOMB.

Disposable v1131 full rollback tracing changes this to `<EARH` and
one terminal ZOMB, v1132 unused-BSS-only padding does not, and v1133
minimal RUN claimed/return counters show the rollback entry and
return label were reached but do not restore parent R. These
diagnostics are NOT acceptance. D13 stays OPEN pending a proven
mapped-user syscall/parent continuation fix plus genuine MM_BUSY
unpublished-RUN rollback integration. The complete 29-item
pre-activation gate remains blocked; requester-only OOM stays OFF.

# DAIMOS PDP-6 requester-only OOM: activation gate v1123

Status: **NOT READY** (2026-11-04). This is the persistent OOM
acceptance specification, not an implementation handoff. The detailed
action-by-action checklist is the top `v1123` section of DAIMOS
`TODO.MD`. That section assigns unique IDs S01-S08, A01-A04,
B01-B07, C01-C04, D01-D16, E01-E13, F01-F13 and X01-X08.
At v1125, 34 of 65 mandatory items are complete. Of the
remaining 31, 29 block the *pre-activation decision*; F12 (enable
the production switch) and F13 (verify/document/commit that
enablement) are necessarily performed later.

## Exact requested policy

Only an already-running user's BRK request which remains impossible
due to **actual aggregate physical MM exhaustion** after the normal
allowed compaction/reclaim may terminate its own process. Production
`syscall_dispatch_pdp6.s` still defines
`PROC_OOM_BRK_ENABLED_V1000,0`; do not enable it on the strength
of isolated successes. BRK returns its existing user-level -1 and
the VM helper carries the actual internal cause in AC2. MM -2 is
fragmentation, -3 descriptor shortage, -4 invalid argument/type/
owner, and -5 busy/pinned ownership; none may be interpreted as
physical exhaustion. A failed child RUN is an unpublished admission
rollback, **not** grounds to terminate its parent. No cross-process
largest-victim selector is permitted.

## Gate disposition and proof required

| Gate | Status | Required release evidence |
| --- | --- | --- |
| S: Scope/classification | Partially closed | Freeze image/profile; verify exact MM code and opt-in victim selection; keep optional XIP off |
| A: Allocation-free release graph | CLOSED v1087 | Rerun graph and unsafe mutations against final exact code |
| B: Provider ownership | CLOSED for *fail-closed safety* v1092 | Preserve in-flight private stack/lease; do not require false bounded liveness; recheck under physical pressure |
| C: PI6/slot-0 finalizers | CLOSED v1086 | No sleeping acquisition or swap-in under PI6; owner retains potentially sleeping pre-VM work |
| D: Error-stage ownership | OPEN | Physical TTY/VM/u-area PASS v1124 and wrong-owner -4 with foreign FS lease PASS v1125; complete APR/EXEC, mixed locks/VM/FILE and exact-once multi-resource release |
| E: Full system | OPEN | Sustained reproducible real pressure, spinner fairness, parent WAIT/CHLD, PID recycling and negative scenarios |
| F: Activation | OPEN | Full matrix, actual resident RAM budget, signoff, only then production enable and repeat |
| X: Optional raw XIP process launch | EXCLUDED | Not wired into ordinary RUN/EXEC today; adding it requires separate opt-in acceptance |

### A/B/C: what closed actually means

`daimos-oom-transitive-release-v1087.py` checks the real bounded
allocation-free fatal graph across FILE/VFS SYNC, HUP/TTY, VM/XIP,
PIPE/DPY, backing bitmap, u-area and notification. It does **not**
guarantee total elapsed teardown time. The provider lifetime rules
accepted in v1092 retain a live in-flight DTFS/D6FS/TSFS lease and
its original owner; a returning error retains FILE/VM identity for
the next checked owner-side retry. An owner that never returns may
remain quarantined indefinitely; administrative repair or controlled
restart, not forced unlock, is the accepted response. Such a system
is safe against the known use-after-free paths but cannot promise
reclaim or progress from that specific dead provider.

PI6-masked slot-zero cleanup is non-sleeping (v1086), while HUP and
provider calls that can sleep remain on their owner-private stack.
PREVM/POSTVM markers retain the process identity, MM owner and
exit/WAIT status until the checked release truly commits. Never
replace those stages with a PI6-masked blocking callback.

### D: already demonstrated resource combinations

- v1129: The exact `VM_SPACE_NONE` (-1) sentinel is now recognized
  by the production PDP-6 VM destructor before any physical MM/XIP
  free, while checked backing detach remains active. Native source-
  linked positive and negative-mutant tests PASS; the previously
  timed-out *unmodified* full v890 post-commit EXEC guest now PASSes
  its real FILE retry and parent WAIT. This fixes a D13 prerequisite,
  but the combined nested APR/EXEC/failed RUN guest is still OPEN.

- v1101: physical wrong-owner -4 failure quarantine and bystander.
- v1104/v1105/v1106/v1107: independent physical PIPE, DPY, process VM
  and DISK bitmap release refusal/repair.
- v1108/v1109: source-linked MEMFS MRES XIP unpin and VM destroy;
  **not** a scheduled XIP process.
- v1118: real PIPE+VM MM_BUSY, FS lease, deferred u-area owner and
  independently progressing process with stale WAIT checks.
- v1119: PIPE+DPY on the same delivered session HUP; live PI6 replay.
- v1121: physical DPY MM_BUSY while one process genuinely swaps out
  and then restores; checked bitmap detach was **not** executed.
- v1122: first-member physical PIPE MM_BUSY and separate member's
  actual DISK bitmap corruption/refusal/restore during one session
  teardown, including exact WAIT/TTY and no leak. Standalone v1104
  and v1107 were rebuilt and rerun after harness composition.
- v1124: a real canonical TTY line pin rejects owner-side EXIT HUP,
  leaving the original session attached. A second user EXIT unpins
  TTY, then separate real process-VM and u-area MM_BUSY refusals
  are recovered by the qualified-clock slot-0 reaper with correct
  PREVM, VM_NONE, u-area owner and exact final commit. All three
  distinct busy/pin/unpin counters are 1, and the independently
  rebuilt legacy v969 TTY-only guest also PASSes after preflight
  trigger modernization. D11 was checked at v1124.
- v1125: a real u-area MM descriptor is given a wrong owner in the
  disposable kernel and all seven real `mm_free` retries return -4.
  Another process holds the actual serialized FS gateway lease
  `5,,0` throughout those MM refusals on its own private kernel
  continuation, with 80 sleep cycles. The lease is sampled directly
  inside the MM faulting path and again at HALT. The original faulted
  owner remains in MM teardown, an independent spinner advances, and
  the parent observes no premature WAIT completion or forced slot
  release. Independently rebuilt v1101 legacy guest and the current
  source-linked v1096 release matrix also PASS; the optional complete
  `P0D_FULL=1` boot matrix has not yet been rerun. D12 is checked.
  D13-D16 remain open.

Remaining D release cases must use actual production MM/FILE/TTY/
provider error returns on valid owned objects. Synthetic error codes
or test-only fake process descriptors do not demonstrate a physical
fault. Require exact owner, type, base, pin/reference, FILE/VM/
swap backing, scheduler and process-slot counters before refusal,
after refusal, and after successful finalization. A wrong-owner -4
must *remain* quarantined; do not force a success to make the guest
HALT. Include a physical TTY buffer fault, nested APR/EXEC fatal
entry, RUN rollback, and long-held provider contention in a booted
cross-product. All D items in DAIMOS TODO must be checked before the
normal-process-only activation decision.

### E: integrated stress acceptance

The existing 256K real-OOM v1075 guest proves requester-only fatal
cleanup against an unrelated blocked witness. The v1120 guest
further completes 24 sequential RUN/WAIT lifetimes with 23 adjacent
numeric PID reuses; v1118 checks 48 stale WAIT/PID requests and 37
spinner cycles during its held-owner interval. These are valuable
short-form proofs, not sustained liveness under repeated OOM.

Before declaring E complete, decide and document a *measurable*
stress horizon, run multiple true exhaustion/recovery cycles within
that horizon, and retain counter snapshots after every transition.
At least one independent runnable process must advance through
provider contention and slot-zero retry while qualified timer
interrupts continue. Combine a failed unpublished RUN with a
separately OOM-killed BRK requester and APR/EXEC fatal flow.
Maintain correct CHLD/WAIT, foreground TTY and PID/slot lifetime,
without free-after-use, unbounded runnable kernel loops, leaking
process words, backing sectors, MM descriptors or unexpectedly
killing another process. A reused numeric PID can validly address
a *new* process: test stale requests **between** lifetimes and
exact event attribution, not fictitious generation-number semantics.

Distinct negative cases are mandatory: invalid/out-of-range BRK,
address ceiling, fragmentation with enough total words but no
aligned fit, exhausted descriptor table, pinned MM_BUSY, storage
exhaustion without physical shortage, and failed child RUN. The
requester may only die for real MM_ERR_NOMEM. Rebuild both default-
off and isolated enabled variants on the same candidate revision.

### F: image, physical RAM and enablement

First freeze the supported boot profiles and root device/provider
set. The most recent unchanged KCC full-image KCORE low endpoint
reported after the v1100-v1122 tests is `023171` octal. This is
not a measured runtime peak, and the historical v513 full/auto
`permanent-size` package sum exceeded the `040000` octal image
budget. **Do not infer that all packaged MRES are resident at once**;
measure the actual live set, KCORE text/data, MM extents, provider
buffers, stacks, APR/alignment waste, and remaining 16K executive
address space. The strict `<020000` octal KCORE goal and full 32K
DSH command-boot optimization are already explicitly deferred;
do not silently use a 256K guest to claim either of them.

After a clean final source/negative native matrix and complete
`P0D_FULL=1` reboot matrix, rerun all v1075/v1110-v1116/v1120
default-disabled and disposable enabled physical-OOM variants.
Compare complete source and RAM/size reports with the last disabled
production baseline. Manual acceptance must explicitly record
provider quarantine limitations, supported media/profiles, testing
platforms and any excluded optional features. Only then change the
production switch from 0 to 1, rebuild, rerun the entire matrix,
and revert it on an unexplained regression. Commit and push only
the verified result with updated testkit STATUS, DAIMOS TODO/
PROJECT-STATE/KCORE and this documentation.

### X: optional MEMFS XIP must not expand this release accidentally

The private MEMFS MRES op `027` offers XIP admission; op `026`
provides release. Production `proc_run.s` calls `vm_space_startup`
once and `exec_lowmem_pdp6.s` calls it twice. Its `movem 3,(11)`
writes the user's startup record even when argc/envc are zero.
No ordinary RUN/EXEC loader calls the raw-XIP admission routine.
The v1123 source guard and six negative mutations check this
**current scope**, not the safety of a scheduled XIP process.

Therefore raw-XIP process creation remains OFF and may be excluded
from *current normal-process* OOM activation. Once root-only XIP
admission is introduced, this exemption ends: complete the separate
X03-X08 gate first (trusted non-writable image, strictly no-user-
write startup, swap/BRK guards, physical pins, provider lifetime,
scheduled RUN/EXEC/EXIT/WAIT, failure rollback, PDP-6 simulator
coverage) and re-run the applicable D/E/F OOM tests. Ordinary PURE
DXR files do not automatically satisfy that distinct raw-XIP gate.

## Fast review procedure

1. From `daimos-testkit`, run `make test-daimos-oom-scope-v1123` to
   check the default-off and optional-XIP source boundary. A PASS
   confirms only that limited contract; six mutations must fail.
2. Run the source-linked component matrix
   `make test-daimos-p0d-release-matrix-v1096` on current source.
   Before production enable, run with `P0D_FULL=1` to require all
   disposable guest modes and record their complete logs.
3. Complete every applicable unchecked S/B/C/D/E/F item in DAIMOS
   `TODO.MD` **through F11**, plus any relevant new code-path
   regressions. F12 and F13 occur only after signoff. Do not infer
   green readiness from a partial matrix or from this document.
4. Reconfirm `PROC_OOM_BRK_ENABLED_V1000,0` until the documented
   final signoff. The switch remains 0 while any *pre-activation*
   gate is open. Optional raw-XIP is explicitly excluded until separately
   admitted and qualified.
