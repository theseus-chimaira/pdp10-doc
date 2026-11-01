# v915 - Provider lock rejects same-owner recursive deadlock

The DTFS/D6FS/TSFS serialized provider gateway stores the owning
process slot plus one in the existing `fs_provider_ready` lock word's
left half. An attempted nested provider call by the same owner formerly
entered `proc_wait_event` and could sleep permanently awaiting its own
lease. v915 tests the encoded owner before entering that wait: the
nested invocation returns -5, restores the saved ACs and seven-word
request stack frame, and *does not release the caller's existing lease*.
Other processes still wait and resume after the owner unlocks. The
bypassed MEMFS provider is unaffected. Nine resident KCORE instructions
are added; no extra BSS, process metadata or ABI change.

Native PDP-6 v915 self-owner and independent-waiter assertions PASS,
alongside v866/v326/v327 owner/ABI regression tests. This is local
self-deadlock protection, not a solution for a provider stalled while
another process owns the lease. Full booted provider lock and safe
allocation-free OOM victim retirement still require acceptance.

Post-change full 256K PDP-6 v914 APR-fault/FATAL-FILE regression also
PASSes against the final nine-instruction v915 gateway: 11 permanent
provider errors, 11 retained FD checks, bounded 512-tick sleep retry,
zero errors and a runnable bystander until controlled test HALT. This
is an integration regression, not a booted nested-lock injection.

# v914 - Permanent invalid FILE-provider error retains its owner

A booted 256K PDP-6 extends the existing v891 APR-fatal FILE test to
one permanently refusing VFS provider returning -4 (invalid-provider
class), rather than -5 (busy). The copied-kernel injection applies only
to the faulting process's FD 0 VFS_SYNC call. Eleven consecutive rejects
leave that descriptor allocated, its private executive continuation
asleep, and its u-area error witness intact. GETTEARDOWN reports FILE
stage and the actual class-4 diagnostic. Eleven owner-stack retries
reach the 512-tick delay cap, zero exits/forced releases occur, and
another user remains runnable before the controlled test HALT.

This is further FILE error-class containment coverage without a kernel
change. MM invalid/missing extent repair, provider lock handling and
allocation-free OOM retirement remain separate unresolved issues.
Native v888, v909 and v885 regressions PASS; automatic OOM killing stays
OFF. No changes to production KCORE, RAM, MRES or ABI.

# v913 - Booted Type-340 confirms physical double-refusal ownership

A 256K booted PDP-6 with live PI6/PI7 executes raw display-list
replacement while a disposable test-only kernel pins the genuine
owner-014 MM extent for the old list and the newly allocated list.
Both release attempts reject with MM_BUSY. An additional WRITE_WORDS
and CLOSE also reject before any third extent can be allocated. The
observed old MM base 19760 decimal remains `dpy_list_base`, and the
new MM base 19792 remains `dpy_list_orphan_v912`. The guest counts four
refused frees, two pins, no unpins, 18 raw PI6 refreshes, 18 PI7 frame
completions (12 refreshes after first refusal), zero errors and normal
HALT. FILE/process slot counts remain stable across the rejection.

Native v912, v874 and v904 regressions PASS. This is the full-system
PI7/double-free acceptance missing after v912, not a claim that a
permanently pinned provider will eventually release RAM. All fault
hooks remain outside production in daimos-testkit. No new production
code, data, ABI or KCORE size; fatal FILE teardown and safe automatic
OOM victim retirement still require separate proof.

# v912 - Type-340 replacement preserves both owners on double MM refusal

The DPY native-program replacement can hold two MM owner-014 extents:
the published old list and the unpublished new allocation. If release
of both fails, the former driver executed HALT because the new extent
could not be tracked. v912 records the unpublished extent in one
optional DPY-MRES BSS word, `dpy_list_orphan_v912`, while retaining the
old published list and its PI6/PI7 replay state. Any subsequent DPY
WRITE_WORDS or STOP first retries release of this pending extent; if
MM refuses, the syscall returns an error without any new allocation
or old-list mutation. Once MM releases it, the pointer is cleared and
the requested operation proceeds. No forced free, main KCORE BSS or
syscall ABI change is introduced.

Native PDP-6 v912 tests both rejected frees, tracked owner, no third
allocation while blocked, release-and-retry success and original v874
replacement/STOP. The v871/v874/v904 and PI7 policy tests PASS. One
extra word of *optional DPY-MRES* BSS is required. Permanently pinned
owners remain allocated until their provider releases them. Full
booted double-fault/PI7 acceptance and fatal FILE/MM cleanup remain
open; automatic OOM termination remains disabled.

# v911 - SYS_MEMINFO FILE count survives the shared request buffer

The resident PDP-6 SYS_MEMINFO syscall uses the caller's nine-word
result structure as a temporary seven-word filesystem MRES request.
The initialization BLT for words 2..8 previously occurred *after* the
FILE descriptor scan had stored its count in result word 7; the BLT
therefore erased the count. This was unrelated to actual descriptor
allocation, which already used the entire FILE_NFILE=16 u-area table.
The SYS_MEMINFO scan and reported file_slots_total were additionally
limited to octal 015 (13), not octal 020 (16).

The production correction moves the unchanged FILE scan after MEMFS
request completion and uses octal 020 for both limits. The syscall
record layout, FILE lifecycle, process accounting, resident BSS and
instruction count are unchanged. On the 256K PDP-6, a C89 guest
repeatedly DUPs a live CTY descriptor, checks file_slots_used after
each operation through final FD 15, fills all 16 FILE slots, and
returns to the initial count by CLOSE. Guest <F, normal HALT: PASS.
The independent old-capacity negative fails at !1; the independent
old-ordering negative fails at !4. Both are real booted test images
with disposable source modifications in daimos-testkit v911.

The v910 permanent-DPY-refusal full-system test was rerun against
corrected SYS_MEMINFO and still PASSes: three real MM_BUSY returns,
one physical pin, no unpin, 18 raw PI6/PI7 frames, 12 after the first
refusal, zero errors, stable populated FILE/process counters. The
original v910 counter comparison did not independently establish
positive FILE occupancy before v911, because both old samples were
overwritten. Permanent provider refusal remains a retained owner;
this correction is not OOM or forced-release support.

# v910 - Permanent DPY MM_BUSY retains the same physical owner

The full-system 256K PDP-6 v910 acceptance uses a headless but present
Type-340 with live PI6/PI7, not a stubbed DPY driver. Its disposable
kernel hooks the ordinary MM free path, pins the exact owner-014 dynamic
raw-list extent (physical base 19732 decimal), and deliberately never
unpins it. Three successive FILE CLOSE requests all receive real
MM_BUSY (-5). `dpy_list_stop` restores the published raw-list base on
each rejected free; FILE_CLOSE retains the descriptor. FILE and process
slot counts measured after the original OPEN remain stable through all
three refusals. The raw program continues to refresh: 18 PI6 starts,
18 PI7 frame completions and 12 additional frames after the first busy.
There are three rejected frees, one pin, zero unpins and zero test
errors. The guest HALTs normally. v871/v874/v904 and PI7 policy tests
also PASS. No permanent kernel text, BSS or ABI is added.

The pinned extent deliberately remains in use at test HALT: v910 proves
failure containment and ownership preservation, not reclamation under
permanent refusal. Replacement rollback double faults, locked-provider
cleanup and allocation-free OOM process retirement remain unaccepted.
The OOM killer stays disabled, and new TSFS v1.2 work is post-KCORE.

# v909 - Permanent MM refusal has bounded idle retry and correct stage

A 256K booted PDP-6 test permanently pins the actual executive u-area
of a child terminated by the APR protection-fault path. No fake MM
return is used: the real allocator rejects every free of that extent.
Before v909, timer-driven scheduler admission enforced the shared
exponential delay, but an already-running slot-0 idle loop ignored it
and retried on every clock tick: 1,281 refused frees and 1,280 reaper
passes over 02400 octal observed ticks. The new idle-loop elapsed-time
guard uses the existing packed last-attempt epoch and capped interval,
with modular subtraction across timer wrap. No new persistent state.

The same guest also caught a stale fatal-FILE-close phase bit after
FILE closure and VM destruction. On the post-VM MM-deferred path the
kernel now clears only that expired bit, preserving TRANSITION,
VM_SPACE_NONE, FINALIZE and the pinned MM owner. `GETTEARDOWN` then
reports MM stage and 512-tick backoff; its detail query reports one
physical MM pin. Nonblocking WAIT cannot reap the victim, and a
second process continues to run. Corrected counters: seven failed
MM frees, six idle-reaper passes, one pin, zero unexpected errors.
Native v909 validates the FILE-to-MM phase transition and a pre-fix
negative control; v881/v885/v886/v888 regressions remain applicable.
This proves bounded safe retention, not eventual recovery from a
permanently failing provider. Automatic OOM killing remains disabled.

# v908 - Booted PDP-6 confirms DPY PI7 replay across MM_BUSY

The Type-340 raw display list's final CLOSE is now accepted with actual
physical MM pin/refusal/unpin/free on a running 256K PDP-6. The
instrumented kernel is disposable; normal driver `dpy_list_stop`
and FILE_CLOSE code are unchanged. The test confirms `DPY OK`, native
raw WRITE_WORDS, PI6 clock refresh, PI7 BLKO frame completion and
replay *after* the first MM_BUSY (-5). The failed CLOSE leaves the
original FILE descriptor and MM owner live, not leaked or discarded.
After physical unpin the same descriptor closes normally; another
program opens/writes/closes and process/FILE accounting is unchanged.
Observed MM base 19730 decimal, three tracked free attempts, one
pin/unpin, one busy, 18 raw refresh/PI7 completion cycles, six
refreshes after failure before unpin, zero errors, normal HALT.
Existing native v709/v871/v874/v904 and PI7 policy tests PASS.
This proves bounded recoverable MM_BUSY, not indefinite provider
refusal or safe automatic OOM termination. No production RAM change.

# v907 - Full-system physical u-area HUP partial-commit acceptance

The production v905 session-HUP retry is validated on a booted 256K
PDP-6 using the genuine MM extent table. A disposable test kernel
pins the first member's u-area before its fatal HUP. VM destruction
commits, but u-area free returns MM_BUSY and retains a stable
TRANSITION+VM_NONE+FINALIZE owner. A second member refuses its first
HUP; the controlling TTY remains attached. Parent-initiated PROCCTL
HUP retry skips the committed first child, delivers the second HUP,
and reaches the TTY-detach commit. The test unpins the exact extent
only after verifying the deferred owner. The parent waits for all
children and reattaches the TTY; guest HALT is normal.

Measured counters: first child slot 3, leader 2, u-area physical base
20020 decimal, one pin/unpin, one injected refusal, one later member
HUP delivery, zero duplicate first-member HUPs, one pending-owner
verification, zero errors. Kernel instrumentation is isolated in
daimos-testkit; the production syscall ABI and code are unchanged.
Persistent FILE/MM provider refusal and safe automatic OOM termination
still require separate recovery proof.

# v905 - Session HUP retries must skip only committed fatal exits

A session leader may deliver HUP to child A, whose VM destruction
commits but whose u-area MM free remains pending, then fail to HUP
child B. A retry formerly attempted another HUP on child A, which is
TRANSITION and rejected by `proc_event_apply`, indefinitely preventing
control-TTY release even when child B recovered.

The session scanner now tests three independent pieces of evidence:
TRANSITION in the process descriptor, VM_SPACE_NONE, and a finalize
marker in the still-owned u-area. Only the complete committed fatal
exit is skipped. A partially transitioning living member continues
to block teardown, rather than being silently bypassed. No provider
resources are forcibly discarded. Native PDP-6 positive and isolated
pre-v905 negative tests cover both members, retry, TTY ownership and
partial-marker cases. The older v875 regression also passes.
Full-system injected partial-provider failure remains to be tested;
automatic OOM termination remains disabled.

# v904 - DPY STOP quiesces hardware before unpublishing its raw list

The raw Type-340 STOP path previously cleared `dpy_list_base` before
`CONO 0130,000100` disabled the controller/PI7. An interrupt at that
point could classify the still-running raw BLKO completion as a
retained-text frame. The two existing operations are reordered:
first quiesce the controller, then clear the published base and
refresh state, and attempt MM release. A refused release still restores
ownership and retries through the existing FILE close path.
Native PDP-6 v904 executes real MM free/unpin with a pinned and then
wrong-owner descriptor; both reject close without losing ownership.
A corrected owner frees the exact extent; absent-list close succeeds.
The pre-v904 order fails an isolated source-order negative control.
v709/v871/v874 regressions PASS; live device-interrupt injection
remains pending. No added code instructions or permanent data.

# v896 - RUN startup cannot leave an unowned u-area MM extent

For a newly claimed SIDL RUN child, the PDP-6 constructor now records
the base and PROC_F_UAREA bit immediately after zeroing its successfully
allocated private u-area. `vm_space_startup` follows this publication.
If it rejects startup, `proc_slot_discard` sees the real MM owner and
can release it, or defer cleanup using the v892/v894 pending markers
if a provider refuses. This does not publish a runnable child, change
the three-word descriptor or add resident state. Ordinary startup
currently returns success; the full-system test injects a rejection
at the unpublished RUN call site rather than disrupting boot-time
INIT startup. The real MM extent probe returns MISSING for the exact
u-area base after rollback; a subsequent RUN/WAIT succeeds. An
isolated negative control restores the earlier publication ordering
and observes one leaked extent even though the child slot is FREE.
Full-system tests PASS; native deferred cleanup tests remain green.
There is no added permanent RAM or kernel code: same-profile GCC
KCORE 9,426, KCORE+MRES 15,912, endpoint 15,959 words. The remaining
provider-failure and automatic OOM-kill restrictions are unchanged.

# v895 - booted PDP-6 confirms real pre-VM RUN MM_BUSY recovery

