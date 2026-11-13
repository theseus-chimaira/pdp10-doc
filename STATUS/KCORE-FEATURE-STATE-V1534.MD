# KCORE v1534: real mixed TSFS/DTFS/MEMFS physical write fault

Date: 2026-11-13. P1-C feature-validation increment.
No optimization, new userland feature, additional fixed
KCORE data or filesystem-format change.

## Four simultaneously live providers/volumes

The full 256K KCC PDP-6 physical TSFS-root guest
mounts independent stamped writable DTC3/DTC4 native
DTFS volumes and a writable MEMFS instance. The
host validates DTC3 and DTC4 have distinct fully
stamped nonzero 72-bit identities. The real PID1
creates one DTC3 FILE, verifies the exact two-word
DTC4 FILE and the exact three-word MEMFS FILE, and
checks all active FSINFO records and seven-slot
runtime mount capacity. TSFS root stays mounted
on DTC0-DTC2.

After a final DTC3 STAT reloads its shared directory
cache, SIMH physically DETACHes only DTC3 at a
synchronized PID1 checkpoint. Native CHMOD performs
an actual failed DTC3 metadata WRITE and records
WRITE_UNCERTAIN. DTC3 FSINFO/CLOSE/UNMOUNT refuse
without dropping the original FILE owner.

While DTC3 is physically absent, DTC4 DTFS and
MEMFS contents and FSINFO, TSFS root and KINIT
capacity still pass. Reattaching exactly the old
DTC3 does not remove WRITE_UNCERTAIN: FSINFO/STAT/
CLOSE still refuse. A **new healthy DTC4 metadata
CHMOD WRITE succeeds**. DTC4/MEMFS data and FSINFO
remain good, and both secondary mounts retire
normally; the poisoned DTC3 slot and FD persist.

The complete real guest reports
`MIXEDIO V1534 WRITE_FAILED`, `HEALTHY_DURING`,
`HEALTHY_RETURN`, `RETIRED_HEALTHY`, `PASS`;
the host exits zero.

An earlier attempt read DTC4 last, so detached
DTC3 first failed a directory READ. It legitimately
revalidated on matching-ID return (READ_SUSPECT).
That test-ordering problem was fixed by prewarming
DTC3 last, not by changing the existing KCORE
recovery policy.

The historical v1522 native fixture also had
to stop extracting v1531's unrelated replacement
validator to keep its original mutation anchors
unique. Its baseline and all eight unsafe PDP-6
source mutations again reject. Production code
and MRES remain unchanged.

**Accepted:** P1-C physical cross-provider DTC
WRITE_UNCERTAIN isolation against healthy DTC,
MEMFS and TSFS root.
**Still open:** mixed DSK270/DRM236/DTC device/
volume mapping, BACKSTORE/MM rollback, persistent
LOGSTORE+MEMFS snapshot with concurrent process
swap and full provider fault matrix.

Sources:
`DAIMOS/design/kcore-mixed-storage-physical-v1534.md`
and `daimos-testkit/tests/validation/
daimos-mixed-storage-fault-v1534.md`.

P0-A authoritative permanent critical-loss HALT,
P0-B durable old-media admission, P1-D additional
inventory profiles and P1-E swapless PURE/MM/APR
remain open. VERSION 0.5, strict 32K/optimization HOLD.
