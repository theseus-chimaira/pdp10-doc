# KCORE K04 v1801: bounded teardown recovery acceptance

Date: 2026-11-22. **K04 CLOSED** for the present implementation.
This is an engineering acceptance of owner-safe recovery, not
an exhaustive proof of every possible hardware failure.

## Runtime contract

The process that owns a private fatal continuation completes
FILE close and session HUP before a deferred, allocation-free
MM finalizer may retire the slot. A refused VM, u-area, FILE
or TTY resource remains owned. The reaper distinguishes
published PREVM EXIT, post-VM FINALIZE, SIDL DISCARD and
unpublished VM_NONE DISCARD rollback. It retries with bounded
exponential delay; a permanently refused resource remains
quarantined, not freed speculatively. The scheduler and other
processes remain available for operator intervention.

Zombie publication is permitted only when a WAIT-capable
parent exists. A dying original parent cannot receive a late
child; healthy INIT can adopt it, and an ineligible INIT cannot.
If no eligible parent exists after actual child resource release,
the terminal child becomes FREE directly, not an uncollectable
ZOMB. Live children retain their resource ownership until
their own cleanup has finished.

## Acceptance evidence

The accepted 256K PDP-6 physical experiments are documented
in DAIMOS `design/kcore-k04-closure-v1801.md` and its versioned
predecessors. They include temporary and permanent real MM
pins, independent two-owner retry, SIDL/PREVM/post-VM orderings,
single-owner 17-bit clock wrap, fatal FILE-close refusal,
session HUP with FILE and VM_NONE DISCARD owners, and both
healthy and ineligible INIT orphan recovery.

The v1800-A guest completed late-child adoption/WAIT while
the original parent still had a real pinned VM. The v1800-B
guest held INIT on its own private fatal-FILE close and PID3
on a real PREVM MM pin. PID4 was reclaimed directly once,
with no zombie publication. Both used independent privileged
guest HALT and returned read-only D6FS state=CLEAN.

`daimos-testkit/tests/validation/daimos-k04-closure-regression-v1801.sh`
rechecks ten independent verifier self-tests and their negative
controls, plus the actual v1800-A/B source-linked post-HALT
memory witnesses and clean disk images. The v1801 closeout
regression PASSed. No permanent KCORE RAM or ABI changes were
needed for v1800-A/B or the closeout.

## Scope and remaining risks

No separate physical test combined two independent MM owners
with the 17-bit clock wrap, combined published PREVM/post-VM
physical pins with session HUP, or caused an organic controller
or disk-media fault. Those untested combinations are **not**
claimed to have passed. Revisit them only if a related defect,
provider change or operational incident warrants it.

K04 closure does **not** enable automatic OOM victim killing.
K03 was subsequently CLOSED at v1803 by explicitly rejecting
cross-process victim killing; see
`KERNEL/KCORE-K01-K04-CLOSURE-V1803.MD`. Existing KCORE documentation
with earlier dated `K04 OPEN` statements records historical
intermediate states; this v1801 closure supersedes those
statements for the current implementation.
