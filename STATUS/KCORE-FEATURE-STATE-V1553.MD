# KCORE v1553: authenticated DTFS first-write gate (native)

Date: 2026-11-14. KCORE feature completion, P0-B **OPEN**.

The new standalone `dtfs_intent_first_write_v1553.s` authenticates
the actual published VFS DTFS root and public mount ID, native
physical Type-551 unit, nonzero saved 72-bit DTVOL2 identity and
MM-held scratch owner before beginning a mutating transaction.
It uses the existing provider-serialized lease, and calls the
v1552 MM-pinned/v1551 raw RBN0 write/readback to verify on-media
WRITE_INTENT. Only after a successful transaction, complete MM
release and unchanged media word does it set an INTENT_ARMED bit
in the spare right half of the per-mount media word. It preserves
UID/GID ownership and refuses old-format, wrong-provider/mount,
READ_SUSPECT, WRITE_UNCERTAIN and pending MM cleanup cases.

Testkit genuine PDP-6 source-linked acceptance: **18 native
scenarios PASS and 13 unsafe source mutants rejected**. The
cases include wrong published mount ID, absent 36-bit identity
halves, failure of the underlying physical arm, retained scratch
pointer, altered media while I/O sleeps, foreign personality,
invalid mount limit, and preserved 18-bit UID/GID owner. The
shared header and assembly flag values are checked for agreement.

The existing `fs_provider_reg_call` serializes DTFS/TSFS/D6FS
across sleeping requests, but the ordinary `dtfs_dtc_write`
veneer has only physical UNIT/BLOCK/BUFFER, not an authenticated
mount slot. Before wiring the gate to every real mutation, the
provider needs a durable mount association and a per-request
physical check of DTVOL2/DIRTY and original identity after the
first arm; a cached ARMED bit cannot prove that a removable tape
was not exchanged. VFS final owner rechecks also do not exclude
new owners throughout sleeping CLEAN/unmount I/O. This must be
fixed before committing an on-media CLEAN seal.

The gate is therefore **NOT linked into production mounted DTFS**
and new-format media still cannot be safely mounted writable.
No fixed RAM, syscall ABI, on-media format default or optimizations
changed. Physical Type-551 partial-write/reinsertion/cold-boot
acceptance is still outstanding. P0-A complete; P1-C open.

Design `DAIMOS/design/kcore-dtfs-first-write-v1553.md`.
Testkit `tests/validation/daimos-dtfs-first-write-v1553.md`.
VERSION 0.5, strict 32K/optimization HOLD.
