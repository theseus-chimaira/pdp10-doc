# KCORE K02 process-owned resource cleanup: CLOSED (v1730)

Date: 2026-11-20. Completed production source: DAIMOS v1720
5d6e91c1; final acceptance baseline DAIMOS v1729 38e6b4d1,
testkit v1730 6be1309.

K02 covers the process lifetime and error propagation of existing
MM, VM, FILE descriptors, anonymous PIPE, named FIFO, and backing
records during ordinary/fatal EXIT, destructive EXEC and failed or
unpublished RUN. A close or free refusal must leave a discoverable
owner, retain the correct process/u-area or private fatal continuation,
and permit safe, bounded retry without double release. An owner
that cannot safely be recovered must fail closed. This completion
does NOT add automatic OOM victim selection or forcibly reclaim
a permanently refusing provider's resource.

## Reproducible acceptance

The daimos-testkit v1730 final acceptance driver independently
passed nine source-linked native PDP-6 gates:

- v1719 transactional staged EXEC owner and backing rollback.
- v1720 irreversible low-memory EXEC, VM_NONE and stable u-area.
- v1722 rejected unpublished RUN, duplicate PIPE FD references
  and independent FILE/PIPE close failures.
- v892 unpublished RUN failed MM free and exact-once reaper release.
- v1728 repeated session HUP with retained FD and attached TTY.
- v970 foreign HUP protection for the private fatal stack.
- v1099 typed MM -4/-5 PIPE/FIFO owner release.
- v1102 bounded reaper sleep and retry.
- v922 repeated partial session HUP and committed-member skipping.

All nine tests passed, including each existing unsafe mutation
control. In addition, four isolated full-system 256K PDP-6 tests
returned success with exact physical owner/refusal/WAIT witnesses:

| Guest | Key physical result |
| --- | --- |
| v1721 | Actual PROCESS MM freed before two backing-detach refusals; no second MM free; stable record, third accepted detach and parent WAIT |
| v1726 | Concurrent RUN D6FS and PIPE owners; two live cross-process checks, one real MM_BUSY, exactly one final free |
| v1729 | Permanent D6FS provider refusal, three refused session HUPs retaining TTY, independent low-memory post-VM_NONE fatal EXEC/WAIT |
| v900 | Real named FIFO MM pin/BUSY, preserved namespace/reopen, successful unpin/retry |

The combined v1729 verifier additionally rejected six unsafe
transcript modifications. A full SHA-256 log list and exact counters
are retained in the versioned testkit
tests/validation/daimos-k02-closure-v1730.md and DAIMOS
design/kcore-k02-closure-v1730.md.

The prior v1729 OPEN note about physical post-MM-free backing-detach
refusal is superseded: v1721 already completed that exact physical
case and it passed the fresh v1730 rerun.

## RAM and strict profile limits

K02 production fixes grew KCORE by 23 executable words in v1719
and 7 in v1720 (total 30), with zero new permanent or per-process
RAM words. There have been no production kernel source changes
since v1720; the v1721-v1730 tests and documentation add no
production resident memory.

The last matched GCC low-memory disk-root build of these identical
production inputs measured KCORE 10,461 words, KCORE+MRES 17,186
words and last permanent address 17,233 decimal. The separate
strict 040000 octal 32K permanent-address gate remains OVER BUDGET,
with -850 free-word accounting. K02 closure does not accept that
profile or certify 48K residency.

## What remains open

K03: automatic OOM victim termination is still DISABLED pending
its own selection, policy and complete teardown acceptance.
K04: recovery policy for permanently refused provider/backing
owners, retained zombies and offline repair is still OPEN. Do not
force-unpin, force-free or discard an unproven owner/private stack.
Actual removable-media and filesystem/provider-specific faults
are distinct implementation responsibilities, not K02 lifetime
work. Unrelated KCORE memory targets and other feature work are
not closed by this sign-off.
