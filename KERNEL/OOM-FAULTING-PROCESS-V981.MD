# OOM policy: terminate only the faulting process (v981)

## v1075: full PDP-6 physical-OOM requester-only test

The testkit builds a disposable 256K PDP-6 kernel image with the
already-existing default-off `PROC_OOM_BRK_ENABLED_V1000` branch enabled.
A child requests BRK 0776000 octal, a valid address but too large to
allocate after normal memory reclamation. The child is terminated with
OOM exit status 1 through the real fatal-owner teardown path. The parent
waits for this result, then releases an unrelated child blocked on PIPE
input; that child finishes normally. Process-slot and process-word
accounting return to their initial values. The full guest outputs `<KWP`
and a controlled HALT without any failure marker.

The separately built default-off guest runs the same program. Its BRK
call returns -1 rather than killing the child, which prints `D` and
exits normally. The parent confirms the non-OOM exit status, releases
the independent PIPE-blocked witness, checks memory accounting and
prints the complete successful sequence `<DNWP`. This negative control
confirms the test would detect a missing OOM-kill branch without using
an intentional assertion failure as its expected result.

Both variants are isolated build/test images, not production flags.
Neither variant injects an indefinitely failing FILE/MM provider.
Bounded provider cancellation and a transitive no-allocation proof
through all teardown callbacks remain required before enabling the
production OOM killer.

## v1074: provisional BRK extent double-release invariant

The nonadjacent PDP-6 BRK growth path can temporarily hold both the old
running image and an uncommitted larger replacement image. Previously,
if `mm_free(old)` refused, the rollback called `mm_free(new)` but did
not inspect its result. A second refusal could leak the replacement
without a process-VM reference. The kernel now preserves the exact
old MM error in an existing stack local and checks `mm_free(new)`.
If the new extent also cannot be released, execution enters the existing
resident storage-invariant HALT rather than returning with an orphaned
allocation. This is fail-closed containment of an internal MM ownership
invariant, not an OOM termination policy or a new recovery allocator.

Native PDP-6 source-linked tests accept successful -4/-5 rollback and
the double-refusal guard. Pre-fix and deliberately unguarded controls
fail. GCC KCORE and full KCC boot-image builds pass; the latter reaches
LOGIN under SIMH. Full native `daimos-brk-v1` KCC 256K guest regression
PASSes BRK/SBRK growth, heap data preservation and shrink.
Provider quiescence, fatal teardown liveness and
whole-process physical-OOM acceptance remain open. Production OOM
termination stays disabled.

DAIMOS uses a simple OpenBSD-style OOM decision: only the process whose
user-space memory allocation cannot be satisfied is a possible termination
target. It must not kill another process because that process occupies more
resident memory or backing sectors. No victim-score heuristic is used.

The native allocator first attempts normal allocation, cache reclamation,
compaction and permitted swapping. Swapping a different process to free RAM
is not OOM killing. The former read-only `proc_oom_select_v863` scanner,
which selected the largest eligible resource owner, was unused in production
and is deleted. `proc_swap_victim` remains for swap-out decisions only.

Automatic OOM termination is still DISABLED. Allocation failure currently
returns an error rather than entering an unsafe kill/teardown path. Enabling
termination requires allocation-free and bounded owner-side FILE/TTY/VM,
provider, backing and parent-notification cleanup, plus real RUN and BRK
exhaustion tests with failures and concurrent provider ownership. Kernel
internal allocation failures must follow their own safe rollback paths;
they must never trigger an unrelated process kill.

The v981 256K PDP-6 image builds and links. The source-linked policy,
provider and teardown audits pass. Historical tests written for the removed
largest-process selector are superseded by the v981 policy and must not be
used as current acceptance for OOM killing.

## v982: RUN and BRK have different failure owners

For failed growth of an existing process's address space (BRK), the
requester is the only potential fatal OOM candidate. The current BRK
allocator returns an error after normal reclaim fails; automatic killing
remains disabled.

