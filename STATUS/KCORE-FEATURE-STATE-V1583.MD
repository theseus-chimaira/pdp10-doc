# KCORE P1-C v1583: full physical mapped snapshot PI2 fail-stop

Date: 2026-11-15. **ACCEPTED** full 128K KCC PDP-6 with production
four-member DR0..DR3 D6FS root VOLUME, physical mapped BACKSTORE,
persistent MEMFS/XIP, LOGSTORE LSREC1 and four independently sleeping
24K-word swapped children. P0-A and trusted-operator P0-B remain
CLOSED; P1-C remains OPEN. VERSION 0.5, strict 32K optimization HOLD.

Earlier v1547/v1548 already proved fatal BACKSTORE-first and
ROOT-first errors on the same physical four-member DRM root while
swapped backing was owned. v1581 verified snapshot HEADER PI2 failure
with direct AUXSTORE backing. **v1583 specifically verifies their
intersection:** a persistent MEMFS shutdown itself attempts a real
snapshot commit-header WRITE through the production four-member
BLOCKSET BACKSTORE mapper with original positive swap ownership.

An isolated copy of the actual MEMFS MRES requires original KCORE
`proc_swap_blocks_used` positive at shutdown. Reused v1581 disposable
diagnostics select the first invalid header WRITE to the last logical
BACKSTORE block, bind the actual Type-236 descriptor and redirect
only its real DR_DONE PI2 interrupt through the production driver
error leaf. The original v1541 critical BACKSTORE gateway HALTs at
the linked v1515 reason-2 instruction **`004011` octal**. No error
status is forged at the storage gateway, no permanent source changes.

The independent host verifier parses all four 8192-sector, 128-word
36-bit Type-236 member images and each D6FSR2 sector-zero descriptor.
Every member has base `1`, filesystem span `017377`, BACKSTORE tail
`0400` and physical tail end `017777` octal. The original KCORE
reports mapped filesystem span `075774`, total BACKSTORE capacity
`02000`, real D6FS member-set `032104`, selected root vnode
`060101000000`, original swap ownership `01374` and original DRM
error count `1`. The last logical BACKSTORE sector `01777` goes to
**DR3 block `017777`**, and the real selected Type-236 group word
`777770` matches `(3 << 16) | (017777 << 3)` exactly. The snapshot
header remains zero/invalid, 248 other real physical tail sectors
have nonzero data, the request descriptor retired, pending selector
cleared, and all four physical D6FSR2 descriptors stayed unchanged.
An abrupt HALT can legitimately leave a DIRTY D6FS superblock; that
is not treated as damaged root layout. Twelve deliberately corrupted
linked-word, owner, VOLUME, PI2, root and media variants were all
REJECTED by the actual independent postflight verifier.

This selected injected PI2 branch follows a true hardware-simulator
DMA completion, not natural damaged magnetic media. The test does
not prove an independently scheduled process swap-in completed
*during* synchronous snapshot DMA or successful recoverable MM/
BCACHE/process-owner rollback on another healthy provider. **P1-C
remains OPEN** for those independent acceptance tests, P1-D for
alternate physical inventories, and P1-E for uncompressed PURE.
No production code, fixed RAM, process count, ABI, partition format
or capacity change. Original KCORE low end `024364` octal.

Detailed source record:
`DAIMOS/design/kcore-p1c-physical-mapped-snapshot-v1583.md`.
Testkit reproduction:
`tests/validation/daimos-p1c-mapped-snapshot-pi2-physical-v1583.sh`,
companion `.py` independent oracle and `.md` methodology.
