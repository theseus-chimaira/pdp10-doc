# KCORE v1580: physical swapped owner at persistent MEMFS HALT

Date: 2026-11-14. Physical P1-C integration increment accepted,
not full combined-fault or truly simultaneous PI/snapshot closure.
P0-A and trusted-operator P0-B CLOSED; VERSION 0.5 and 32K/
optimization HOLD.

Two separate full 128K KCC PDP-6 cold boots operated on the identical
retained 8 MiB Type-236 AUXSTORE drum and fresh CPU state. A
test-only user program created and ran original restricted MEMFS
XIP, checked a persistent data file and appended/read a real LSREC1
through LOGCTL. It launched four independently scheduled 24K-word
private-image children and confirmed each was sleeping before its
own privileged HALT. A disposable copy of the original MEMFS
snapshot MRES checks the production fixed KCORE
`proc_swap_blocks_used` at shutdown entry and refuses HALT if zero.
Both physical guests completed the original checked snapshot
physical writes and halted at PC octal `021777`, proving an actual
outstanding process BACKSTORE owner when each commit started.

An independent raw Type-236 media oracle located BACKSTORE and
LOGSTORE from their existing physical AUXSTORE descriptor, checked
the nonoverlapping ranges, decoded actual complete 128-word sectors,
and verified the valid final v2 snapshot header. The real effective
MEMFS pool on this guest is 3,648 words (not the requested 4,096),
requiring 35 reserved backing sectors, octal 07735..07777, ending
at BACKSTORE's boundary. 248 nonzero physical backing sectors lay
outside the snapshot. Both boot inspections retained the original
AUXSTORE descriptor byte-for-byte. Boot two restored exact MEMFS
data/XIP and found the earlier committed physical LOGSTORE record,
then appended a second valid record and repeated the swapped HALT.

This differs from the v1579 test-double I/O interleaving: actual
process swap remains owned on genuine physical backing during
HALT. The execution still does NOT demonstrate that a separately
scheduled process's PI-driven swap-in was dispatched between
snapshot I/O completions. Nor does it inject natural controller
loss, mapped DEVICE/VOLUME failure or BCACHE/PI rollback in-flight.
**P1-C remains OPEN** for those proofs; P1-D alternate real mount
inventories, P1-E uncompressed PURE and final feature freeze remain
open. No production KCORE/MRES, MM allocator, ABI, process table or
persistent media format changes; `__kcore_low_end` stays `024364`.

Source details: DAIMOS
`design/kcore-p1c-physical-swap-snapshot-v1580.md` and testkit
`tests/validation/daimos-p1c-swap-snapshot-physical-v1580.md`.