SYS_RUN allocates resources for a not-yet-published child. A failed child
VM or u-area allocation discards that child through `proc_slot_discard`
and returns an error to its parent. Do not kill the parent merely because
it requested creation of a child that cannot fit. Never publish an
incompletely initialized child.

Testkit v982 source checks assert these existing paths. They are not
substitutes for booted exhausted-RAM failure and teardown acceptance.

## v983-v984 booted BRK testing

After repairing a missing test-only `memset` link, the standard BRK/SBRK
growth, data-preservation and shrink guest passes on a 256K PDP-6.
A separate v984 guest repeatedly grows its own break until a request
fails, verifies that the break remains unchanged, shrinks to the initial
break and exits normally. This is a verified failure/rollback behavior,
not a demonstrated physical-memory OOM: a user-address limit could also
cause the failing request. Instrument MM failure cause before claiming
true RAM exhaustion. Automatic termination remains disabled.

## v985: physical-memory pressure accepted

An isolated clean v197 RUN fixture with the existing v198 physical-OOM
extension passes on a booted 256K PDP-6. Four valid 18-bit BRK growth
requests fail under physical-MM pressure while preserving the caller's
break, VM size, data and RUN rollback/recovery. This provides a genuine
physical-pressure test distinct from the unclassified v984 repeated-growth
failure. It still does not authorize automatic termination; allocation-free,
bounded owner-side teardown is not yet certified.

## v986: owner-private fatal FILE retry audit

Native PDP-6 v926 validates fatal TERM owner-return FILE retries; current
v986 static test verifies that the owner stack performs FILE close, preserves
resources on refusal and sleeps between retries with an interval capped at
01000 octal ticks. A capped interval is not a cap on retry count. Permanent
provider refusal can prevent reclamation indefinitely. The owner-private
teardown, provider recovery and allocation-free closure graph must be proved
before any automatic OOM termination is activated.

## v987: permanent-provider containment

The source-linked v987 test verifies that the serialized provider lease is
only released by the normal owner return path. Recursive acquisition refuses,
foreign waiters sleep and retry acquisition, and owner-private fatal FILE
retry neither forces an unlock nor releases the u-area. This is containment,
not proof of a quiescence/cancellation protocol for indefinitely stalled
providers. Automatic OOM termination remains OFF.

## v988: pre-VM owner-exit safety boundary

Full-system PDP-6 v978 mixed fatal retry passes, including two failed FILE
closes and one u-area MM refusal with successful retry. A separate v988
source-linked test confirms that the current ordinary owner EXIT has a
specific limitation for OOM: `proc_exit_finish` detaches the current slot
on the idle stack; post-VM MM refusal is deferred, but earlier HUP or VM
refusal returns failure and leads to diagnostic HALT 1. Do not connect
ordinary exhausted-user-allocation handling to this exit path until its
recoverable pre-VM failures have a safe owner-preserving completion route.

## v989: pre-VM work must retain a schedulable continuation

The existing slot-zero retry is a post-VM MM finalizer; its scheduler
context inhibits PI6. Pre-VM session HUP and VM/provider callbacks may
sleep and therefore cannot simply be moved to that retry path. Native
source-linked testkit v989 protects this boundary. A safe requester OOM
exit needs a valid owner continuation through any sleeping work, followed
by the current non-sleeping final MM release after leaving its stack.
No OOM killing is enabled by v989.

## v991: live-owner HUP stage

Ordinary EXIT now completes the potentially sleeping session-HUP preflight
on its existing private kernel stack after FILE closure and before switching
to slot zero. On refusal it returns -1 without discarding the owner's VM or
stack. The existing finalizer continues to recheck session state. A full
256K PDP-6 BRK runtime and the v991 source guard pass. The older v905
standalone HUP fixture failed to link (missing `proc_tty_line_try_reset_v897`)
and is not counted as a passing HUP test. VM destruction is not yet staged;
OOM termination remains disabled pending fault-injected exit acceptance.

