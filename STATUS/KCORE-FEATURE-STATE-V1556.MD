# KCORE v1556: DTVOL2/CLEAN mounted read-fault recovery

Date: 2026-11-14. Production optional DTFS MRES feature,
**P0-B OPEN: writable DTVOL2 is not yet enabled**.

v1555 made an independently formatted DTVOL2/CLEAN DECtape
mountable READ-ONLY, and forbade all ordinary mounted V2
DATA/DIR writes at the central Type-551 write veneer. The
remaining problem was the old mounted READ_SUSPECT recovery
path: it accepted only DTVOL1, so a transient read error
left an otherwise healthy original V2 volume unusable.

v1556 updates actual `dtfs_identity_revalidate_v1522` in
`system/kernel/fs/dtfs_runtime.s`. The routine clears the
shared 0200-word workspace after invalidating its BCACHE
tag and before an uncached physical RBN0 read. This prevents
false-successful short transfers from accidentally accepting
stale complete records. It saves the original mount slot and
whole media word on the process stack across sleeping I/O.

The original media FORMAT_V2 flag selects the native six-word
v1550 parser and requires a completely valid CLEAN state;
otherwise the original four-word V1/zero validator is used.
The physical on-media 72-bit ID must match both original saved
MM-mounted halves. The mount media word must not have changed
across I/O, and V2 additionally requires the still-published
DTFS provider/root and exact public mount ID, plus the original
read-only VFS policy. Only then does READ_SUSPECT clear;
WRITE_UNCERTAIN, owner bits, ARMED and FORMAT_V2 are untouched.
DIRTY/malformed/replaced V2 never recovers as healthy media.

Native testkit `daimos-dtfs-v2-revalidate-v1556.sh` executes
the actual PDP-6 production recovery code and both real native
validators with RBN0 bytes from two independent host-created
V2 volumes and a V1 volume. **29 native cases PASS, 14 unsafe
source mutations rejected.** They include full recovery and
owner preservation, DIRTY/replacement refusal, torn/unknown
seals, short or no-byte false-successful controller completion,
original ID changes, media flags changing while I/O sleeps,
stale provider/mount, RO loss, and V1 backward compatibility.
The existing v1522 mounted recovery suite plus its eight
mutations remains PASS. The real full-profile KCC DTFS MRES
assemble/fold/link is PASS.

This is not a full guest Type-551 DMA, media exchange or
cold-boot acceptance. No writable DTVOL2 DATA/DIR operation,
exclusive final CLEAN seal, or transaction MM scratch owner
has been installed. A new removable-media generation contract
is still required to address the read-then-write exchange race.

Next P0-B tasks: link and enforce the existing verified
v1551-v1554 physical WRITE_INTENT/owned-MM gateway for every
DTFS DATA/DIR and resize/chain write, keep unfreed MM owners,
block all new owners across sleeping CLEAN/unmount, then prove
full-guest partial-write, reinsertion and cold-boot rejection.

No fixed KCORE data allocation, syscall ABI, default on-media
format or optimization changed. P0-A complete; **P0-B OPEN**;
P1-C separate. VERSION 0.5, strict 32K/optimization HOLD.

Source: `DAIMOS/design/kcore-dtfs-v2-recovery-v1556.md`.
Tests: `daimos-testkit/tests/validation/
daimos-dtfs-v2-revalidate-v1556.md`.