The v894 unpublished-RUN VM retry now passes full-system 256K PDP-6
acceptance using actual MM allocator operations rather than mocked
VM failure return codes. After RUN loads a child process VM and fails
an invalid fourth FD mapping, a test-only copied-kernel hook pins
the exact child VM extent before `vm_space_destroy`. The normal
destructor consequently receives two genuine MM_ERR_BUSY (-5)
responses. The SIDL+TRANSITION and scheduler-bit-02 ownership marker
remain intact and readable through `GETTEARDOWN` stage 3.

The slot-zero reaper dispatches twice, the test unpins the real VM
before its third release attempt, and the unpublished child is
discarded without a zombie, session HUP or parent WAIT report.
An independent user process runs and exits normally. Guest SIMH
counters PASS: target 3, pins 1, VM release calls 3, busy 2,
unpins 1, reaper 2, errors 0. CTY `<RFBP` and normal HALT PASS.

Instrumentation and counters exist only in the disposable copied
kernel; production code, syscall ABI, resident data and image size
are unchanged. Last GCC lowmem disk-root KCORE remains 9,426 words.
Permanent provider refusal, complete allocation-free OOM policy
and the 8,192-word resident target remain outstanding.

# v894 - retry unpublished RUN VM release before u-area existence

Before v894, failed `vm_space_destroy` in `proc_slot_discard` left
an unpublished RUN child in SIDL, still owning its VM, without a
retryable finalizer. The v892 post-VM u-area marker was not usable
because the child might not yet have an allocated u-area.

The resident PDP-6 rollback now retains TRANSITION and marks
unused SIDL scheduler RH bit 1 (`PROC_SIDL_DISCARD_VM_PENDING=02`).
The existing slot-zero deferred-cleanup service recognizes this
specific combination and retries VM destruction without allocating
another stack or descriptor. Once the VM is actually destroyed,
the marker is removed; any subsequent failed u-area release uses
the independent v892 FINALIZE|DISCARD marker in word 067.
The pre-VM marker does not overlap SIDL PURE mount-extension bit 0
or live runnable-queue links. No forced MM unpin or provider bypass
is performed, and permanently failing owners remain protected.

`PROCCTL(GETTEARDOWN)` exposes read-only stage 3 RUN_VM plus the
existing shared MM retry interval even for a child with no u-area.
The previous NONE/FILE/MM stages and detail flag are unchanged.
Native PDP-6 tests verify two failed VM releases each for a no-uarea
and with-uarea child, then successful VM destruction, a separate
u-area MM refusal and eventual exact-once FREE without a zombie,
HUP or WAIT. The old v893 source fails the new negative control;
v881/v883/v885/v886/v887/v892 regressions PASS.

Identical GCC lowmem disk-root KCORE 9,381 -> 9,426 words (+45),
KCORE+MRES 15,867 -> 15,912, and permanent endpoint 15,914 ->
15,959. Static RAM, per-process descriptor and syscall slots are
unchanged. A full booted guest with real pinned process VM and
recovered pre-VM rollback remains to be accepted before OOM work.

# v893 - full-system real-MM recovery for unpublished RUN rollback

The v892 post-VM RUN discard retry now has booted PDP-6 acceptance,
not just an isolated native assembly fixture. A valid user-mode RUN
allocates a child VM and u-area, then fails its fourth file-descriptor
mapping. After production VM destruction, a temporary test-only
kernel hook pins the child's physical u-area, so the normal MM
allocator rejects the first two frees with genuine MM_ERR_BUSY (-5).
The existing slot-zero reaper retries until a test-only unpin enables
the third free. The unpublished child becomes FREE, never a zombie.

In the 256K full-system PDP-6 guest, an independent child runs and
exits normally; the parent observes no WAIT event for the rejected
RUN child. The test records target PID 3, one pin, three release
calls, two MM_ERR_BUSY, one unpin, two reaper dispatches, zero
errors, console `<RFBP` and normal HALT: PASS.

All instrumentation stays in a disposable copied-kernel build.
Production source, ABI, static allocation and KCORE size are
unchanged in v893; the same-profile v892 lowmem measurement remains
9,381 words. Permanent provider refusal, failed pre-VM rollback,
and automatic OOM process termination remain separate open gates.

# v892 - unpublished RUN rollback retries failed post-VM u-area release

In resident PDP-6 `proc_slot_discard`, a failed u-area `mm_free` after
successful VM teardown formerly left an uncommitted RUN child in
TRANSITION/VM_NONE without the marker required by slot-0 cleanup.
The allocation could stay owned indefinitely even after its provider
became able to release it.

The v892 fix uses two bits in existing u-area word 067: FINALIZE and
DISCARD (`0600000` octal in the left half). The existing deferred-MM
reaper distinguishes this unpublished child from the ordinary EXIT
FINALIZE marker and retries only its u-area release. On success it
publishes FREE, not a zombie, and does not notify a parent or repeat
VM/session/TTY teardown. Retry delays and scheduling rules are shared
with the v881-v885 service, without allocating any additional RAM.

Native PDP-6 fault injection PASS: first two releases reject, a third
succeeds, VM and TTY teardown occur once, and neither HUP nor WAIT
publication happens. The pre-v892 source FAILS the same fixture at
the missing marker; v881/v883/v885 regression tests remain PASS. The
full-system PDP-6 RUN v197 regression also PASSes 16 ordinary post-VM
fdmap rollback failures, but does not inject a rejected MM release.
Same-configuration GCC lowmem disk-root package link reports KCORE
9,351 -> 9,381 words (+30 code), KCORE+MRES 15,837 -> 15,867,
and permanent endpoint 15,884 -> 15,914; static RAM is unchanged.
Full-system injection of RUN u-area pinning is still pending.
Permanent provider refusal and safe automatic OOM termination are
not resolved by this change.

# v891 - indefinitely refusing FILE provider under scheduler load

A booted PDP-6 test now demonstrates the conservative fatal FILE
ownership policy when a provider *never* accepts close. The isolated
guest's VFS-sync hook returns -5 on every FD-0 close of a genuine
APR-faulted process. A separate user process continues running.
The victim retains its private kernel stack, FILE entry and recorded
real provider-error witness; the user-visible
`GETTEARDOWN(pid|0400)` call continues to show stage FILE, FD 0,
ERR 5. A saturated retry interval is not permission to discard
any of these resources.

Full-system native PDP-6 testkit v891 PASS after 02400 octal ticks:
11 refused VFS operations, 11 owner-stack retries and sleeps,
exponential backoff reaching the 01000 octal (512 decimal) tick
cap, 11 owned-descriptor checks, independent process progress,
no descriptor or word-067 witness loss, no victim EXIT and zero
test assertion failures. The test-only HALT is for inspection,
not a claim that the still-owned process was reaped.

The production kernel is unchanged at KCORE 022124 octal
(9,300 words, +1,108 above the 8,192-word goal); no additional
persistent RAM, API or recovery privilege is introduced. Permanent
provider failure remains a resource-lifetime problem needing
provider-specific diagnosis and repair. Malformed MM allocation
metadata, guaranteed recovery and automatic OOM victim termination
remain unresolved. Do not force release or automatically kill an
owner based solely on elapsed retries.

# v890 - full-system low-memory EXEC fatal recovery

v890 verifies that the existing fatal FILE owner-stack
continuation is also safe after an irreversible EXEC commit.
An actual user-mode `dsys_exec` submits a valid launch
record. A disposable PDP-6 kernel returns test-only
`EXEC_LOAD_NOMEM` at the staged-load boundary so that the
production `exec_replace_current_lowmem` fallback really
destroys the old VM, including normal swap detachment.
The guest confirms `VM_SPACE_NONE` and only then makes
the replacement loader return failure. The kernel propagates
`EXEC_REPLACE_FATAL` through `native_sys_exec`, without
restoring the destroyed user image.

The unchanged `proc_fatal_close_current_v879` owns all
remaining FILE descriptors and the original private
executive stack. The isolated VFS provider rejects two
FD-0 closes with -5. The process yields for 1 and 2
clock ticks while retaining its FILE descriptor and error
witness (the existing `GETTEARDOWN` ERR class is 5).
The third close succeeds; the descriptor and u-area
word-067 witness clear, and the parent receives exactly
one fatal EXEC exit status 1 through `WAIT`.

Full-system SIMH PASS: one staged NOMEM, one real VM
destroy, one failed replacement loader, three close
passes, two rejections, two sleeps, preserved ownership,
one final exit, correct `<EH`/HALT and no test errors.
Prior native fatal EXEC/provider/MM regressions PASS.
No production kernel source or persistent-memory changes:
KCORE `022124` octal = 9,300 words, 1,108 above the
8,192-word goal.

Both APR-fatal (v889) and low-memory EXEC-fatal (v890)
transient FILE refusal are accepted in full-system tests.
Indefinite provider refusal still has unbounded ownership
lifetime; automatic OOM killing remains disabled.

# v889 - real APR-fatal user with transient FILE-provider refusal

The private-stack fatal FILE retry policy is now exercised in
a full booted PDP-6 guest. The test program accesses a virtual
address outside its mapped APR range; the real CLK PI6 APR
fault handling redirects to `proc_fatal_close_current_v879`.
No synthetic EXEC fatal return is used. The guest's copied
test kernel intercepts `vfs_sync` for the faulting process's
FD 0 only and returns -5 twice before allowing ordinary
VFS synchronization. No production VFS or kernel changes.

For the rejected calls the descriptor remains owned and the
existing `GETTEARDOWN(pid|0400)` reports FILE stage 1,
error class 5. The owner-stack continuation yields for 1 and
then 2 clock ticks. A successful third VFS call clears
the descriptor and u-area word-067 witness before the
process exits. The parent receives the expected fault
termination through WAIT; subsequent user execution and
final HALT are normal.

Full-system SIMH counters PASS: one actual APR fault, three
VFS-sync calls/two -5 results, three fatal-close passes,
two clock sleeps, two retained-FD checks, one successful
release, one witness clear, one exit, zero unexpected errors.
The native fatal-EXEC/backoff and provider tests still PASS.
No source or data modifications in production KCORE:
`022124` octal = 9,300 resident words, unchanged from v888.
Full-system post-commit EXEC fatal injection and indefinite
provider refusal remain unverified recovery cases; OOM
victim killing remains disabled.

# v888 - capture the first real fatal FILE-close provider failure

On a fatal APR or post-commit EXEC private-stack continuation,
`file_close_all_at_v865` now captures the first genuinely
rejected descriptor and the return value from its actual
FILE provider. This is stored in the owner's previously
unused u-area word 067 while the FILE fatal-close bit is set.
LH holds a valid witness marker (octal 040) and FD 0..017;
RH stores the provider's signed 18-bit return. Retries replace
the witness. On successful closure of all entries it is
cleared, allowing the existing post-VM MM finalizer to reuse
word 067 for its own unrelated high-bit marker and exit status.
An ordinary/nonfatal bulk close never writes this record.

The `GETTEARDOWN(pid|0400)` detail response adds FILE
error classification in bits 15..17: 0 not recorded,
1..5 the real negative return -1..-5, 6 another nonzero
result. Existing stage, count, first descriptor and VFS/
PIPE/DPY fields are unchanged, as is the operation-041
UUO dispatch. `ps` prints the class in numeric `ERR`.
This reports the boundary error but not necessarily the
root cause inside a filesystem or device provider.

MM free still distinguishes -5 (physical pin) from -4
(no exact matching extent, wrong type or wrong owner);
v887's existing exact-extent snapshot offers current
owner/type/pin evidence without adding a second failure
record. A detail snapshot is not an atomic history of
the preceding MM operation.

Native PDP-6 tests PASS first actual provider error/FD,
changing errors on later close passes, retained descriptors,
successful clearing, and no nonfatal witness. Pre-v888
source FAILs the negative control as expected. Previous
fatal-close and teardown-query tests PASS. The separate
native `daimos-mm-release-cause-v888.sh` runs production
`mm_free` and `mm_unpin`: it proves exact -5 for physical
pins, -4 for missing/type/owner metadata errors without
discarding the extent, and successful free after unpin.
Full-system ordinary session HUP and TTY detach/reattach
PASS with the changed FILE runtime; actual injected fatal
FILE provider refusal in a booted guest remains untested.
KCORE
`022124` octal = 9,300 words (+26 over v887),
no new permanent RAM. Automatic OOM killing is disabled.

# v887 - on-demand FILE and MM owner detail

The v886 `GETTEARDOWN(pid)` query remains ABI-compatible.
For more specific diagnosis, set `SYS_TEARDOWN_DETAIL_FLAG`
(octal 0400) in the process PID argument. Only the low eight
bits select a PID; additional bits beyond the detail flag are
rejected. The query remains read-only and requires no new
permanent state or syscall selector.

When stage is fatal FILE-close, the detail result packs the
number of live entries in the process's existing 16-descriptor
table, the first remaining FD (020 means no descriptors) and
its provider class: VFS=0, PIPE=1 or DPY raw list=2. All
fields are derived from the stable u-area, not a global current
FILE pointer; no vnode, private address or kernel pointer leaves
the query. A remaining descriptor is not proof that its provider
is permanently unavailable.

When stage is deferred post-VM MM-release, the detail result
reports the actual pin count (0..255) of the exact u-area MM
extent, after checking type MM_TYPE_KERNEL_DYNAMIC and owner
01000+PID. The sentinels 0400 (no extent), 0401 (wrong type)
and 0402 (wrong owner) distinguish an invalid extent from
an ordinary pin. These values are returned shifted left two
bits and combined with stage 2. A pin count of zero can mean
the next cleanup attempt will succeed; it does not prove that
a previous busy result was wrong.

Native PDP-6 detail and v886 compatibility tests PASS.
The pre-v887 source FAILs the native negative control. A
full-system PDP-6 guest PASSes a live MM detail snapshot (one
physical pin), two actual MM_ERR_BUSY release failures, later
unpin and successful final reaping, an independently runnable
process and normal HUP/TTY/PIPE completion. The `ps` program
adds a numeric `DETAIL` column alongside TD and RETRY.
Production KCORE `022072` octal = 9,274 words, +72
resident words from v886, no new permanent data. The
interface provides diagnosis, not arbitrary forced resource
release. OOM termination remains disabled.

