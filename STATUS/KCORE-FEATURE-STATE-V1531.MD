# KCORE feature v1531: checked physical DTFS replacement

Date: 2026-11-13. Feature completion only.
VERSION 0.5; strict 32K placement/optimization HOLD.

## Accepted root-only administrative replacement

The real rebuilt 256K KCC PDP-6 runs fixed
KCORE and installed DTFS/TSFS MRES from a
physical TSFS DTC0-DTC2 root. A root-only
UUO-077 extension 054 introduces two
destructive operations on the existing
WRITE_UNCERTAIN noncritical DTFS mount:

* `DROP_FD` explicitly abandons one current
  process's selected FILE descriptor without
  performing the ordinary failed SYNC.
  It cannot abandon a healthy/non-native
  mount or a PIPE, cannot affect remote
  descriptors and never declares damaged
  media repaired.
* `REPLACE_SLOT` requires an unowned,
  noncritical poisoned DTFS mount and a
  physically present **different** valid
  native DTVOL1 volume. Fresh Type-551
  reads verify the native directory and
  both nonzero halves of its full 72-bit
  stamped RBN0 identity against the
  old persisted mount identity.

The original same-ID damaged volume
**must not** retire the slot, even when
the caller has dropped every FILE and
cwd owner. A successful different-volume
admission retires the old VFS slot
without SYNCing uncertain data, and
the new volume must be mounted through
an independent normal MOUNT validation.
It is a discard-and-replacement protocol,
not a data repair algorithm or silent
automatic recovery.

## Physical acceptance

The complete real guest:

1. Mounts stamped RW DTC3, opens/writes
   a FILE and retains cwd; clean admin
   operation attempts are rejected.
2. Physically DETACHes DTC3; a real
   metadata WRITE fails and quarantines
   the mount.
3. Reattaches the original same-ID tape;
   ordinary CLOSE still refuses, root
   DROP_FD succeeds and repeated drop
   is rejected. Cwd blocks replacement.
4. Releases cwd; the old same-ID tape
   is still rejected with no owners.
5. Physically ATTACHes a separately
   stamped, distinct valid DTC3. The
   old mount is safely retired only
   after fresh device validation.
6. Confirms FSINFO slot empty, new
   MOUNT succeeds, old filename absent,
   clean UNMOUNT succeeds, and TSFS
   root remains available.

Guest `DTFSADMIN V1531 PASS`,
host exit zero. This was rerun after
the final native-personality guard
on the rebuilt KCORE; PASS.

## Native source-linked security/ownership tests

The v1531 PDP-6 oracle runs the actual
VFS mount-retirement and FILE drop code,
with actual stable-uarea process descriptors.
It verifies independent swapped-process
FD15 and cwd owners, PIPE/swap exclusion,
and a new remote FILE appearing during
sleeping provider verification. The
post-provider final owner check prevents
recycling a live mount. Missing
WRITE_UNCERTAIN, foreign personality,
wrong slot/root and clean FILEs are
denied. The oracle PASSes and rejects
four executable unsafe source mutations:
skip initial owner scan, skip final
owner check, skip physical verifier,
skip WRITE_UNCERTAIN guard.

An initially faulty oracle set its failure
witness after the first assertion, creating
a false positive for an early failing
mutation. The testkit fixed this and
all four mutations now fail in real PDP-6
execution. No production workaround.

## Full regression and footprint

The actual physical v1526 remote FILE/cwd
mount ownership, v1527 stamped media return
and independent DTC4, v1528 failed-write
sticky containment and v1529 interrupted
RBN0145 partial write **all PASS**
against the rebuilt v1531 production kernel.
Older native v1514/v1523 tests also PASS
with their original assertions and corrected
source extraction boundaries.

Full KCC DXR headers, v1527 -> v1531:

| Region | v1527 octal | v1531 octal | Delta |
| --- | --- | --- | --- |
| Fixed KCORE image | 023116 | 023261 | +99 words |
| Fixed KCORE BSS | 000415 | 000415 | 0 |
| DTFS/TSFS optional MRES image | 003747 | 004057 | +72 words |
| DTFS/TSFS optional MRES BSS | 000002 | 000002 | 0 |

No new persistent data, process tables,
on-media format fields or recoverability
metadata. The feature costs code words,
not additional resident BSS.

## Remaining KCORE features

Destructive noncritical DTFS replacement
is physically accepted. It does not
recover partial user data or automatically
close other processes; a stronger direct
multi-process physical recovery scenario
remains desirable, though the native scan
and v1526 cross-process physical owner
behavior are accepted separately.

P0-A authoritative permanent critical
TSFS root/BACKSTORE loss and qualified
HALT still OPEN. P1-C mixed fault
paths, P1-D dynamic FSINFO capacity,
P1-E swapless PURE/MM/APR and final
KCORE feature freeze remain OPEN.

Source: DAIMOS/design/kcore-dtfs-admin-replace-v1531.md.
Test evidence: daimos-testkit/tests/validation/
daimos-dtfs-admin-replace-v1531.md.
