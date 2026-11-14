# KCORE v1569: actual partial Type-551 DTVOL2 directory + cold boot

Date: 2026-11-14. **Physical DIR and second production cold boot PASS.**
P0-B remains OPEN and production DTVOL2 stays READ-ONLY.

## Physical guest acceptance

The full KCC 256K PDP-6 booted real TSFS and independently stamped
DTVOL2/CLEAN DTC3. Production v1555 RW refusal was first verified.
An exact DTC3/mount2 exception exists only in a disposable,
source-matched test image, after the provider publishes the exact
root, physical unit and saved 72-bit ID. The genuine v1551-v1554
first-write MM lease and checked physical WRITE_INTENT gate execute
before the first DATA/DIR physical write. No production code changed.

An initial native CREATE wrote on-media DIRTY and directory RBN0144.
At a linked test-only VFS breakpoint just before the second CREATE,
SIMH DETACH persisted that first write. Independent full-tape
comparison confirmed only RBN0 and RBN0144 changed. The diagnostic
Type-551 was set to `PARTIALWRITE=0144:1`. The next unmodified DTFS
CREATE was interrupted after exactly one changed 36-bit RBN0144
word. Another verified instruction breakpoint and DETACH captured
the actual failed-write tape: exactly six stored bytes (one word)
changed, while the whole valid DIRTY RBN0, 72-bit ID and all other
physical blocks stayed unchanged.

VFS returned failure, retaining the poisoned mount. Guest FSINFO,
CHDIR and UNMOUNT refused, while TSFS root remained healthy.
The exact physical torn tape was then attached to a second, entirely
unmodified production KCORE/MRES cold boot. Both RO and RW mounts
were refused; no public DTFS mount appeared. Its complete SHA-256
remained unchanged:

`a85673f46bc72ce3707186c159c794253e091878a5ad5fe398b4e2dae864f8ae`

Accepted physical failure classes now include v1567 partial CLEAN,
v1568 partial DATA, and v1569 partial DIR, each followed by a fresh
production mount-refusal proof. None implies recovery of damaged
old-data contents.

## Remaining P0-B and storage budget

The Type-551 has no persistent physical reel-generation identifier,
prevent-eject or atomic media-exchange interlock. External reel
exchange can happen between a checked RBN0 ID and a later hardware
WRITE. Same/replacement tape reinsertion, exchange under live RW
and a defensible mechanical/operator policy remain the release
blockers. The v1555 production V2 read-only veto stays mandatory.

No production KCORE/MRES, physical-driver, ABI, fixed RAM or
boot-image growth. `__kcore_low_end` remains `024364` octal.
P0-A closed, P1-C/D/E unchanged, VERSION 0.5; strict 32K and
optimization HOLD.

See `DAIMOS/design/kcore-dtfs-partial-dir-v1569.md` and
`daimos-testkit/tests/validation/daimos-dtfs-v2-partial-dir-v1569.md`.