# v886 - read-only teardown-owner diagnostics

Persistent FILE and MM providers may refuse release indefinitely.
v885 reduces retry frequency but cannot safely prove that a live
provider handle or pin is abandoned. v886 therefore exposes
the **existing ownership state**, not a forced reclaim mechanism.

The already resident `procctl` UUO 077 gains sub-operation
`SYS_PROCCTL_GETTEARDOWN` (octal 041), which takes one process
slot/PID and returns a nonnegative diagnostic:

Sub-op 020 is already the UUO-077 PIPE extension; 041 is the
first available selector correctly forwarded to PROCCTL. This
preserves all preexisting extension operations.

- 0: normal process or completed zombie (no active teardown)
- 1: fatal APR/post-EXEC FILE-close owner, with private stack intact
- `(ticks << 2)|2`: deferred post-VM u-area MM release owner,
  with the current shared reaper retry interval in 60 Hz ticks.

Invalid and FREE user slots return -1. The MM state is recognized
only when TRANSITION, VM_SPACE_NONE, a stable u-area and its
word-067 finalize marker all agree. FILE ownership is recognized
by the existing `PROC_FATAL_CLOSE_BIT`. The low two result
bits form the stage; the upper bits provide the **shared** MM
backoff (16..512), not a per-process provider-failure count.
The FILE stack's current private delay is not exported. Neither
query exposes raw MM/FILE pointers, kernel addresses or user data.

`ps` reports `TD` and `RETRY` numeric columns using this
query. An operator can locate the stuck process and investigate
the provider's I/O completion, reference ownership or MM pin;
if the provider can legitimately release the reference, normal
background cleanup finishes the process. There is no unsafe
forced-close or arbitrary MM unpin operation in this interface.

Native PDP-6 query tests PASS for normal, FILE, deferred MM,
invalid/FREE, stale-marker and zombie cases. PS target C
compilation and KCORE link PASS. Resident KCORE ends at
`021762` octal (9,202 words; +37 from v885, including four
packed dispatch-table words), with no
permanent-data growth. Automated OOM termination remains disabled.
The full-system native-UUO query also PASSes: ordinary and invalid
PID results, exact MM-pending state during two real MM_BUSY
release failures, then normal finalizer and session HUP/TTY
completion while a separate user keeps running. The early
op-020 experiment failed because it collided with PIPE creation;
the corrected op 041 preserves PIPE and other older ABI calls.

# v885 - resource-preserving capped teardown retry intervals

In PDP-6 fatal APR and post-commit EXEC teardown, a FILE provider
can reject close without releasing its handle. The process must
retain its private executive continuation and original exit status
until every FILE descriptor is actually closed. v885 stores a
retry delay in one word on the existing private kernel stack and
backs off unsuccessful attempts from 1 tick up to 01000 octal
(512) ticks, or about 8.5 seconds at 60 Hz. Repeated rejection
never authorizes kernel-forced release.

The deferred post-VM u-area finalizer similarly treats a failed
MM free as resource ownership, not as successful reclamation.
The existing single resident word `proc_finish_last_tick_v881`
now combines pending (high LH bit), a capped 020..01000-octal
tick shared retry delay (remaining LH), and the last service tick
(low 17 RH bits). Reaper service increases the interval after
continued rejection and clears it once no owner remains pending.
A newly deferred owner resets the interval for prompt first
retry; an older pending owner does not reset it each pass.
Normal runnable-user and RT/swap priority rules remain those
accepted in v884.

Native PDP-6 FILE cap, MM shared interval and ownership/retry
tests PASS. The unchanged pre-v885 FILE-close code fails the
negative-control acceptance. The full-system PDP-6 session HUP
test with a continuously runnable companion, real MM pin and
two MM_ERR_BUSY results before eventual successful release still
PASSes. KCORE exclusive end `021715` octal: 9,165 words,
30 above v884, no new static data word.

Persistent provider failure can retain a process slot, FILE
handle or MM extent indefinitely despite the reduced retry CPU
overhead. Releasing such resources by timeout would violate the
ownership invariant. There is no automatic forced reclaim and
OOM victim termination remains disabled.

# v884 - deferred u-area reaper scheduling under sustained load

The v881-v883 deferred finalizer originally ran only while slot 0
was idle. Another permanently runnable user could prevent that idle
loop from executing, so even a transient MM pin could leave an
already-dead process's u-area quarantined indefinitely.

v884 repurposes the high bit of the existing resident
`proc_finish_last_tick_v881` word as a pending-finalization flag;
the low 17 bits retain the last MM service tick. A failed u-area
free arms this flag. The slot-0 reaper clears it before processing,
and any still-failing finalizer re-arms it. No new data word or
process-table field is required.

At a normal PI6 quantum, pending deferred cleanup overdue by
020 octal ticks is granted one slot-0 service turn, even when an
ordinary process remains runnable. The service finishes by
requesting ordinary scheduling immediately, so it does not hold
the CPU for a further clock quantum. Runnable real-time ownership
and outstanding disk-swap service retain priority. A sleeping or
stopped RT lease does not prevent maintenance. Time comparison
uses the 17-bit modular clock, handling epoch wrap. Without
pending cleanup, slot 0 skips MM scanning.

Native PDP-6 maintenance throttle, runnable-user scheduling,
RT/disk priority, dormant-RT handling and timer-wrap fixtures
PASS. Pre-v884 scheduler code FAILs the new negative control as
expected. Full-system PDP-6 MM pin/unpin fault injection with a
separate continuously spinning user PASSes; its guest records two
reaper turns while the run queue is nonempty, two actual MM_ERR_BUSY
results followed by a successful release, and normal session
HUP/TTY completion. Production KCORE links at `021657` octal
(9,135 words), +39 resident text over v883, no new static data.

This proves normal-user fairness, not a hard guarantee under an
indefinitely running real-time owner or permanently refusing MM
and FILE providers. Automatic OOM termination remains disabled.

# v883 - non-current fatal HUP commit versus deferred MM release

After an external fatal event closes a process's FILE entries,
VM destruction is irreversible. If MM cannot immediately release
the now-dead process's stable u-area, v881 retains it quarantined
under TRANSITION, VM_SPACE_NONE and the per-u-area word-067 marker.
Before v883 this committed delivery still returned -1 to a
session leader: the HUP appeared to fail, but re-delivery would
correctly be refused because the target was already transitioning.

`proc_event_apply` now returns success for this precisely
identified post-VM pending state. Pre-VM or failed FILE-close
operations continue to return an error, and duplicate fatal
events are rejected. Slot-0 idle reaping owns the eventual
MM release, parent notification and zombie publication.
The C policy fallback matches the native PDP-6 branch.

The native PDP-6 integrated event/finalizer test PASSes for
precommit VM-destroy rejection, committed delivery, retry,
duplicate-event exclusion and exactly
one parent notification. The pre-v883 negative control FAILs
as expected. KCORE link PASS: `021610` octal (9,096 words),
+17 resident words over v882, no permanent-data increase.
Full-system non-current MM pin acceptance also PASSes using the
testkit-only `daimos-noncurrent-uarea-pin-v883.sh`: target
slot 2, 3 release attempts, two real MM_ERR_BUSY responses,
one pin/unpin each, zero errors and normal session HUP/TTY HALT.
The v882 current-EXIT injection still PASSes. Continuously
runnable companion fairness and permanent-MM-failure policy
remain separate acceptance gates.

# v882 - full-system pinned u-area finalization validated (2026-10-31)

An isolated, test-only PDP-6 boot image exercises the production
`proc_exit_finish`, `proc_uarea_release`, `mm_pin`, `mm_free`
and `mm_unpin` with an actual pinned stable u-area. The harness
pins the first process executing its own EXIT, releases its pin
on the third attempted free, and inspects guest MM counters after
HALT. The selected exiting slot varies with scheduling (1/2
across runs); it observes three release attempts, two actual
`MM_ERR_BUSY` returns, one pin and unpin each, and no hook errors.
The original session HUP and TTY detach/reattach integration also
finishes normally. This confirms the v881 idle finalizer can
recover from transient MM pinning in a booted guest.

The harness changes only an ephemeral kernel source copy under
`$TMPDIR`. Production KCORE remains `021567` octal (9,079
words); no permanent data or code was introduced for the test.
The non-current fatal-termination path can return a transient
failure to its caller, and permanent MM failure or sustained-load
reaper fairness remain unverified. OOM killing stays disabled.

# v881 - deferred MM release after VM teardown (2026-10-31)

Process EXIT switches away from the target's private kernel stack
before destroying VM and its final stable u-area. If MM rejects
the last u-area free after VM_SPACE_NONE is published, an immediate
retry from the shared permanent stack must not block other runnable
processes. Before v881 this case returned failure and HALTed EXIT.

The finalizer now leaves TRANSITION and the u-area owner intact, and
records a pending marker with the original low-18-bit exit status
in the preallocated unused u-area word 067. The session/domain/event
control word is not changed (its bit 010 is part of session ID, not
a spare flag). Slot 0 scans for these marked VM_NONE processes
at most once per timer tick, with scheduler PI6 inhibited during
shared MM and process-table updates. A successful retry follows
only the last release, run-queue, adoption and zombie/free commit
stages, without repeating session HUP or VM destruction.

The C fallback validates the same marker, TRANSITION and VM_NONE.
`proc_exit_finish` now returns to idle when this specific deferred
owner exists, and continues reporting other failed finalizations.
One permanent last-retry-tick word is added, with no change in
process-descriptor or u-area size. Native fault-injected finalizer
and compiled KCC EXIT handoff tests PASS; older FILE/VM tests PASS.
KCORE links at `021567` octal (9,079 words, +100 from v880).

The idle-only scan is neither a progress guarantee for permanently
failing MM nor a full-system injected-MM acceptance test. OOM victim
killing remains disabled pending PI6 and failure recovery proof.
The ordinary headless full-system session HUP and TTY lifecycle
regression PASSes with v881. It does not inject a failed MM free.

# v880 - error-preserving DISK swap detach and VM ownership (2026-10-31)

Before v880, `proc_swap_detach` returned void and discarded a disk
process's swap record even if `backstore_free` silently rejected
the recorded block range. `vm_space_destroy` then published
VM_SPACE_NONE, severing the owner from unreleased backing storage.

The detach operation now validates every block of the allocation
before releasing the backing bitmap and modifying the process swap
record or counters. If the block range, occupancy, or process block
counter is invalid, it returns -1 without publishing any changes.
`vm_space_destroy` checks this error and retains the previous VM
state, making ownership retryable instead of silently lost.

Validation reuses the existing read-only bitmap verifier in SWAP MRES
via a new export `backstore_mres_verify_allocated_v880` at index 6.
MINIT patches a single resident entry, which fails safely when
the module is unavailable. There is no duplicate verifier and no
new permanent bitmap or descriptor table. The SWAP MRES link
and its new export PASS.

Native VM destroy error/retry and real C detach bitmap fault
injection tests PASS; pre-v880 VM error propagation FAILs the
negative control as expected. Existing backing verifier and range
checks PASS. KCORE link PASS at `021423` octal (8,979 words),
16 words over v879 `021403`. Errors after VM destruction,
especially u-area MM release from the permanent slot-0 stack,
remain unresolved; this change does not enable OOM killing.
The post-v880 full-system headless session teardown regression PASSes
with the updated SWAP MRES installed. It does not inject an invalid
backing bitmap during actual guest execution.

# v879 - owner-stack fatal cleanup and VM-less executive retry (2026-10-31)

User-origin APR faults and post-commit fatal EXEC now enter the common
resident `proc_fatal_close_current_v879` continuation *before* moving
onto the permanent slot-0 stack. The process's exit status remains on
its stable private executive stack while `file_close_all` runs.
Rejected closes retain their FILE entries; the continuation sleeps
one clock tick with `proc_sleep_ticks`, then retries when scheduled.
Successful closure alone permits irreversible `proc_exit_current`.
This does not allocate new heap, swap, or permanent descriptor storage.

The previously spare low bit 2 in the per-u-area control word now
marks an active fatal-close continuation. A second event (including
STOP/CONT) cannot destroy or suspend its sleeping executive stack.
Already-transitioning and u-area-less targets are rejected by event
delivery. OOM and ordinary swap-out selection exclude such a process.
The C event/swap/scheduler fallbacks implement the corresponding
policy.

After post-commit EXEC failure there may be no valid user VM at all.
A tagged fatal-close kernel continuation may nevertheless be selected
and resumed without a swap-in request, including when owned by a
real-time process. Executive-context restore skips APR activation
for a tagged VM_SPACE_NONE (-1) or zero-base state. The native
scheduler now distinguishes VM_SPACE_NONE from actual resident base
and genuine zero-base swapped processes; untagged destroyed processes
are excluded even with an RT lease or ready-once selection. Generic
`VM_SPACE_ACTIVE` and the C scheduler fallback also exclude
VM_SPACE_NONE instead of treating its all-ones RH as a live VM.
Direct native swap-out and exchange-pair preflight now reject the
fatal-close flag, preventing MM from evicting that private stack.

Native PDP-6 v879 tests check repeated failed closes, timer yields,
descriptor ownership, duplicate-event exclusion, status retention,
successful exit, VM-less selection, RT dispatch and APR-safe restore:
PASS. Updated EXEC/FILE/EXIT/session/RT/OOM selector native suites
PASS. The native swap guard test covers flagged owners, mapping hold,
transition and normal admission: PASS. Pre-v879 swap-out FAILs the
negative-control fixture as expected. Lowmem KCORE link PASS,
exclusive end `021403` octal (8,963 words), 108 words above v878 and
771 words above the strict 020000
octal resident target; no new persistent data words.

