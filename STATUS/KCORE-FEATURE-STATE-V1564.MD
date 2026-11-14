# KCORE v1564: physical DTVOL2/CLEAN and WRITE_INTENT full guest

Date: 2026-11-14. Actual full KCC 256K PDP-6, Type-551 physical
DTFS transfers, two independent guest cold boots. **PASS.**

## First cold boot: DTVOL2/CLEAN

An independent full host-format `mkdtfs --durable-v1550` produced
a complete native 578-block DTC3 DECtape. The test independently
verified the full 128-word RBN0: exact SIXBIT DTVOL2 stamp,
nonzero two-half 72-bit identity, complement, CLEAN state,
salted seal, and zero padding. RBN0144 retained the correct
directory owner mapping. Its full-tape SHA-256 was recorded.

Actual KCORE rejected a writable DTVOL2 mount, accepted a
read-only mount, then ran two independently scheduled processes.
PID2's cwd and later FILE references to the DTFS mount caused
PID1's UNMOUNT to fail in each case. After the child released both,
the real VFS retired the mount while PID2 was still alive.
Following child completion, another RW attempt was refused and
another RO mount/unmount succeeded. **The full DTC3 SHA-256 was
unchanged after the physical guest HALT.**

## Second cold boot: DTVOL2/WRITE_INTENT

A fresh DTVOL2 physical tape was intentionally placed in the
exact valid WRITE_INTENT state with recalculated salted seal;
the 72-bit ID, directory map and other RBN0 words remained valid.
This models a previous aborted write after DIRTY arm, not a
corrupt or missing identity. Independent verification accepted
the on-media encoding before the second guest started.

Actual booted PDP-6 KCORE refused both RW and RO admission,
left the FSINFO public slot unpublished and rejected an attempted
unmount of that absent slot. **The full DTC3 image SHA-256 remained
unchanged after HALT.**

The historical v1526 physical DTVOL1 cross-process fixture also
PASSed against the same current full KCC image after testkit
v1525/v1526 harness changes.

## Scope and remaining work

This v1564 stage changes no production DAIMOS source, VFS ABI,
fixed RAM allocation, MRES code or on-media default format.
Actual DTVOL2/CLEAN RO physical admission and mount retirement,
and real DTVOL2 WRITE_INTENT rejection now have full-guest proof.

Still unproven: a deliberately simultaneous syscall owner
publication while unmount SYNC sleeps, an actual physical
DIRTY-to-CLEAN marker write/readback under an exclusive VFS owner
barrier, protection against removable-tape exchange, partially
completed RW data writes, reuse/reinsertion and cold boots after
such real RW failures. No DTVOL2 writable gate was enabled.

**P0-A complete; P0-B OPEN; P1-C/D/E unchanged. VERSION 0.5.**
Strict 32K placement/optimization on HOLD.

Source: `DAIMOS/design/kcore-dtfs-physical-v1564.md`.
Tests: `daimos-testkit/tests/validation/daimos-dtfs-v2-physical-v1564.md`.
