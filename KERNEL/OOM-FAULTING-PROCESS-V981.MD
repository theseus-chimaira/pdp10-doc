# OOM policy: terminate only the faulting process (v981)

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