Final-source full-system ordinary session-leader HUP and TTY
detach/reattach PASS in a headless simulator. This does not include a
rejected provider close, PI6 continuation replay under fault, or
bounded fatal cleanup recovery.

This does not guarantee the overall FILE release eventually succeeds
or that a blocking provider returns. Full-system PI6/timer retry
under actual failing I/O, errors after VM destruction and automatic
OOM termination remain unaccepted; OOM killing stays disabled.

# v878 - post-commit fatal EXEC closes FILE descriptors (2026-10-31)

The PDP-6 low-core EXEC replacement can report
`EXEC_REPLACE_FATAL` after the old user VM has been removed. The
`native_sys_exec` handler formerly jumped immediately to
`proc_exit_current`, bypassing `file_close_all`. The finalizer's
required live-FILE guard then rejected the process even when its
descriptors could have been closed normally.

The post-commit fatal branch now closes the stable process-private
FILE table before entering nonreturning slot-0 EXIT. A rejected close
retains the descriptor for ownership accounting; the finalizer must
not release that u-area. Since the old VM is gone, a persistent
close failure cannot safely return to EXEC and still requires a
deferred cleanup design. This change adds no permanent memory.

Native PDP-6 v878 tests fatal EXEC with successful and rejected
FILE close, normal EXEC, pre-commit failure and failed mapping:
PASS. The pre-v878 negative control FAILs as expected. Earlier
FILE, EXIT, session and finalizer regressions PASS. KCORE links at
exclusive end `021227` octal (+1 word against v877). Full-system
post-commit fault injection and automatic OOM termination remain
unverified.

# v877 - post-v876 full-system session HUP PASS (2026-10-31)

The ordinary full-system `daimos-session-teardown-v1.sh` guest PASSes
with v875/v876 source: session-leader exit HUPs running and stopped
jobs, detaches the controlling TTY, and allows reattachment. The
test uses headless Type 340 and disables unrelated graphical
WCNSLS/OCNSLS devices; the unisolated simulator had previously
crashed in SDL before INIT with exit 139. This does not alter kernel
semantics or hide the session workload. Native failed-HUP and failed
VM-destroy probes remain separate passing tests. Neither provider
failure injection inside the full-system guest nor nonreturning
APR/EXEC/OOM termination has passed. No production code or size
change in v877; KCORE endpoint remains `021226` octal.

# v876 - RT ownership survives failed HUP and VM cleanup (2026-10-31)

Previously `proc_finish_slot` revoked real-time ownership before
session-HUP callbacks and only marked the target TRANSITION afterward.
Because a member's FILE/provider cleanup can sleep or fail, the target
could be selected for execution during its own teardown, or retain a
live descriptor/u-area without its original RT owner lease.

The PDP-6 resident finalizer and C fallback now reject an already
TRANSITIONing target, mark TRANSITION after successful FILE preflight
but before HUP, and restore that state on failed HUP or failed VM
destruction. RT ownership is revoked only when VM destruction succeeds.
There are no new persistent data words. Later release errors *after*
VM removal remain nonrecoverable through normal user scheduling.

Native v876 checks the transition preflight, RT ownership through
failed HUP and failed VM destroy, retry and eventual zombie commit:
PASS. The same test FAILs against pre-v876 source. A lowmem KCORE
link PASSes at exclusive end `021226` octal, four words above v875.
The full-system session-teardown regression PASSed after v875 and
again on post-v876 source using CTY with graphical backends disabled.
Automatic OOM killing is still disabled.

# v875 - session HUP teardown retains TTY on failure (2026-10-31)

The session leader formerly detached its controlling TTY before
delivering fatal HUP to members. When a member's FILE close failed,
the leader's cleanup aborted but the TTY was already detached, leaving
a partially terminated session without its original TTY association.

The resident PDP-6 `proc_session_teardown` now waits until all member
HUP calls succeed before clearing the TTY state. The old output route
remains preserved. A failed HUP keeps the original record intact, and
subsequent cleanup may retry. One additional saved AC holds the TTY
index across member event callbacks; no permanent data is allocated.
This does not undo member HUPs already committed before a failure.

Native v875 fault-injection tests a failed second HUP, original TTY
retention, retry, nonleader and detached cases: PASS; the same test
FAILs against pre-v875 source. Lowmem KCORE link PASS at `021222`
octal, four words above v874. Full-system scheduler/session acceptance
and nonreturning APR/EXEC fatal-exit resource safety remain open.
OOM victim termination is not enabled.

# v874 - Type 340 display-list replacement is ownership-atomic (2026-10-31)

Before v874, the optional DPY driver published its new list and IOWD
before attempting `mm_free` on the old list. A failed old release was
ignored and its allocated extent lost its sole published owner.

The corrected replacement prepares the new image and IOWD privately,
disables PI7 while releasing the old extent, and publishes the new base
and IOWD only after release succeeds. If old release fails, the old
list/IOWD remain published; the newly allocated extent is freed and
the old list is scheduled for clock-driven refresh. The caller receives
-1 and the accepted-write counter is not incremented. This needs no
extra permanent KCORE or MRES data words. If rollback of the new
allocation *also* fails, an explicit MM invariant HALT prevents a
silent unowned allocation; that double-failure case remains untested.

Native PDP-6 v874 checks failed release, rollback, retry, successful
replacement, allocation failure, initial installation, STOP, IOWD
preservation and write-count publication: PASS. The same regression
FAILs against pre-v870 source, proving it detects the old bug. v709,
v871, v870 and Type 340 PI policy also PASS. Direct DPY MRES link and
lowmem KCORE link PASS; the resident end is unchanged at `021216`
octal. Actual Type 340 replay after a forced release failure and
system-wide OOM kill acceptance remain outstanding.

# v873 - native PIPE rollback versus full-system FIFO acceptance (2026-10-31)

The v870 native PIPE final-MM-free fixture fails on pre-v870 v869
source and passes after the repair. The separate full-system KCC
named-FIFO regression still times out (exit 124) after D6FS startup;
the same timeout occurs against isolated v869 commit `8b4e7d9`,
so it predates the new close-ordering code. The cause and any possible
additional regressions are not established by this comparison. Keep
full-system FIFO acceptance open, independently of native v870 PASS.

# v871 - Type 340 final-close ownership survives failed MM free (2026-10-31)

The native FILE DPY close previously ignored the raw-list stop result:
`vfs_sync` returned zero for this device even if its dynamic-list extent
could not be freed. The DPY MRES also discarded the active list base
before checking `mm_free`. The corrected close now propagates the real
stop failure so FILE retains the descriptor. With no DPY module loaded,
close is a successful no-op; a write still returns an error.

The DPY MRES disables PI7 while attempting the free. If MM rejects it,
the same extent base and original IOWD are restored and the 30-Hz
refresh is scheduled again, permitting a subsequent close to retry.
On success, the IOWD is cleared. Native v871 fault injection and the
updated v709 direct-gateway tests PASS; direct DPY assembly and lowmem
KCORE link PASS. KCORE exclusive end is `021216` octal, one word below
v870. No additional permanent data is allocated.

Failure while freeing the old list during *replacement* is still an
unresolved resource-lifetime issue. Actual hardware replay following
an injected failure, nonreturning APR/EXEC fatal exits and bounded OOM
reclaim progress also remain unverified. Automatic OOM kill is OFF.

# v870 - last PIPE/FIFO release is retry-safe (2026-10-31)

The PDP-6 `pipe_close_ref` previously published zero reader/writer
references before freeing the final MM allocation. If the free failed,
the FILE layer correctly retained the descriptor, but its next close
would observe a pipe with zero references and could not undo the
partial teardown. The final reference is now freed before publication.
If `mm_free` rejects an active named FIFO, the code relinks its existing
object at the FIFO list head without new storage; reference counts and
the descriptor remain unchanged. Non-final closes continue to publish
updated counts and wake waiting peers as before.

Native PDP-6 v870 exercises failed MM release and retry through the real
FILE and PIPE close code, for both anonymous and named pipes, and tests
ordinary non-final decrement: PASS. KCORE lowmem link PASS, exclusive
end `021217` octal, six words above v869. This resolves one teardown
atomicity bug; it is not an enabled OOM killer. Forced-fatal exits,
provider waits and other resource destructors still require coverage.

# v868-v869 - current EXIT and final teardown FILE guard (2026-10-31)

The native PDP-6 EXIT syscall and current-process fatal-event handler
now respect the aggregate `file_close_all` return. If any VFS sync fails,
that descriptor remains open and the process returns an error rather
than freeing its u-area. The matching C reference fatal-event path
checks close failure too. Native v868 exercises failed first-close,
successful second-close, retry, and preservation of exit status: PASS.

The resident `proc_finish_slot` now rejects a missing stable u-area or
any of its 16 still-open two-word FILE entries *before* modifying RT
ownership, session membership, VM state or run-queue state. The C
reference includes the same preflight. Native v869 confirms that even
the last live descriptor blocks all teardown helper calls, then verifies
normal zombie publication after the entry is cleared: PASS. This adds
no permanent metadata or heap allocations. Lowmem KCORE links at
exclusive end `021211` octal (18 words above v867 `021167`).

This guard prevents orphaned descriptors but does not ensure progress on
unrecoverable fatal exits. APR faults and post-commit EXEC errors cannot
return to the faulting userspace process; provider waits, recursive HUP
and failed teardown still need allocation-free bounded resolution.
OOM termination remains disabled, and full-system OOM acceptance has
not been established.

# v867 - fatal-other FILE close failure keeps victim resources (2026-10-31)

The native PDP-6 `file_close_all_at_v865` now returns a failure result
after visiting all 16 fixed file descriptors. An FD whose `vfs_sync`
fails remains open, so `proc_event_apply` must not finalize that target
and free its u-area. It now returns failure instead. Native v867 fault
injection confirms the failed descriptor survives, another descriptor
closes, and finalization succeeds on retry after provider recovery.
The ordinary v865 target-file isolation test also passes.

This consumes no additional permanent KCORE data. It is a cleanup
correctness prerequisite, not enabled OOM killing or guaranteed resource
progress: current-process EXIT, recursive session teardown, permanently
failed sync, and held provider locks still need bounded OOM-safe handling.

# v670 — swap pressure test still blocked before scheduling

Testkit v670 confirms that the 256K pressure image stops in KINIT
D6FS pre-VFS mounting after BLOCKSET reads and superblock selection
entry. This is not an MM rollback result. Keep isolated native v664-v668
successes distinct from still-open full-system scheduler acceptance.

# v669 follow-up — halt classification

The full-system swap pressure test halts in KINIT root mount/INIT
validation (`?RT`), prior to scheduler or swap transaction execution.
Full-system scheduler rollback has therefore not been exercised.

# v669 — scheduler-level swap fault acceptance status

The native branch-level rollback tests remain valid, but scheduler
integration is not yet established. An updated full-system regression
no longer assumes fixed KCORE addresses for the loadable swap module;
the current pressure guest halts with `?RT` at `065206` before its
expected completion markers. This is an unresolved test/startup
blocker, not evidence of successful or failed swap recovery.

# v668 — late MM-free failure native regression

Native PDP-6 execution now verifies recovery from a failed `mm_free`
after successful swap-out unpin. The provisional BACKING span is
released while original resident VM ownership and swap record are
retained; transition and provisional u-area state are cleared.
An unexpected `mm_unpin` failure remains fatal by v667 design.
Full-system scheduler fault injection remains outstanding.

# v667 — failed unpin is an invariant violation

Contrary to the broader v666 rollback wording, an unexpected failed
`mm_unpin` cannot safely return to normal scheduling: the physical pin
state may be inconsistent. Both swap-out unpin sites now fail closed
through the existing storage invariant handler. In contrast, an
ordinary failed `mm_free` after successful unpin still rolls back the
provisional BACKING allocation before publishing DISK state.
Isolated native PDP-6 branch failure tests PASS. Full-system MM fault
injection and scheduler acceptance remain open.

# v666 — late swap memory release (2026-10-29)

Swap-out keeps DISK state unpublished until both `mm_unpin` and
`mm_free` succeed. The error branch releases provisional BACKING and
clears TRANSITION while retaining the resident extent. Testkit v666
checks the source instruction ordering. Real native MM error injection
is still necessary to establish the complete runtime guarantee.

# v665 — process/MEMFS BACKING failure observation (2026-10-29)

Native PDP-6 BACKING I/O returns failure on the first failed block
without issuing later writes. Process swap-out and MEMFS eviction
keep newly written BACKING extents unpublished until the write succeeds
and release allocations on write failure (testkit v665). This does not
guarantee multi-block physical atomicity or complete late-MM rollback.

V513 SIZE UPDATE (2026-10-27): FRESH CURRENT GCC PERMANENT-SIZE RESULTS,
NOT THE OLD V219 KCORE SNAPSHOT. 32K LOWMEM TAPE: KCORE `020544` (8548
WORDS), PACKAGED KCORE+MRES 13463, PERMANENT LAST 13510, 2873 WORDS FREE
BELOW `040000`. 32K LOWMEM DISK: SAME KCORE, PACKAGED 13539, LAST 13586,
2797 FREE. FULL/AUTO GCC: PACKAGED 18271, SINGLE LAST 18318 (1934 OVER
`040000`), MULTI LAST 18351 (1967 OVER); THAT FULL PACKAGE SIZE CHECK
**FAILS**. PACKAGED MRES SIZE IS NOT A RUNTIME RESIDENT HIGH-WATER MARK.
CONSOLIDATED REMAINING XIP RUN/EXEC WORK AND THE DEFERRED 512-WORD
GRANULARITY INVESTIGATION ARE IN DAIMOS `TODO.MD`. THE PDP-6 APR SETS
RELOCATION/PROTECTION AT OCTAL `02000` (1024 DECIMAL) WORDS, NOT 512;
ANY FINER SOFTWARE ALLOCATION OR FUTURE BACKEND GRANULARITY MUST NOT BE
MISTAKEN FOR HARDWARE-SUPPORTED PDP-6 USER MAPPING GRANULARITY.

