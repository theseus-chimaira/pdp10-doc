# KCORE v1561: production DTFS DIRTY intent/MM ownership bridge linked

Date: 2026-11-14. **P0-B OPEN**. DTVOL2 remains READ-ONLY;
no physical DTVOL2 data/directory write is enabled by this stage.

The actual optional production DTFS MRES now links the complete
previously standalone native v1551-v1554 RBN0 WRITE_INTENT
arm/readback, already-armed physical identity verification,
MM alloc/pin/unpin/free wrapper and authenticated first-write
gate. The v1560 provider-private per-mount scratch owner is a
word in the existing MM-backed six-array mount slab, shared
with D6FS reader ownership only on disjoint provider slots.

The new `dtfs_intent_mount_gate_v1561` is called by the existing
production `dtfs_dtc_write` DATA/DIR veneer. It resolves the
physical Type-551 unit to a unique published VFS DTFS V2 mount
using v1557 exclusivity, verifies the original public mount ID,
rejects non-RW state, and passes the exact MM-backed `aux[slot]`
word as the retained `basep` to the real v1553 gate. After any
sleeping MM/physical I/O it requires the original published
root, expected media flags, still-RW VFS policy and completely
released scratch owner before authorizing an ordinary write.
The caller saves the physical UNIT/BLOCK/BUFFER across this
operation on the private process stack and never invokes a
DATA/DIR raw write on gate failure. The raw RBN0 marker helper
still uses a separate physical WRITE entry to avoid recursion.

**Safety cutover remains disabled.** The v1555 mandatory V2 RO
write guard still runs *before* the new bridge, so every currently
admitted V2/CLEAN mount rejects DATA/DIR writes before any physical
DIRTY marker operation. Neither writable V2 admission nor an
exclusive CLEAN/unmount protocol has been enabled. Integration
tests use simulated hypothetical RW V2 to exercise the native
bridge and lower callback ABI, not real writable mounted tapes.

Testkit source-linked native acceptance: **35 actual PDP-6
mount/intent/owner bridge cases PASS, 13 unsafe mutations
rejected**. Actual central v1555 veneer **12 cases/9 mutations
PASS**, including two mutations that lose the pending block or
buffer when the lower gate clobbers AC2/AC3. Existing v1554
already-armed gate **23 cases/8 mutations PASS**; real native
v1552 MM allocation/pin/free PASS; v1559 source write-path audit
**11 callsites and 5 mutations PASS**. Complete KCC full-profile
KCORE and optional DTFS MRES assemble, fold and link PASS.

No new fixed KCORE word or VFS mount-slab word, syscall ABI,
physical on-media layout or default format. Optional MRES
executable grows to include the previously standalone native
physical/MM services. There is no optimization in this stage.

P0-B completion still requires an exclusive owner-quiescent final
CLEAN marker/unmount across sleeping physical I/O, a physical
media-generation/exchange safety contract for marker-to-data
transfer, and actual Type-551 partially written data, damaged
original tape reinsertion, distinct replacement and cold-boot
guest acceptance. Never remove the v1555 RO veto before these.

P0-A complete; **P0-B OPEN**; P1-C separate. VERSION 0.5;
strict 32K/optimization HOLD.

Design: `DAIMOS/design/kcore-dtfs-mounted-intent-v1561.md`.
Tests: `daimos-testkit/tests/validation/
daimos-dtfs-intent-mount-v1561.md`.
