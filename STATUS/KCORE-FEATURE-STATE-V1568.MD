# KCORE v1568: real physical V2 DATA partial-write cold boot

Date: 2026-11-14. **PASS, P0-B still OPEN.**

An unchanged production 256K KCC PDP-6/TSFS-root image was used
as the source-matched base for an isolated test-only KCORE/MRES.
The test first proved the v1555 DTVOL2 RW mount veto remained
operative. Only exact DTC3/public mount2 was then permitted to
become RW after the real DTFS provider published the physical unit,
both saved 36-bit ID halves and the mount owner. The real v1551-v1554
DIRTY first-write and MM-pinned physical verification remained intact.
No test-only RW policy change entered production sources or binaries.

Real guest PID1 created a native file, physically wrote its directory
and two seeded data words, and detached DTC3 to persist the complete
media. Independent full-tape verification proved the CLEAN->DIRTY
RBN0 state and salted seal, native RBN0144 directory and RBN0145
DATA changes; all other blocks were unchanged. A diagnostic real
Type-551 `PARTIALWRITE=0145:1` then interrupted a second physical
DATA write after one changed 36-bit word. The other requested word
remained old, and the entire valid DIRTY RBN0 was untouched. The
guest refused stale FSINFO, cwd and unmount while TSFS root survived.

The exact persisted damaged tape was cold-booted by an **entirely
unchanged production KCORE/MRES**. Both RO and RW mounts refused,
the DTFS mount remained unpublished, and whole-tape SHA-256 remained
unchanged. No old-data recovery is claimed.

This closes the real interrupted V2 DATA physical acceptance case,
not P0-B. Outstanding: physical interrupted V2 directory update,
same/replacement reel reinsertion after real user-data RW and a
defensible operator/hardware Type-551 reel-exchange interlock policy.
The Type-551 provides no atomic cartridge-generation identifier.
Production DTVOL2 remains **READ-ONLY**.

No production KCORE TEXT/DATA/BSS, mount slab, ABI or MRES growth;
`__kcore_low_end` stays `024364` octal. P0-A closed, P1-C/D/E
unchanged, VERSION 0.5, strict 32K optimization HOLD.

See `DAIMOS/design/kcore-dtfs-partial-data-v1568.md` and
`daimos-testkit/tests/validation/daimos-dtfs-v2-partial-data-v1568.md`.