### Shared BACKSTORE allocator in pinned MRES (v484, 2026-10-27)

The process swap service and the common process/MEMFS backing allocator
share the pinned SWAP MRES. Four public backing-store calls remain as
single-word patchable KCORE entry points; the four backing-state words
remain authoritative and resident. There is no second package/bitmap.
The KCORE end marker is deliberately linked after these state words;
the MM descriptor table starts exactly at `__kcore_low_end`. GCC PDP-6
lowmem disk KCORE is 8548 words and combined KCORE+MRES is 13415 words
after relocation, 104 fewer KCORE words with no growth in combined RAM.
The 32K native swap-wait and swap-stress tests pass. Dynamic swap-record,
bitmap and storage-mode boot allocation remains unchanged.

### Process swap transfer service in boot MRES (v481, 2026-10-27)

The permanent PDP-6 KCORE retains `proc_swap_reclaim` and
`proc_swap_service_one` as one-word patchable jump entries. The SWAP MINIT
installs a pinned `swapv481` MRES and retargets these entries to its two
exported methods. Calls before publication return -1 through the existing
resident error leaf. Memory allocation retry policy, VM growth retry, and
the idle-loop swap request check remain in KCORE. Swap-in/out transactions,
reclaim orchestration, and PURE prefix omission execute in MRES. The PURE
backing-descriptor decode and packed per-process storage-state functions
remain resident because they have independent VM/lifecycle consumers.

GCC lowmem linked disk-root measurement: KCORE 8960 -> 8652 words; MRES
+308 words; permanent KCORE+MRES 13415 -> 13415 words. A tape-root link
gives 8652 KCORE and 13339 combined, likewise unchanged overall. Native
32K swap-wait and swap-stress and PDP-6 PURE reconstruction/verifier tests
pass. The SWAP MRES source and MINIT entries must appear in the same order,
shortly after MonitorFS; a late package installation failed under 32K
boot allocation pressure even though the final permanent sizes matched.

### Dedicated AUXSTORE and root-mount mapping order (v277)

The D6FS root mount uses the shared BLOCKSET direct-tail configuration.
AUXSTORE therefore publishes its dedicated backing-store mapping **after**
root filesystem mounting, before process initialization. Publishing
AUXSTORE earlier allowed D6FS to replace its I/O span without resetting
the advertised backstore capacity. The corrected order passed the native
PDP-6 256K eight-pressure-child swap/resume and pipe-integrity test.
No permanent kernel words were added (32K GCC lowmem disk-root KCORE 10742,
KCORE+MRES 13605 words). The test does not replace long-duration swap
stress and repeated eviction/recovery verification.

# DAIMOS V0.9 MEMORY MANAGEMENT AND DEMAND MODULES

STATUS: SELECTED V0.9 ARCHITECTURE, 2026-09-07.  THIS DOCUMENT DEFINES THE
PDP-6 MEMORY-MANAGEMENT, COMPACTION, SWAPPING, AND DEMAND-MODULE MODEL FOR
DAIMOS V0.9.

## 1. GOALS

THE V0.9 MEMORY SYSTEM SHOULD PROVIDE A FULL MULTIUSER SYSTEM WITHIN THE
PROJECT MEMORY BUDGET: ALL FIXED RESIDENT AND IRREDUCIBLE SCRATCH KERNEL MEMORY
MUST REMAIN BELOW 16K WORDS.  EXACT KCC KCORE/MRES VALUES ARE BUILD
MEASUREMENTS, NOT A SECOND ARCHITECTURAL LIMIT.  THE MEMORY MANAGER MUST
ALSO FORM A SMALL, CLEAN BOUNDARY FOR MACHINE-SPECIFIC VM BACKENDS, INCLUDING A
LATER HARDWARE-PAGER BACKEND.

THE FIRST IMPLEMENTATION TARGETS PDP-6 RELOCATION/PROTECTION HARDWARE, NOT A
PAGE TABLE MMU.  THE PDP-6 HAS USER-MODE MEMORY PROTECTION AND RELOCATION
REGISTERS.  USER LOGICAL ADDRESSES ABOVE THE LOW UNRELOCATED AREA ARE RELOCATED
BY THE HARDWARE; THIS ALLOWS A STOPPED CONTIGUOUS USER IMAGE TO MOVE IN PHYSICAL
CORE WITHOUT REWRITING ITS INTERNAL LOGICAL POINTERS.

THE PERMANENT KERNEL NUCLEUS IS NEVER MOVED AND MUST REMAIN PROTECTED FROM USER
MODE.

### RAM-scaled MM extent descriptors (v261, 2026-10-24)

KINIT reserves the PDP-6 MM descriptor registry immediately after the
linked KCORE boundary, before publishing the first free-memory arena.
The symbol `mm_extents` retains its fixed direct address, so assembler MM
and VM consumers do not acquire pointer indirection. Each descriptor
requires two 36-bit words. Boot-time capacity is twice the selected
process-slot count plus eight non-process/transient entries, giving:

| Installed RAM | Process slots | MM descriptors | MM table (words) |
|---|---:|---:|---:|
| 32K | 8 | 24 | 48 |
| 64K | 16 | 40 | 80 |
| 128K | 32 | 72 | 144 |
| 256K | 64 | 136 | 272 |

Both process slots and extent capacity grow at complete 8K memory-bank
boundaries. The user-image and per-process kernel u-area normally each
require one extent. Capacity is a metadata ceiling, not a promise that
all processes fit resident in physical RAM. The native PDP-6 allocator
boundary tests pass at both 24 and 136 extents; the five-process
fresh-image scheduler test passes with this scheme. A fresh 256K GCC full-profile five-process scheduler boot also
passes; simultaneous 64-process load and peak occupancy testing
remain open.

KCORE size is measured against the 32K-word lowmem GCC disk-root link.
The v261 KCORE label is 10,742 words, while the actual fixed nucleus
plus its external 48-word descriptor backing is 10,790 words. The
corresponding KCORE+MRES physical footprint is 13,653 words, not the
bare 13,605-word linked report. These are 15 words below the earlier
fixed 32-entry registry baseline.

### 256K guest descriptor samples (v266)

A first-boot PDP-6 256K GCC test with all 63 numbered process
slots occupied completed successfully. SIMH sampled the actual
registry without modifying permanent KCORE. At the final process's
start marker, 132 of 136 extent descriptors were occupied; at its
completion marker, eight were occupied. These are point readings,
not lifetime peak statistics. Only four descriptors remained free
at the final-start checkpoint, so additional concurrent allocation
and swapping must be tested before regarding the reserve as adequate.

### 256K native insertion trace (v267)

A new 63-process fresh-boot PDP-6 SIMH run traced every entry into
`mm_extent_insert` with a simulator breakpoint action, without changing
permanent KCORE. The 151 recorded entry samples reached a maximum
pre-insert count of 131. The assembler performs its only positive
extent-count update on successful insertion inside that routine;
together with the independently observed 132-descriptor occupancy in
v266, this establishes a peak of **132 out of 136** for this particular
workload. Only four descriptors remained free at peak. Allocation and
swap-transition pressure beyond this workload is still unverified.

## 2. MEMORY-MANAGEMENT BOUNDARY

V0.9 PLACES ALLOCATION, COMPACTION, PINNING, PROCESS-IMAGE MOVEMENT, AND
SWAPPING UNDER ONE SMALL KERNEL MEMORY-MANAGEMENT SECTION.  RUNTIME MODULE
MOVEMENT HAS ITS OWN RELOCATION IMPLEMENTATION BUT IS NOT YET WIRED INTO THE
GENERAL MM COMPACTOR; THAT INTEGRATION REMAINS DEFERRED UNTIL THE RUNTIME
MODULE/OVERLAY STAGE.

THE INITIAL INTERFACE SHOULD REMAIN NARROW.  CONCEPTUALLY:

```TEXT
MM
 |
 +-- ALLOCATE/FREE CONTIGUOUS EXTENTS
 +-- COALESCE FREE EXTENTS
 +-- PIN/UNPIN PHYSICALLY STABLE OBJECTS
 +-- COMPACT MOVABLE OBJECTS ON DEMAND
 +-- MOVE STOPPED PDP-6 PROCESS IMAGES
 +-- LATER: MOVE/RELOCATE LOADABLE KERNEL MODULES
 +-- PROCESS SWAP POLICY WITH CLEAN/DIRTY TEXT SEMANTICS
 |
 +-- LATER: MACHINE-SPECIFIC PAGER BACKEND
```

THE REST OF THE KERNEL SHOULD REFER TO PROCESS OR MODULE DESCRIPTORS RATHER
THAN SPREADING PHYSICAL-BASE ARITHMETIC THROUGH UNRELATED SUBSYSTEMS.

## 3. PDP-6 ADDRESS MODEL

THE PDP-6 IS NOT A PAGED MACHINE, BUT IT IS ALSO NOT A SIMPLE UNRELOCATED
"NO-MMU" TARGET.  USER MODE HAS MEMORY PROTECTION AND RELOCATION REGISTERS.
THE REPOSITORY PDP-6 HANDBOOK STATES THAT THE MP REGISTER LIMITS THE USER
PROGRAM AND THAT, FOR RELOCATABLE USER REFERENCES, RL IS ADDED BEFORE THE
MEMORY CYCLE.  THE PDP-6 ADDRESS MODEL HERE IS THE ACTIVE DAIMOS DESIGN;
HAVING ONE PROTECTION AND ONE RELOCATION REGISTER.

FOR DAIMOS THIS MEANS A STOPPED PROCESS IMAGE MAY BE COPIED FROM ONE PHYSICAL
EXTENT TO ANOTHER AND RESUMED WITH A NEW SAVED RELOCATION BASE WHILE ITS
LOGICAL ADDRESSES REMAIN UNCHANGED.

THIS DOES NOT APPLY TO EXECUTIVE/KERNEL MODULE CODE.  MOVED MODULES REQUIRE
SOFTWARE RELOCATION.

### 3.1 MEMFS XIP RESERVATION (IMPLEMENTED FOUNDATION V202, 2026-10-23)

TRUE PDP-6 EXECUTION-IN-PLACE MAY USE ONE ALIGNED, MEMFS-OWNED CONTIGUOUS
REGION, BUT THE MACHINE HAS NO INDEPENDENT SHARED-TEXT/PRIVATE-DATA MAPPING.
THE APR CANNOT STOP A USER PROGRAM FROM WRITING INTO ITS OWN TEXT. ORDINARY
DXR PROGRAMS MUST NOT BE TREATED AS SAFE SHARED XIP IMAGES.

THE MEMFS MRES NOW PROVIDES KERNEL-ONLY `MEMFS_XIP_PIN` AND
`MEMFS_XIP_UNPIN` SERVICES FOR ALREADY RESIDENT REGULAR FILES ON DEDICATED,
`02000`-WORD-ALIGNED CHUNKS. FOUR PACKED SLOT/REFERENCE ENTRIES COVER THE
MAXIMUM FOUR DEDICATED MEMFS CHUNKS. THE FIRST FILE REFERENCE TAKES AN
EXISTING MM PIN; THE FINAL REFERENCE RELEASES THAT PIN. PINNED FILES
REJECT WRITES, RESIZING, AND TRUNCATION; RECLAIM SKIPS PINNED CHUNKS;
UNMOUNT REJECTS LIVE REFERENCES.

THE TARGET MEMFS REGRESSION PASSES ON BOTH SIMH AND AAP PDP-6; FIFO TESTS
PASS ON SIMH. THIS IS A COMPLETED PINNING PRIMITIVE, NOT A WORKING PROCESS
XIP LOADER. IMAGE ADMISSION, GUARANTEED ALIGNED ALLOCATION, VM OWNERSHIP,
AND VERIFIED NON-WRITING USER CODE REMAIN UNIMPLEMENTED. NO EXEC/RUN XIP
PATH IS ENABLED.

V203 UPDATE (2026-10-23): NEW LARGE DEDICATED MEMFS EXTENTS NOW USE THE
EXISTING MM ALIGNED ALLOCATOR WITH `02000`-WORD ALIGNMENT. SMALL PACKED
EXTENTS RETAIN THE ORIGINAL PATH. THE NATIVE MEMFS/PINNING TEST PASSES ON
SIMH AND AAP. IMAGE ADMISSION, VM OWNERSHIP AND PROCESS EXECUTION ARE STILL
NOT IMPLEMENTED; ALIGNMENT FRAGMENTATION UNDER PRESSURE NEEDS MEASUREMENT.

V207 UPDATE (2026-10-23): MEMFS MRES PROVIDES `MEMFS_XIP_ADMIT`, WHICH
REQUIRES A DISTINCT RAW-IMAGE MAGIC (OCTAL 054506000000), A VALID LOGICAL
ENTRY, AND PINNED 02000-ALIGNED BACKING. THE ADMISSION PROBE UNPINS ON
FAILURE. SIMH AND AAP NATIVE FIXTURES PASS. THIS ONLY VALIDATES FILE FORMAT,
NOT NON-WRITING PROGRAM BEHAVIOR: XIP PROCESS MAPPING, STARTUP AND TEARDOWN
REMAIN DISABLED. ORDINARY DXR IS NOT ELIGIBLE.

V209 UPDATE (2026-10-23): A TESTKIT-ONLY AAP PDP-6 USER-MODE APR TEST NOW
EXECUTES INSTRUCTIONS FROM A PINNED MEMFS FILE USING ITS ACTUAL RELOCATION
BASE. THE TEST DELIBERATELY STORES TO LOGICAL ADDRESS 030 AND OBSERVES THE
CHANGED PHYSICAL MEMFS WORD. THIS CONFIRMS BOTH RELOCATED INSTRUCTION FETCH
AND THE ABSENCE OF HARDWARE WRITE PROTECTION. IT IS NOT A SAFE PRODUCTION
SHARED-XIP PROGRAM. PROCESS TEARDOWN, SWAP EXCLUSION AND NON-WRITING USER
STARTUP REMAIN TO BE IMPLEMENTED. KCORE AND EXEC/RUN ARE UNCHANGED.