## v992: native HUP fixture recovery

The historical native v905 HUP retry fixture has been repaired with a
proper text-section test-only TTY reset callback. Current deferred
VM_NONE+FINALIZE HUP retry PASSes, while the old-behavior negative
control fails as expected (exit 010). This is not yet a booted v991
EXIT preflight fault-injection test. Both existing OOM P0 gates remain open.

## v993: owner EXIT preflight refusal

The native PDP-6 testkit v993 executes the production v991 EXIT branch,
with test-only FILE/HUP helpers. The first HUP refusal returns -1 without
entering the permanent-stack exit path; a second attempt reaches normal
exit handoff. This control-flow proof passes, but does not certify the
full-system provider/VM cleanup path or activate automatic OOM termination.

## v995: fatal owner-side HUP retries

The APR/EXEC fatal exit performs session-HUP cleanup on the live private
stack following successful FILE close. A temporary HUP refusal retains the
owner continuation, sleeps with the pre-existing backoff, and retries.
Native PDP-6 v970 and fault-injected v995 regressions pass. No additional
resident state was introduced. VM-destroy failure after the permanent-stack
handoff still prevents automatic requester OOM termination; both existing
OOM P0 tasks remain open.

## v997: pre-VM EXIT release retry

Production EXIT now distinguishes pending pre-VM release from the existing
post-VM u-area finalizer using bit 33 of the otherwise-used finalizer
u-area word. A failed VM release retains TRANSITION, identity, and saved
exit status; slot zero retries only this marked, non-sleeping release path.
After VM destruction succeeds the PREVM mark is cleared and existing
u-area release/finalization resumes. No permanent retry table or extra
kernel stack is introduced. Full-system v997 injected VM rejection then
MM u-area refusal, plus unmodified v978, and native v881/v869 PASS. These
are teardown tests, not full physical-OOM requester-killing acceptance;
automatic OOM termination remains disabled until that proof is complete.

## v998: classify failed BRK before requesting termination

The allocator distinguishes insufficient total free words after reclaim
(-1) from fragmented free space (-2), descriptor exhaustion (-3),
invalid arguments (-4), and busy/pinned extents (-5). Current BRK merges
all allocation and validation rejections into its ordinary -1 return;
this is not sufficient evidence to kill its caller. Source contract v998
PASSes and booted v985 RUN/BRK failure-and-recovery regression PASSes on
the v997 kernel. The next isolated test must preserve the allocator cause
at the failed BRK call and terminate only for real physical shortage.
The production killer remains off and the two existing P0 gates remain.

## v999: retain the BRK allocator cause

On failed PDP-6 BRK growth, the VM helper now preserves the internal MM
failure code in AC2 while retaining the existing user-visible AC1=-1.
Local validation failures are -4, and old-extent release refusal is -5.
Only MM -1 indicates insufficient aggregate free memory after reclaim;
fragmentation, descriptor capacity and busy failures remain distinct.
This adds no permanent RAM and changes no syscall ABI. Source gate v998
(updated for v999) and the booted BRK/sbrk growth, data and shrink test
pass. Automatic OOM killing remains disabled pending a full-system
physical-OOM requester termination and reclamation test.

## v1000: default-off requester termination branch

PDP-6 BRK now retains its existing syscall ABI, with an optional
compile-time branch (`PROC_OOM_BRK_ENABLED_V1000`, default 0) that enters
the already-established fatal-owner cleanup only when both AC1 and AC2
are -1 (failed BRK and MM_ERR_NOMEM). The isolated native v1000 test
activates this branch and verifies successes, invalid and fragmented
failures return normally, while simulated physical shortage enters the
owner fatal path. The unchanged full-system BRK growth/data/shrink test
passes with production behavior disabled. Genuine physical shortage,
whole-process cleanup and unrelated-process survival are not yet proven
in one booted test; the automatic OOM path remains off.
