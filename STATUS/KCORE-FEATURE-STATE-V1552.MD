# KCORE v1552: physically pinned MM scratch for DTFS write intent

Date: 2026-11-13. P0-B staged KCORE feature, still OPEN.

The standalone DTVOL2 v1551 physical RBN0 write-intent arming
now has a verified temporary MM scratch lifecycle. The versioned
`dtfs_intent_owned_arm_v1552.s` wrapper uses the production
six-argument no-reclaim MM allocation for exactly 128 words,
MM_TYPE_KERNEL_DYNAMIC=3 and private owner 015 octal. The
temporary extent is **pinned** across the entire v1551 Type-551
physical WRITE/readback, then unpinned and freed. A refused
MM cleanup returns failure and retains the exact physical base
in a caller-owned pointer; the wrapper will not overwrite a
pre-existing nonzero pointer or reuse somebody else's extent.

This is necessary because an unpinned MM extent could relocate
while real Type-551 DMA sleeps. No new fixed core data words,
permanent MM slabs, syscall ABI or default media-format change
are introduced by this standalone wrapper.

Testkit PDP-6 source-linked validation PASSes **11 strict-MM
boundary scenarios** and rejects **10 unsafe source mutations**
(missing pin/unpin/free, wrong owner or size, omitted arm,
discarded cleanup owner, ignored pin error). A second independent
native test executes **real production MM allocator, fit/extent,
pin/unpin and free instructions** with the actual wrapper.
It completes two exact allocations and frees, confirms the
physical buffer's pinned owner/size, and refuses a third
transaction after eliminating the free arena. PASS.

These tests use simulated raw Type-551 completion, even though
the MM source is genuine. The earlier v1551 separately tested
the actual raw RBN0 arming code against controlled I/O; they
are not a booted combined physical Type-551 acceptance.

**P0-B remains OPEN**: the new MM wrapper is still unlinked
from regular mounted DTFS. DTVOL2 CLEAN admission and mandatory
first-write arm, provider-retained MM owner, exclusive final
CLEAN/unmount and real partial-media reinsertion/cold-boot
testing have not been completed. P0-A complete, P1-C separate;
32K/optimization HOLD.

DAIMOS: `design/kcore-dtfs-owned-scratch-v1552.md`.
Testkit: `tests/validation/daimos-dtfs-owned-scratch-v1552.md`.