V210 UPDATE (2026-10-23): EXPLICIT XIP ADMISSION REQUIRES EXECUTE MODE
AND REJECTS ALL FILE-MODE WRITE BITS BEFORE ACQUIRING THE IMAGE PIN.
NATIVE TESTS CONFIRM REJECTION OF WRITABLE/NONEXECUTABLE MODES AND ACCEPT
MODE 0555 ON SIMH AND AAP. THIS DOES NOT GIVE PDP-6 HARDWARE READ-ONLY
PROTECTION: PRIVILEGED IMAGE TRUST, VM TEARDOWN AND NON-WRITING USER
STARTUP ARE STILL REQUIRED BEFORE PRODUCTION XIP EXECUTION.

V211 UPDATE (2026-10-23): MEMFS MRES NOW PROVIDES
`MEMFS_XIP_UNPIN_BASE(BASE)` TO RELEASE A LIVE REFERENCE BY ITS EXACT
PHYSICAL IMAGE BASE. THE FOUR EXISTING PACKED REFERENCES PROVIDE REVERSE
LOOKUP WITHOUT ADDING RESIDENT DATA. THE LAST REFERENCE RELEASES THE MM PIN;
UNKNOWN OR REPEATED RELEASES FAIL. SIMH AND AAP PDP-6 NATIVE TESTS PASS.
VM TEARDOWN STILL MUST DISTINGUISH MEMFS OWNERSHIP FROM PROCESS EXTENTS;
EXEC/RUN AND PERMANENT KCORE HAVE NOT BEEN MODIFIED.

V212 UPDATE (2026-10-23): MEMFS MRES ALSO PROVIDES
`MEMFS_XIP_OWNS_BASE(BASE)` AS AN EXACT, NONMUTATING LOOKUP OF A LIVE
PINNED MEMFS ALLOCATION. IT REUSES THE FOUR EXISTING REFERENCE WORDS;
SIMH AND AAP TESTS PASS AND KCORE STAYS AT 025123 OCTAL (10835 WORDS).
THIS IS PROVENANCE SUPPORT ONLY: VM TEARDOWN, SWAP EXCLUSION AND EXEC/RUN
ADMISSION HAVE NOT YET BEEN CONNECTED.

V213 UPDATE (2026-10-23): VM DESTROY NOW USES PRIVATE MEMFS REGISTER OP 026
TO RELEASE A RECOGNIZED XIP PHYSICAL BASE THROUGH THE MOVABLE MEMFS MRES.
AN UNRECOGNIZED BASE STILL REQUIRES THE USUAL MM_TYPE_PROCESS AND OWNER
CHECKS; A MEMFS DYNAMIC ALLOCATION CANNOT BE FREED AS A PROCESS EXTENT.
NATIVE SIMH/AAP DISPATCH TESTS PASS. GCC LOWMEM KCORE IS 025134 OCTAL,
10844 WORDS (+9). PRODUCTION XIP LAUNCH IS NOT IMPLEMENTED YET.

V214 UPDATE (2026-10-23): NATIVE SIMH AND AAP PDP-6 TESTS INJECT AN MM
UNPIN FAILURE THROUGH THE PRIVATE MEMFS XIP RELEASE DISPATCH. FAILURE
RETURNS -1 AND PRESERVES THE LIVE PIN/REFERENCE; RETRY AFTER THE ERROR
CLEARS SUCCEEDS. THIS TEST DOES NOT YET EXECUTE PROCESS-LEVEL VM TEARDOWN
OR EXEC REPLACEMENT. PRODUCTION KCORE/EXEC/RUN ARE UNCHANGED.

V215 UPDATE (2026-10-23): THE NATIVE MEMFS FIXTURE LINKS THE ACTUAL
PRODUCTION `VM_SPACE_DESTROY` ASSEMBLY BODY AND TESTS AN XIP-BACKED VM
DESCRIPTOR. INJECTED MM UNPIN FAILURE PRESERVES THE VM STATE AND MEMFS PIN;
RETRY RELEASES THE PIN AND CLEARS VM STATE. SIMH AND AAP PDP-6 PASS.
SCHEDULER/STORAGE SUPPORT IS TEST-STUBBED; NO KCORE GROWTH.

V216 UPDATE (2026-10-23): A STRICTLY NON-WRITING USER-MODE RAW XIP IMAGE
EXECUTES DIRECTLY FROM PINNED MEMFS BACKING AND ISSUES MONITOR UUO 040.
ITS EXECUTIVE TRAP HANDLER CHECKS THE IMAGE IS UNCHANGED, INVOKES THE
ACTUAL `VM_SPACE_DESTROY` ROUTINE, CHECKS THE PIN AND VM STATE ARE CLEARED,
AND CONFIRMS THE FILE CAN BE RECLAIMED. BOTH SIMH AND AAP PDP-6 PASS.
THE TEST USES FOCUSED SCHEDULER/STORAGE STUBS AND DOES NOT YET EXERCISE
PRODUCTION EXEC/RUN, LIVE PROCESS EXIT, OR THE PRIVILEGED NO-WRITE TRUST
CONTRACT. KCORE REMAINS 025134 OCTAL (10844 WORDS).

V217 UPDATE (2026-10-23): RAW XIP ADMISSION REUSES THE EXISTING DXR PURE
FLAG VALUE 0200000 IN THE LEFT HALF OF RAW HEADER WORD 1, WITH THE LOGICAL
ENTRY IN ITS RIGHT HALF. THE FLAG MUST BE PRESENT; ABSENCE IS REJECTED
WITHOUT LEAVING AN MM PIN. ORIGINAL DXR PURE EXECUTABLES CONTINUE TO USE
THE NORMAL LOAD/COPY PATH. PURE IS A SOFTWARE PROMISE AND CANNOT PREVENT
PDP-6 USER-MODE WRITES; A PRIVILEGED NON-WRITING XIP PATH IS STILL REQUIRED
FOR PRODUCTION EXEC/RUN. NATIVE MEMFS/USER-APR/LIFECYCLE TESTS PASS.

V218 UPDATE (2026-10-23): HOST DLINK NOW HAS `--XIP --PURE -B 020` AS AN
EXPERIMENTAL RAW-IMAGE PRODUCER. IT EMITS EXACTLY 04000 WORDS WITH XIP
MAGIC AT OFFSET ZERO, `PURE,,020` AT OFFSET ONE, PROGRAM TEXT AT 020,
AND ZERO PADDING. NO DATA/BSS IS PERMITTED. THE INITIAL SAFE INSTRUCTION
SUBSET CONSISTS SOLELY OF DIRECT, UNINDEXED MOVEI INSTRUCTIONS FOLLOWED
BY SYS_EXIT UUO 040. THE LINKER REJECTS EVERYTHING ELSE. THE OUTPUT IS
NOT A NORMAL DXR FILE AND PRODUCTION EXEC/RUN DOES NOT YET ADMIT IT.
HOST FORMAT/REJECTION TESTS LIVE IN DAIMOS-TESTKIT. KCORE IS UNCHANGED.

V219 UPDATE (2026-10-23): THE EXISTING MOVABLE MEMFS REGISTER GATEWAY NOW
IMPLEMENTS PRIVATE OP 027 FOR XIP ADMISSION, WITH AC1=VNODE, AC2=&BASE,
AND AC3=&ENTRY. THE MRES DOES VALIDATION AND PINNING; ITS EXISTING OP 026
RELEASES ONE XIP REFERENCE BY BASE. NATIVE SIMH AND AAP TESTS PASS USING
THE REAL GATEWAY. THIS ADDS NO KCORE ENTRY VECTOR OR PROCESS WORD.
PRODUCTION RUN/EXEC MUST STILL PROVIDE ROOT-ONLY ADMISSION AND A
NON-WRITING STARTUP BEFORE USING THE RETURNED BASE.

## 4. PERMANENT NUCLEUS AND MANAGED CORE

MEMORY BELOW THE MANAGED-MEMORY BOUNDARY IS THE PERMANENT NUCLEUS AND IS NOT A
COMPACTION OBJECT.  ABOVE THAT BOUNDARY THE MM LAYER TRACKS CONTIGUOUS EXTENTS.

A MINIMAL REGION DESCRIPTION SHOULD IDENTIFY:

```TEXT
BASE
SIZE
OWNER/TYPE
FLAGS OR PIN COUNT
```

THE EXACT PACKING SHOULD BE CHOSEN BY MEASUREMENT.  DO NOT ADD GENERIC VM
METADATA THAT IS NOT REQUIRED BY THE NON-PAGED V0.9 IMPLEMENTATION.

USEFUL OBJECT CLASSES ARE:

```TEXT
FREE
PROCESS
MODULE
KERNEL-DYNAMIC
RECLAIMABLE-CACHE
```

PHYSICAL STABILITY AND LOGICAL LIFETIME ARE SEPARATE PROPERTIES.  A MODULE MAY
BE LOGICALLY RESIDENT BUT STILL PHYSICALLY MOVABLE.


### 4.1 RECLAIMABLE KERNEL CACHE MEMORY

RECLAIMABLE CACHE IS KERNEL-OWNED BUT NOT PERMANENTLY RESERVED CORE.  ITS
PURPOSE IS TO TURN IDLE MEMORY INTO I/O SPEED WITHOUT REDUCING THE MEMORY
AVAILABLE TO PROCESSES WHEN THE MACHINE BECOMES BUSY.

THE CURRENT FIRST USER IS THE D6FS BLOCK CACHE.  THE PLANNED BSTORE/BCACHE
CONSOLIDATION IN `KERNEL/BSTORE.MD` REPLACES THAT PRIVATE CACHE WITH ONE
GENERIC TWO-BLOCK CLEAN RAM CACHE SHARED BY D6FS AND DTFS/TSFS.  BCACHE IS
WRITE-THROUGH AND CLEAN-ONLY, SO EVERY L1 EXTENT MAY BE DISCARDED WITHOUT I/O.
CACHE RECLAIM PRECEDES MEMFS EVICTION AND PROCESS SWAPPING; IT IS ALSO TRIED
WHEN THE FIXED MM EXTENT-DESCRIPTOR TABLE, RATHER THAN FREE WORD COUNT,
PREVENTS AN ORDINARY ALLOCATION.

THE CACHE MAY GROW ABOVE A LOW FREE-MEMORY WATERMARK AND SHOULD SHRINK AS THAT
RESERVE IS APPROACHED.  DO NOT MAKE THE WATERMARK A LARGE FIXED FRACTION OF
RAM: A 32K PDP-6 AND A LARGER PDP-10 SHOULD BOTH USE THEIR OTHERWISE-IDLE CORE
EFFECTIVELY.  THE MM INTERFACE SHOULD SUPPORT A SMALL RECLAIM CALLBACK/SERVICE
RATHER THAN EMBEDDING D6FS, DTFS, OR DEVICE POLICY IN THE GENERAL ALLOCATOR.

RECLAIMABLE CACHE DOES NOT COUNT AS FIXED KERNEL-IMAGE SIZE FOR THE BELOW-16K
RESIDENT+SCRATCH GATE BECAUSE THE MEMORY IS RETURNED TO THE GENERAL MANAGED-
CORE ALLOCATOR UNDER PRESSURE.  ITS RESIDENT CODE, DESCRIPTORS THAT CANNOT BE
FREED, AND IRREDUCIBLE SCRATCH DO COUNT.  ACTIVE DIRTY I/O AND OTHER
NONREGENERABLE DYNAMIC KERNEL STATE ARE NOT RECLAIMABLE MERELY BECAUSE THEY
WERE ALLOCATED FROM MANAGED CORE.

## 5. ON-DEMAND COMPACTION

UNLOADING OR FREEING AN OBJECT SHOULD ONLY COALESCE ADJACENT FREE EXTENTS.  IT
SHOULD NOT AUTOMATICALLY COMPACT MEMORY.

WHEN AN ALLOCATION OF N CONTIGUOUS WORDS CANNOT BE SATISFIED:

```TEXT
1. RECLAIM CLEAN RECLAIMABLE-CACHE EXTENTS THAT CAN HELP THE REQUEST.
2. EVICT ELIGIBLE MEMFS DATA TO BACKSTORE WHEN THAT CAN SATISFY THE REQUEST.
3. IF TOTAL FREE >= N BUT LARGEST FREE EXTENT < N, RUN THE KERNEL COMPACTOR.
4. MOVE ONLY ENOUGH MOVABLE OBJECTS TO CREATE AN EXTENT >= N.
5. RETRY THE ALLOCATION.
6. IF ACTUAL FREE CORE IS STILL INSUFFICIENT, SWAP AN ELIGIBLE PROCESS TO
   BACKSTORE AND RETRY.
7. FAIL ONLY WHEN CACHE RECLAIM, MEMFS EVICTION, COMPACTION, AND PROCESS
   SWAPPING CANNOT SATISFY IT.
```

THE CURRENT COMPACTOR MOVES PROCESS EXTENTS ONLY.  LOADABLE KERNEL MODULES MAY
EVENTUALLY BECOME COMPACTION OBJECTS, BUT MODULE RELOCATION IS NOT PART OF THE
ACTIVE V0.9 MM ALLOCATION PATH YET.

THE PERMANENT NUCLEUS IS NEVER MOVED.  ACTIVE DMA/I/O USERS AND ANY OBJECT WITH
A NONZERO PHYSICAL PIN COUNT ARE NOT MOVED.

THE IMPLEMENTED COMPACTOR RELOCATES THE EXISTING EXTENT DIRECTLY.  WHILE PI IS
SUPPRESSED, MM TEMPORARILY EXCLUDES THE SOURCE DESCRIPTOR FROM THE FIT SEARCH.
THE SOURCE RANGE THEREFORE PARTICIPATES IN THE CANDIDATE FREE SPAN, WHICH ALLOWS
A DESTINATION TO OVERLAP THE OLD IMAGE EVEN WHEN THE ADJACENT FREE HOLE IS
SMALLER THAN THE OBJECT.  NO SECOND FULL IMAGE AND NO SECOND EXTENT DESCRIPTOR
ARE REQUIRED.

