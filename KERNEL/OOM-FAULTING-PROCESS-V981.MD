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
