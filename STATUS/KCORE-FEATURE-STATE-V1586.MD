# KCORE v1586: real swap READ PI2 fault DURING independent snapshot overlap

Date: 2026-11-15. Accepted a full 128K KCC PDP-6 physical guest
with four real Type-236 DR0..DR3 D6FS root VOLUME members, original
BLOCKSET INTERLEAVE mapped BACKSTORE, LOGSTORE and persistent MEMFS.
This is **original BACKSTORE fail-stop correctness when the failure
occurs in a genuine independently scheduled process swap READ during
MEMFS snapshot DMA**, not recoverable MM/BCACHE rollback.

Original PI6 wakes a real long-sleeping, disk-backed child after an
opt-in disposable test-only snapshot-entry timer adjustment. The
original slot-0 scheduler starts the real swap READ. Its actual
physical Type-236 DR_DONE PI2 for the original selected descriptor
is directed into the original DRM error handler, which publishes
the error and retires the descriptor. The unmodified critical
BACKSTORE gateway reaches the original v1515 reason-2 HALT at
**octal `004011`**. No synthetic syscall return, direct HALT,
fabricated DMA or process state/run-queue modification occurs.

Exact KCORE post-HALT values:

| Witness | Value |
| --- | ---: |
| Original sleeping swap slot/timer PI6 wake | 6 / 1 |
| Original independent swap-in starts and physical READs | 1 / 1 |
| Selected real matching DRM PI2 completion/error | 1 / 1 |
| Original total snapshot-window DRM PI2 | `0343` octal |
| Actual successful DISK-to-RESIDENT publications | 0 |
| Original DRM driver error counter | 1 |
| Original generic logical storage errors | 0 |
| Remaining original process-owned swap blocks | `01650` octal |
| Remaining original BACKSTORE allocation | `01734` octal |
| Original snapshot in progress at fatal HALT | yes |
| Original selected descriptor and one-shot arm | both cleared |

The actual hardware group `0375060` targets DR1 physical block
`017506` = original INTERLEAVE logical BACKSTORE block `000431`.
All four original D6FSR2 on-drum member descriptors are unchanged,
the selected physical sector contains actual backing payload,
268 other BACKSTORE physical sectors contain nonzero data, and
the original final MEMFS v2 commit header at DR3 block `017777`
remains invalid. Fifteen deliberately falsified PI/driver/owner/
physical/header/descriptor witnesses were rejected by the
independent physical oracle. Existing v1583 mapped snapshot-header
failure and v1585 two-boot healthy swap/snapshot overlap regressions
remain passing.

All v1586 changes are isolated to a disposable test kernel and
testkit harness. The original physical completion is authentic,
but the error is deliberately selected: it is not proof of naturally
damaged drum media. P1-C remains OPEN for recoverable owner/MM/BCACHE
rollback and additional fault timings/owner transitions, as well as
repeated-load timing stability from v1585. P1-D/E remain OPEN.
VERSION 0.5, production KCORE low end `024364` octal and strict
32K/optimization HOLD are unchanged; no permanent code/RAM/ABI or
media-format modification.

Source and test evidence: DAIMOS
`design/kcore-p1c-overlapped-swap-read-failure-v1586.md`, testkit
`tests/validation/daimos-p1c-swap-read-fault-physical-v1586.{sh,md}`
and retained `$TMPDIR/daimos-p1c-swap-read-fault-v1586.5lg2J3`.