PROCESS IMAGES KEEP THE HIGH-PACKING POLICY.  A MOVE MAY ALSO SELECT A NONOVERLAPPING DESTINATION IN ANOTHER MANAGED ARENA WHEN
THAT IS THE BEST FIT.  `fs_move_words()` PROVIDES OVERLAP-SAFE WORD MOVEMENT:
THE NORMAL FORWARD/BLT PATH IS RETAINED WHERE SAFE AND UPWARD OVERLAP COPIES
BACKWARD.  THE PROCESS SAVED-RELOCATION-BASE UPDATE AND THE MM EXTENT-DESCRIPTOR
REBASE/RESORT ARE PUBLISHED WHILE PI REMAINS SUPPRESSED SO INTERRUPT PATHS
CANNOT OBSERVE A HALF-PUBLISHED OWNER.

THE TARGET CHEAPEST-FIRST ORDER AFTER THE BSTORE/BCACHE MIGRATION IS: RECLAIM
THE GENERIC CLEAN BCACHE L1, EVICT MEMFS TO BACKSTORE, COMPACT IF FRAGMENTED,
THEN SWAP AN ELIGIBLE PROCESS TO BACKSTORE.  CACHE ALLOCATION ITSELF CONTINUES
TO USE THE NO-RECLAIM ENTRY SO IT CANNOT RECURSE INTO CACHE RECLAIM.

### 5.1 COMPACTOR REGRESSION STATUS

THE 2026-09-30 MM SWEEP USED A FOCUSED CURRENT-SEMANTICS HOST REGRESSION.  IT
COVERS:

- LOW/HIGH ALIGNED PLACEMENT AND SORTED DESCRIPTOR INSERT/DELETE SHIFTS;
- PIN SATURATION, PIN UNDERFLOW REJECTION, AND PINNED-STATE QUERIES;
- THE TWO-02000-HOLE / 04000-WORD PROCESS DIRECT-SLIDE CASE;
- ALLOCATION PRESSURE ORDER: DIRECT ATTEMPT, CACHE RECLAIM/COMPACTION, THEN
  PROCESS-SWAP RECLAIM/COMPACTION;
- INVALID ALIGNMENT, TYPE, PREFERENCE, AND OVERSIZED COMPACTION REQUESTS.

THE OLDER `DAIMOS-TESTKIT` `MM-V1` HOST PROGRAM IS NOT A CURRENT ACCEPTANCE
ORACLE: IT FAILS THE UNMODIFIED PRE-SWEEP TREE AT ITS OBSOLETE 32K PROCESS-SLOT
SIZING EXPECTATION.  THE FOCUSED MM REGRESSION SHOULD BE MOVED INTO THE
DESIGNATED PDP-C-TESTKIT WHEN THAT CURRENT TEST TREE IS AVAILABLE.

THE EXISTING REAL PDP-6 ROOT-LOGIN LIFECYCLE TEST ALSO CURRENTLY STOPS WITH
`?RT` ON THE UNMODIFIED PRE-SWEEP TREE, SO THAT FAILURE IS NOT AN MM-SWEEP
REGRESSION.  THE FULL PDP-6 IMAGE BUILD AND PERMANENT-SIZE ACCEPTANCE COMPLETE
SUCCESSFULLY WITH THE CURRENT MM IMPLEMENTATION.

## 6. PROCESS MOVEMENT

ON PDP-6, STOPPED USER PROCESSES ARE THE CHEAPEST MOVABLE OBJECTS.  THE PROCESS
KEEPS ITS LOGICAL ADDRESS SPACE; MM COPIES ITS CONTIGUOUS PHYSICAL IMAGE AND
UPDATES THE SAVED RELOCATION BASE USED WHEN THE PROCESS IS DISPATCHED.

THE RUNNING PROCESS OR AN I/O-PINNED PROCESS MAY BE TEMPORARILY IMMOBILE.  THE
INITIAL COMPACTOR SHOULD WORK AROUND PINNED ISLANDS RATHER THAN ATTEMPTING LIVE
MOVEMENT OF ACTIVE I/O STATE.

## 7. PROCESS SWAPPING AND TEXT SEMANTICS

V0.9 SHALL SUPPORT PROCESS SWAPPING WITHOUT REQUIRING PAGING, COPY-ON-WRITE,
`fork`, OR A GENERAL VIRTUAL-MEMORY LAYER.

EXECUTABLE TEXT HAS A DELIBERATELY SIMPLE ONE-BIT SWAP CONTRACT:

```TEXT
PURE EXECUTABLE TEXT
    THE EXECUTABLE PROMISES THAT ITS TEXT PREFIX IS IMMUTABLE AFTER EXEC.
    DO NOT WRITE COMPLETE BACKSTORE SECTORS COVERED ONLY BY THAT PREFIX.
    RELOAD THE OMITTED TEXT FROM THE EXECUTABLE ON SWAP-IN.

NON-PURE / SELF-MODIFYING TEXT
    IT HAS NO EXECUTABLE-BACKING RECONSTRUCTION CONTRACT.
    WRITE AND RESTORE THE COMPLETE PROCESS IMAGE.

WRITABLE DATA / STACK / HEAP
    SWAP NORMALLY WHEN THE PROCESS IS SWAPPED.
```

PDP-6 RELOCATION/PROTECTION DOES NOT DISTINGUISH WRITES FROM READS OR
INSTRUCTION FETCHES, SO DAIMOS DOES NOT ATTEMPT TO TRAP THE FIRST WRITE TO A
TEXT SUBREGION.  `PURE` IS THEREFORE A PROGRAM/TOOLCHAIN CONTRACT, NOT A
RUNTIME DIRTY BIT.  `DLINK --PURE` REFUSES A LINK WHEN ITS STATIC ANALYSIS
FINDS A DEFINITE TEXT WRITE.  PROGRAMS WHICH INTENTIONALLY MODIFY THEIR TEXT
MUST REMAIN NON-PURE.

THE RESIDENT SWAP RECORD STILL COSTS ONE WORD PER PROCESS.  FOR A PURE IMAGE
THE SWAPPER COMPUTES

```TEXT
SKIP = FLOOR((020 + TEXT_WORDS) / 0200) * 0200
```

AND WRITES ONLY `PROCESS_WORDS-SKIP` TO BACKSTORE.  THIS DISCARDS ONLY COMPLETE
0200-WORD SECTORS; THE SECTOR CONTAINING A TEXT/DATA BOUNDARY IS RETAINED SO
NO SUB-SECTOR DIRTY METADATA IS REQUIRED.  ON SWAP-IN, UNCOMPRESSED DXR2 TEXT
IS READ DIRECTLY FROM THE EXECUTABLE.  FOR D6LZ36 DXR2, THE ALREADY-VALIDATED
IMMUTABLE BACKING IS DECOMPRESSED THROUGH THE NORMAL D6LZ VFS FRONTEND AND THE
SWAPPED SUFFIX IS THEN OVERLAID, RESTORING ALL WRITABLE STATE.

THE 2026-10-16 V20 PURE BACKING FORMAT PRESERVES THIS **ONE-WORD SWAP RECORD**
FOR MOUNT IDS UP TO 26: 18 BITS HOLD THE EXECUTABLE VNODE INDEX, FOUR BITS
HOLD THE LOW ZERO-BASED MOUNT SLOT, AND 14 BITS HOLD THE FULL PURE TEXT LENGTH.
THE FIFTH MOUNT-SLOT BIT LIVES IN BIT 27 OF THE EXISTING STABLE U-AREA
CREDENTIAL WORD 066, ABOVE UID9/GID9/UMASK9. NO ADDITIONAL PROCESS WORD IS
ALLOCATED. SWAP-IN AND THE PURE VERIFIER DECODE THE RECORD WITH THE **OWNING**
U-AREA AND VALIDATE THE ACTIVE VFS MOUNT LIMIT BEFORE READING THE MOUNT ROOT.

RUN ATTACHES THE EXECUTABLE BEFORE ALLOCATING ITS CHILD U-AREA. ONLY WHILE THE
CHILD IS SIDL, THE UNUSED SCHEDULER RH BIT 0 STAGES THE FIFTH SLOT BIT; RUN
COPIES IT INTO THE CHILD'S CREDENTIAL WORD AFTER INHERITING UID/GID/UMASK,
WITHOUT COPYING THE PARENT'S EXECUTABLE BACKING BIT. THE STAGING BIT IS GONE
BEFORE THE CHILD'S SRUN RUN-QUEUE LINK IS PUBLISHED. TRANSACTIONAL EXEC SAVES
OLD AND NEW BACKING EXTENSIONS IN TRANSIENT STACK LOCALS AND RESTORES THE
MATCHING BIT ON FAILURE OR COMMIT. DETACH CLEARS THE BIT; PURE BUSY CHECKS
SKIP PROCESS SLOTS WITHOUT A VALID U-AREA. PDP-6 TESTS COVER THE FULL RAW
AND COMPRESSED SWAP TRANSACTION AT MOUNT 26 AND THE EXEC/RUN LIFECYCLE.

THIS IS A BACKING-FORMAT CAPABILITY, NOT PROOF THAT 26 VOLUMES CAN BE
SIMULTANEOUSLY MOUNTED WITH ANY GIVEN BOOT IMAGE. THE 2026-10-16 V22 KINIT
NOW SELECTS AN EXACT MM VFS CAPACITY FROM VALIDATED ROOT PHYSICAL MEMBERS,
SUCCESSFUL CONTROLLER MINIT PROBES, AND FILESYSTEM MRES PROFILES THAT ARE
ACTUALLY INSTALLED. THE SIZE IS SIX WORDS PER SLOT. ALL SIX TABLE BASES
ARE PUBLISHED BEFORE `VFS_MOUNT_LIMIT` IS UPDATED. A FULL DSK+DRM/D6FS OR
DTC/TSFS ROOT PROFILE WITH MEMFS NEEDS UP TO NINE SLOTS, NOT 26: THE OTHER
FAMILY'S FILESYSTEM MRES IS OMITTED. A DISK-ONLY FULL PROFILE MAY REQUIRE
ONLY FIVE SLOTS; THE REDUCED DISK PROFILE MAY NEED FOUR.

THE STRICT 32K KCORE GATE STILL FAILS BY 2146 WORDS. V22 STATIC 32K BOOT
IMAGE CHECKS END AT 073277 (DISK) AND 074277 (TAPE) OCTAL, LEAVING
1345/833 WORDS AFTER THEIR RESERVED KINIT STACKS. THESE ARE FOOTPRINT
MEASUREMENTS, NOT SUCCESSFUL 32K EXECUTIONS.

WHILE A NONRESIDENT PURE PROCESS DEPENDS ON AN EXECUTABLE VNODE, WRITE,
TRUNCATE, AND UNLINK OF THAT VNODE ARE REJECTED AND THE CONTAINING MOUNT MAY
NOT BE UNMOUNTED.  THE CHECK SCANS THE EXISTING PROCESS TABLE; THERE IS NO
SECOND VNODE REFERENCE TABLE OR PER-VNODE RESIDENT STATE.

DEBUG/VALIDATION BUILDS MAY SET `PROC_SWAP_VERIFY_PURE=1`.  THAT OPTION IS
COMPILED OUT OF NORMAL KERNELS.  BEFORE DISCARDING PURE TEXT IT TEMPORARILY
SAVES THE COMPLETE IMAGE TO BACKSTORE, RECONSTRUCTS THE EXECUTABLE-BACKED
PREFIX, COMPARES THE OMITTED TEXT THROUGH THE SHARED 0200-WORD FILESYSTEM
WORKSPACE, RESTORES THE ORIGINAL IMAGE, AND HALTS AT
`PROC_SWAP_PURE_CONTRACT_VIOLATION` ON A MISMATCH.  THE CHECK THEREFORE COSTS
I/O AND TEMPORARY BACKSTORE IN A DIAGNOSTIC BUILD BUT NO EXTRA PER-PROCESS
RAM.

GOOD SWAP CANDIDATES INCLUDE LONG-SLEEPING, TERMINAL-WAITING, CHILD-WAITING,
AND LOW-PRIORITY BACKGROUND PROCESSES.  A RAW RESERVED SWAP REGION IS PREFERRED
FOR THE FIRST IMPLEMENTATION TO A FILESYSTEM SWAP FILE.

## 8. BOOT-IMAGE MODULE STORAGE

DRIVER AND OPTIONAL-SUBSYSTEM MODULE PAYLOADS SHOULD REMAIN PRESENT IN THE BOOT
IMAGE SO THEY ARE ALWAYS AVAILABLE WITHOUT DEPENDING ON A SECONDARY MODULE
FILESYSTEM OR ADMINISTRATIVE INSTALLATION STEP.

THE RESIDENT KERNEL SHOULD NOT CARRY STRING NAMES OR LARGE DIRECTORY RECORDS
FOR THESE PAYLOADS.  MODULES HAVE BUILD-TIME NUMERIC IDS.  A COMPACT BOOT-IMAGE
LOCATION TABLE SHOULD USE THE MODULE ID AS THE ARRAY INDEX.

THE TARGET IS ONE WORD PER MODULE OR BETTER.  ONE SIMPLE REPRESENTATION IS AN
END-OFFSET TABLE:

```TEXT
END[0]
END[1]
...

START(N) = END(N - 1)
SIZE(N)  = END(N) - END(N - 1)
```

THE FIRST START IS THE KNOWN MODULE-PAYLOAD BASE.  MODULE-SPECIFIC SIZE OR
RELOCATION METADATA MAY REMAIN IN THE NONRESIDENT MODULE HEADER IF THAT IS
SMALLER OVERALL.  HUMAN-READABLE MODULE NAMES BELONG IN USERLAND/ADMIN TOOLS.

