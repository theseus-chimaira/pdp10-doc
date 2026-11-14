# KCORE v1554: physical DTFS DTVOL2 DIRTY state revalidated

Date: 2026-11-14. Native PDP-6 source acceptance, **P0-B OPEN**.

v1553 could authorize another DATA/DIR write from an in-memory
INTENT_ARMED bit without confirming the tape remained the original
DTVOL2/WRITE_INTENT volume. That is insufficient for removable
Type-551 DECtape, especially after an earlier partly completed
write and administrative discard. v1554 corrects this in the
still-unlinked DTFS durability component.

New native `dtfs_intent_verify_v1554.s` always reads the physical
reserved RBN0, bypasses BCACHE, clears its independently pinned
scratch before I/O to reject stale false successful reads, and
requires an exact v1550 DTVOL2/DIRTY seal and both saved 36-bit
volume-ID halves. It does not physically write anything. On a
short/failed read, CLEAN/old/torn header or changed ID, it
invalidates BCACHE and marks the mounted physical unit sticky
WRITE_UNCERTAIN. A recoverable READ_SUSPECT is insufficient
because matching a stamp alone does not restore dirty intent.

The existing versioned v1552 MM wrapper now shares its same
0200-word owner 015 octal extent, real MM pin across Type-551
I/O, unpin/free and retained-failure handle with a new VERIFY
entry. The v1553 gate physically rechecks DIRTY for every
already-armed request, then compares the original mount media
word, both saved identity halves and the scratch cleanup pointer
again after I/O before authorizing the next data transfer.

Testkit native PDP-6 acceptance:

- **17** raw physical-record tests PASS and **10** unsafe source
  mutations rejected, using actual host-generated `mkdtfs`
  DECtape bytes and controlled Type-551 completion;
- **23** authenticated mounted-gate tests PASS, **8** new unsafe
  source mutations rejected, including changed ID/media and
  retained MM owner after verification;
- actual production PDP-6 MM allocation, pin/unpin and free
  executes two VERIFY leases and real shortage refusal; **4**
  wrong-dispatch/pin/free source mutations rejected;
- original v1552 11-case/10-mutant and v1553 18-case/13-mutant
  regressions still PASS.

All these are standalone source-linked tests; the real controller
completions remain test-controlled, not a booted full-guest
Type-551 partial-write/reinsertion/cold-boot acceptance.

**Not deployed:** normal DTFS MRES still uses DTVOL1 admission
and has no mandatory DTVOL2 write gate, provider-retained scratch
handle or exclusive final CLEAN-unmount barrier. The provider
lease excludes competing filesystem operations but not manual
medium exchange between a verified RBN0 read and DATA write;
full P0-B acceptance must handle that residual race or state
the actual hardware media-generation contract explicitly.

P0-A complete; P0-B OPEN; P1-C separate. No fixed KCORE RAM,
syscall ABI, default media format or optimization changes.

Design `DAIMOS/design/kcore-dtfs-armed-revalidation-v1554.md`.
Testkit `tests/validation/daimos-dtfs-armed-revalidation-v1554.md`.
VERSION 0.5, strict 32K/optimization HOLD.
