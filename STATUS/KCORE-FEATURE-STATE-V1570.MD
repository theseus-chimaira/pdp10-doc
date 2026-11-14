# KCORE v1570: checked poisoned DTVOL2 physical replacement

Date: 2026-11-14. **Full native and real Type-551 guest PASS.**
P0-B remains OPEN; production DTVOL2 remains READ-ONLY.

## Exact production change

The original v1531 privileged DTFS VERIFY_REPLACE MRES op026
physically checked a different, fully stamped DTVOL1 replacement
and independent valid native directory. v1570 adds a format-aware
case for an already-poisoned DTVOL2 mount: only an independently
verified, different-72-bit-ID, fully sealed **DTVOL2/CLEAN** reel
can replace it. The complete production `dtfs_intent_validate_v1550`
checks identity halves/complement, exact CLEAN state, salted seal
and zero padding after the uncached raw Type-551 RBN0 read.
DIRTY, corrupt, same ID, V1/V2 format downgrade/upgrade and invalid
directory are refused. Previously accepted V1-to-V1 behavior is
unchanged. v1531's privileged explicit FD drop, before/after
all-process FILE/cwd/PIPE/swap owner scans, MM-owner empty checks,
cache invalidation and old-slot retirement also remain unchanged.
This discards a poisoned old mount; it never repairs its file data.

## Tests and physical evidence

Thirty PDP-6 source-linked V1/V2 replacement cases/eight unsafe
mutations PASS. Prior v1558 physical-media source preflight 20
cases/six mutants PASS. The complete v1531 real KCC PDP-6
DTVOL1 administrative physical guest is rerun and PASSes.

An isolated source-matched 256K PDP-6 guest first proves the normal
v1555 V2 RW veto, then temporarily allows exact DTC3/mount2 RW for
testing only. The genuine v1551-v1554 physical DIRTY intent/MM gate
precedes its native directory and DATA operations. A real Type-551
`PARTIALWRITE=0145:1` interrupts a DATA write after one changed
36-bit word. The guest refuses ordinary FD close/unmount and
privileged discard with outstanding FD/cwd, requires explicit FD
drop and cwd release, then still refuses the SAME damaged reel.

The host next attaches an independently formatted, different-ID
DTVOL2/CLEAN reel. The **production** private verifier reads and
authenticates its native directory and CLEAN RBN0, and the existing
VFS post-I/O owner scan permits old-slot retirement. The new reel
mounts RO, rejects CREATE and unmounts normally. Each tape is kept
on its own SIMH backing file and separately SHA-256 verified:

Old physically damaged DIRTY:
`e06664a3bac2dc0a6c730648871eeb0692b22cdde19aec82b65c336b1c59aa94`

New independent CLEAN:
`2a32fdea8e863f3153fd5ce2d83ae8c3887639d6f81755339c90b4891d1ffbe4`

Two more **fully unchanged production KCORE/MRES** cold boots
reject the exact old DIRTY tape RO/RW and mount the new CLEAN tape
RO, respectively, without changing either complete tape SHA-256.
The historical v1531 real V1 physical regression also PASSes.

## Budget and unfinished work

No additional permanent fixed KCORE TEXT/DATA/BSS, mount-slab
words, ABI, or boot objects. Only a small increase in optional
DTFS MRES TEXT. `__kcore_low_end` remains `024364` octal.
P0-A CLOSED, P1-C/D/E unchanged, VERSION 0.5, strict 32K HOLD.

**P0-B remains OPEN** because the physical Type-551 has no
reel-generation counter or mechanical remove/eject interlock;
the reel can change after a verified RBN0 read and before the
following WRITE. Same/replacement reel *after known poison* is
now tested; arbitrary external exchange *during writable activity*
is not safe. No production V2 RW cutover is authorized.

See `DAIMOS/design/kcore-dtfs-v2-replacement-v1570.md` and
`daimos-testkit/tests/validation/daimos-dtfs-v2-admin-replace-v1570.md`.