## 9. LOADABLE/UNLOADABLE MODULE MODEL

THE EXISTING MINIT/MRES RELOCATION MECHANISM SHOULD BE EVOLVED RATHER THAN
REPLACED BY AN ELF-LIKE DYNAMIC LINKER.  THE MODULE FORMAT SHOULD REMAIN
compact and numeric, WITH A SMALL FIXED KERNEL-SERVICE ABI.

A loadable module needs only the metadata required for:

```TEXT
IMAGE SIZE / BSS
RELOCATION MAP
IMPORT SERVICE SLOTS
EXPORT SERVICE SLOTS
INIT ENTRY
FINI ENTRY
```

DO NOT ADD STRING SYMBOL TABLES, GENERAL KERNEL DYNAMIC LINKING, HASH TABLES,
OR VERSIONED ELF-LIKE SECTIONS.

CROSS-MODULE CALLS SHOULD GO THROUGH SMALL STABLE SERVICE SLOTS WHERE POSSIBLE.
MOVING A MODULE THEN REQUIRES UPDATING ITS INTERNAL RELOCATION SITES AND A
SMALL NUMBER OF PUBLISHED SERVICE/PI/CALLBACK ENTRIES, NOT SCANNING ALL KERNEL
CALLERS.

## 10. AUTOMATIC DRIVER AND FILESYSTEM LOADING

DRIVER PAYLOADS REMAIN IN THE BOOT IMAGE BUT MAY STAY NONRESIDENT UNTIL FIRST
use.  A tiny resident device descriptor identifies the module ID required to
activate the device.

RAW DEVICE ACCESS AUTOLOADS ONLY THE DRIVER:

```TEXT
OPEN /dev/mt0
    -> LOAD MTC DRIVER IF ABSENT
    -> RAW READ/WRITE/DD WORKS
```

MOUNTING A MEDIUM MAY AUTOLOAD AN ADDITIONAL FILESYSTEM PERSONALITY:

```TEXT
MOUNT /dev/mt0 /mount/tape
    -> ENSURE TAPE DRIVER
    -> PROBE ONLY ENOUGH MEDIA DATA TO IDENTIFY FORMAT
    -> LOAD THE NATIVE DTFS PERSONALITY; TENEX/ITS ARE DISABLED
    -> MOUNT
```

RAW `dd`-STYLE ACCESS MUST NOT LOAD FILESYSTEM-PERSONALITY CODE.

EXPLICIT MOUNT OVERRIDES MAY SELECT A PERSONALITY DIRECTLY AND AVOID PROBING.
PERSONALITY DETECTION MUST NOT LOAD ALL PERSONALITY IMPLEMENTATIONS MERELY TO
ask each one whether it recognizes the medium.

SLOW REMOVABLE DEVICES SUCH AS MAGTAPE, DECTAPE, PAPER TAPE, AND CARD EQUIPMENT
ARE STRONG INITIAL AUTOLOAD CANDIDATES BECAUSE MODULE-LOAD LATENCY IS SMALL
relative to device latency.

## 11. MODULE LIFETIME, REFERENCES, AND MANUAL RESIDENCY

AUTOMATIC AND MANUAL LOADS HAVE DIFFERENT LIFETIME POLICY.

```TEXT
AUTO LOAD:
    UNLOAD WHEN LOGICAL REFERENCE COUNT REACHES ZERO AND THE MODULE IS QUIESCENT.

MANUAL LOAD:
    REMAIN LOADED AT REFERENCE COUNT ZERO.
    UNLOAD ONLY AFTER AN EXPLICIT PRIVILEGED UNLOAD REQUEST.
```

AN EXPLICIT LOAD OF AN ALREADY AUTOLOADED MODULE PROMOTES IT TO MANUAL
RESIDENCY WITHOUT RELOADING IT.

MANUAL RESIDENCY IS A LIFETIME PIN, NOT A PHYSICAL-ADDRESS PIN.  AN IDLE
MANUALLY LOADED MODULE REMAINS MOVABLE BY THE COMPACTOR.

A separate physical pin count protects active I/O, DMA-visible storage, or any
other state whose address must not change temporarily.

UNLOAD REQUIRES AT LEAST:

```TEXT
OPEN/ACTIVE REFERENCES == 0
MOUNT REFERENCES == 0
DEPENDENT MODULE REFERENCES == 0
PENDING/QUEUED I/O == 0
PHYSICAL PIN COUNT == 0
INTERRUPT/CALLBACK ACTIVITY QUIESCENT
```

FORCED UNLOAD IS NOT PART OF V0.9.  A BUSY MANUAL UNLOAD RETURNS BUSY.

THE KERNEL PRIVILEGE CHECK SHOULD BE SMALL; A USERLAND `sudo`-LIKE TOOL MAY
HANDLE AUTHENTICATION AND THEN INVOKE THE PRIVILEGED MODULE LOAD/UNLOAD ABI.

## 12. MODULE RELOCATION DURING COMPACTION

A module executes at executive addresses and therefore cannot use the user
relocation register as a transparent logical-address mechanism.  MM must
perform software relocation when a movable module changes base.

RETAIN OR RECONSTRUCT THE MODULE RELOCATION MAP WHILE THE MODULE IS LOADED.
MOVEMENT IS:

```TEXT
PLAN DESTINATION WITH THE SOURCE EXTENT TEMPORARILY EXCLUDED FROM FIT SEARCH
MOVE IMAGE OVERLAP-SAFELY
APPLY BASE DELTA TO RELOCATION-MARKED WORDS
UPDATE EXPORTED SERVICE SLOTS
UPDATE REGISTERED PI/CALLBACK ENTRIES
PUBLISH NEW MODULE BASE
REBASE/RESORT THE EXISTING MM EXTENT DESCRIPTOR
```

NO SECOND COMPLETE DESTINATION IMAGE OR EXTENT DESCRIPTOR IS RESERVED.  PI IS
SUPPRESSED ACROSS COPY, RELOCATION, OWNER PUBLICATION, AND MM DESCRIPTOR REBASE
SO NO INTERRUPT CAN ENTER HALF-RELOCATED CODE.
A module is skipped while physically pinned.

## 13. OVERLAYS

DO NOT FREEZE OR IMPLEMENT A GENERAL USERSPACE OVERLAY ABI BEFORE INITIAL
NATIVE USERLAND EXISTS.

THE FIRST OVERLAY DECISION POINT IS AFTER DAS AND ELVIS RUN AND CAN BE
MEASURED.  RECORD AT LEAST:

```TEXT
TEXT WORDS
DATA WORDS
BSS WORDS
PEAK DYNAMIC MEMORY
PEAK USER STACK
TOTAL PEAK USER EXTENT
```

MEASURE DAS1 AND DAS2 SEPARATELY.  DAS ALREADY USES SEQUENTIAL PHASES AND
SCRATCH-BACKED WORKING SETS, SO ITS ARCHITECTURE IS EVIDENCE THAT PHASE
SPLITTING CAN SOLVE LARGE-PROGRAM MEMORY PRESSURE WITHOUT A GENERAL OVERLAY
ABI.

ELVIS IS THE FIRST PRACTICAL INTERACTIVE LARGE-PROGRAM MEASUREMENT.  ITS
NATIVE IMAGE AND WORKING SET SHOULD BE MEASURED BEFORE DESIGNING AN OVERLAY
WINDOW AROUND AN ESTIMATE.

KCC IS THE REQUIRED NORMAL DAIMOS C COMPILER.  CURRENT NORMAL USERSPACE ALREADY
BUILDS WITH KCC/DAS WITHOUT LIBGCC.  A MONOLITHIC DAIMOS-RESIDENT KCC MAY STILL BE A
LARGE-MEMORY PROGRAM, BUT THIS DOES NOT BY ITSELF JUSTIFY A KERNEL OVERLAY ABI.
BEFORE ADDING OVERLAYS, MEASURE REAL DAIMOS-RESIDENT KCC/DAS/EDITOR WORKLOADS AND COMPARE
AT LEAST:

```TEXT
MONOLITHIC EXECUTION ON A LARGE-MEMORY MACHINE
PROGRAM-LEVEL PHASE SPLITTING
SCRATCH-BACKED INTERMEDIATE STATE
USERSPACE/LOADER OVERLAYS
```

PREFER PHASE SPLITTING OR SCRATCH-BACKED WORKING SETS WHEN THEY ARE SMALLER,
SIMPLER, AND DO NOT HARM NORMAL EXECUTION SPEED.

IF OVERLAYS ARE LATER REQUIRED, KEEP THEIR POLICY PRIMARILY IN USERSPACE AND
LOADER SUPPORT.  DO NOT ADD A GENERAL KERNEL OVERLAY WINDOW OR STRING-HEAVY
KERNEL SYMBOL RESOLUTION.

GENERAL RUNTIME KERNEL OVERLAYS ARE NOT THE PREFERRED V0.9 MECHANISM.  DEMAND
MODULES PROVIDE KERNEL MEMORY SAVING WITHOUT A SINGLE SHARED OVERLAY WINDOW AND
ALLOW SIMULTANEOUSLY ACTIVE FILESYSTEMS AND DRIVERS.

BOOT-ONLY MINIT CODE REMAINS DISCARDABLE AND SHOULD CONTINUE TO BE RECLAIMED
AFTER INITIALIZATION.

## 14. FUTURE PAGER BACKEND

THE PROCESS ADDRESS-SPACE/VM BOUNDARY IS A REQUIRED ARCHITECTURAL ABSTRACTION.
GENERIC PROCESS, EXEC, SCHEDULER, VFS, AND OTHER KERNEL CODE MUST NOT DEPEND ON
PDP-6 PHYSICAL-BASE ARITHMETIC OR THE CONTIGUOUS-PHYSICAL-IMAGE REPRESENTATION.

THE CURRENT IMPLEMENTATION ALREADY PROVIDES THIS BOUNDARY THROUGH THE VM
INTERFACE.  THE PDP-6 BACKEND MAY USE CONTIGUOUS EXTENTS, RELOCATION/PROTECTION,
COMPACTION, AND WHOLE-IMAGE SWAPPING, WHILE A LATER MACHINE-SPECIFIC BACKEND MAY
REPRESENT THE SAME PROCESS ADDRESS SPACE WITH A HARDWARE PAGER.  BACKEND-PRIVATE
STATE MUST REMAIN BEHIND THIS INTERFACE.

USERLAND HEAP GROWTH USES THAT SAME VM BOUNDARY.  BRK IS THE ONLY KERNEL
USER-BREAK OPERATION; SBRK IS A LIBC WRAPPER.  THE PDP-6 BACKEND ROUNDS THE
PHYSICAL EXTENT TO 02000-WORD APR GRANULARITY, RETAINS THE EXACT LOGICAL BREAK
SEPARATELY IN THE STABLE U-AREA, AND CAN GROW A HIGH-PLACED CURRENT PROCESS BY
MOVING ITS EXISTING IMAGE DOWNWARD WITH FS_MOVE_WORDS BEFORE RELOADING APR.
RUN AND EXEC ESTABLISH A NEW BREAK FLOOR AT THE END OF THE COMPLETE INITIAL
PROCESS VM INCLUDING THE FIXED STARTUP/STACK RESERVATION.  THIS ADDS NO WORDS
TO THE THREE-WORD RESIDENT PROCESS DESCRIPTOR.

KEEP THE ABSTRACTION NARROW AND COMPACT.  IT MUST NOT REQUIRE GENERIC PAGING
MACHINERY ON THE PDP-6, AND IT MUST PRESERVE MACHINE-SPECIFIC FAST PATHS.  ADD
PAGE-GRANULAR OPERATIONS ONLY WHEN A SUPPORTED PAGED TARGET REQUIRES THEM.

THE PERMANENT PRINCIPLES THAT SHOULD SURVIVE A PAGER BACKEND ARE:

```TEXT
KERNEL NUCLEUS PROTECTION
PROCESS/MM DESCRIPTOR OWNERSHIP
PINNING FOR PHYSICAL I/O
MODULE LIFETIME REFERENCES
COMPACT MODULE SERVICE ABI
MEASURED MEMORY ACCOUNTING
```

## 15. ACCOUNTING

MM SHOULD EXPOSE SMALL NUMERIC COUNTERS, NOT KERNEL FORMATTING STRINGS.  USEFUL
VALUES INCLUDE:

```TEXT
TOTAL/FREE PHYSICAL WORDS
LARGEST FREE EXTENT
COMPACTION COUNT
WORDS MOVED
SWAP WORDS READ/WRITTEN
LOADED MODULE WORDS
AUTOLOADED MODULE COUNT
MANUAL-RESIDENT MODULE COUNT
ALLOCATION FAILURES
```

USERLAND ADMIN TOOLS SHOULD FORMAT THESE VALUES.

## 16. V0.9 SIZE DISCIPLINE

ALL FIXED RESIDENT AND IRREDUCIBLE SCRATCH KERNEL MEMORY MUST REMAIN BELOW 16K
WORDS.  THIS IS THE ACTIVE ARCHITECTURAL LIMIT.  KCC PERMANENT-SIZE ACCOUNTING
MUST BE RE-RUN AFTER MATERIAL KERNEL CHANGES; EXACT RESULTS ARE DATED
OBSERVATIONS, NOT STANDING SHRINK TARGETS.  ELASTIC RECLAIMABLE CACHE BORROWED
FROM MANAGED CORE IS NOT FIXED KERNEL IMAGE MEMORY, BUT ITS CODE AND
NONRECLAIMABLE METADATA ARE.

THIS MUST NOT BE GAMED BY MOVING EVERY REQUIRED CORE FACILITY INTO A MODULE AND
COUNTING ONLY THE NUCLEUS.  SPECIALIZED OPTIONAL HARDWARE OR FILESYSTEM MODULES
MAY INCREASE AN "EVERYTHING LOADED" CONFIGURATION BEYOND THE NORMAL V0.9
RESIDENT SET.
