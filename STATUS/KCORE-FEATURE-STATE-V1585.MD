# KCORE v1585: independent PI6 swap-in during mapped snapshot PASS

Date: 2026-11-15. Accepted two full 128K KCC PDP-6 cold boots using
genuine four-member DR0..DR3 Type-236 D6FS root VOLUME, original
BLOCKSET INTERLEAVE mapped BACKSTORE, persistent MEMFS v2/XIP and
LOGSTORE. VERSION 0.5, fixed production KCORE low end `024364` octal.

Exactly one previously sleeping, disk-backed process had its
existing scheduler timer deadline shortened to the next clock tick
at snapshot entry by an **opt-in disposable test-only** source hook.
The hook did NOT call swap-in, make the process runnable, manipulate
the run queue, issue PI6 or fake controller I/O/completion. The
original 60 Hz PI6 timer service woke it and the original slot-0
scheduler independently performed two actual mapped BACKSTORE READ
swap-ins and two completed DISK-to-RESIDENT state publications
**during the synchronous MEMFS snapshot**, on BOTH physical boots.

Original linked post-HALT evidence:

| Witness | Boot 1 | Boot 2 |
| --- | ---: | ---: |
| Timer selected/swapped slot | 6 | 6 |
| Original PI6 timer wakeup count | 1 | 1 |
| Independent slot-0 swap-in starts | 2 | 2 |
| Successful resident publications | 2 | 2 |
| Actual mapped swap READ requests | 2 | 2 |
| Matched DRM236 swap-READ DR_DONE PI2 | `0700` | `0700` |
| All snapshot-window Type-236 DR_DONE | `01311` | `01311` |
| Original clock ticks in snapshot | 2,129 | 2,137 |
| Original swap-owned blocks at HALT | `0750` | `0750` |
| Original backing blocks used at HALT | `01034` | `01013` |
| Original storage errors | 0 | 0 |

Both native privileged HALTs at octal `022004` completed normally.
Boot 2 physically restored the earlier MEMFS/XIP data and LOGSTORE
record and appended/read a new valid record. Independent raw-sector
verification confirms the v2 snapshot header at DR3 `017777`, true
3,648-word MEMFS pool, 275 other nonzero backing sectors and all
four byte-identical D6FSR2 member layouts. Seventeen deliberately
bad PI/timer/physical/owner/restore witnesses were rejected.

One inherited v1584 verifier defect was corrected: its negative
evidence controls must use strict overlap acceptance whenever the
baseline actually proves a complete physical overlap. The original
v1584 healthy zero-overlap baseline continues to pass normally, and
its strict mode continues rejecting an unproven overlap.

**Controlled independent PI6 swap-in during MEMFS snapshot is now
accepted** as one P1-C subgate. Remaining P1-C: exact recoverable
MM/BCACHE/owner rollback and combined fault/overlap scenarios. The
test-aligned timer deadline is deliberate; no natural coincidence or
real destroyed drum is claimed. P1-D/P1-E, final freeze and strict
32K/optimization HOLD remain OPEN. No production kernel code, RAM,
ABI or physical media-format changes.

Engineering evidence: DAIMOS
`design/kcore-p1c-independent-snapshot-swap-v1585.md` and testkit
`tests/validation/daimos-p1c-snapshot-timer-physical-v1585.md`.

One intermediate independent 180-second probe timed out after partial
snapshot writes without captured CPU state, so slow execution versus
rare scheduler stall remains undetermined. A following complete
two-boot runner PASSed with all 17 negative controls (2,136/2,140
snapshot ticks), and its physical evidence is retained at
`$TMPDIR/daimos-p1c-snapshot-timer-v1585.xYkqo8`. The v1585-only
probe now permits a bounded 300 seconds and captures CPU/PC on future
timeouts. Repeated-load stability remains a follow-up, not a claim of
proven absence of timing-sensitive bugs.
