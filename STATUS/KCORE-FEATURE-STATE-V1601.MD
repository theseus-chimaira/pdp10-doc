# KCORE P1-D v1601: dynamic FSINFO and controller inventory feature closed

Date: 2026-11-15. DAIMOS VERSION 0.5; original PDP-6 KCORE and
installed storage filesystem providers. This is a P1-D feature
acceptance, not a new kernel implementation or a P1-C fault pass.

## Physical controller/root profiles accepted

| Original selected root and controller services | Original FSINFO capacity | Physical evidence |
|---|---:|---|
| DTC0..DTC2, three-member TSFS, DTC controller | 7 | v1532/v1533, 256K PDP-6 |
| DTC0/DTC1, two-member TSFS with empty DTC2 | 8 | v1591, 256K PDP-6, independent DTC3..6 DTFS mounts |
| DSK0..DSK3 four-member D6FS, DRM present | 6 | v1594, 128K PDP-6 |
| DR0..DR3 four-member D6FS, DSK present | 6 | v1593, 128K PDP-6 |
| DSK0..DSK3 four-member D6FS, upstream DP/DR absent | 2 | v1597, 128K PDP-6; current-source rerun v1601 |
| DR0..DR3 four-member D6FS, DSK270 absent | **2** | **v1601**, 128K PDP-6; original linked MM/FSINFO and raw root media |

All mounted FSINFO IDs are stable: D6FS/TSFS root ID1 and MEMFS
ID2 on D6FS profiles. Four native high DTFS mounts on the v1591
two-member TSFS root have public IDs2..5. Original FSINFO returns
0 for a truly empty but published slot and -1 for an invalid slot.
Original v1595 physically detached published high DTFS DTC3:
FSINFO returns -1 (unavailable, **not** empty 0), remaining high
mount identities survive, different reel rejected, original
physical reel revalidates, and all mounts retire. Separate v1596
physically removes highest DTC6 while independent MEMFS performs
correct original file write/read, then restores original DTC6.
The original Type-551 simulator return defect was corrected in
`sims` v1598, not bypassed in DAIMOS.

## New v1601: real DRM root and missing DSK270 controller

The disposable boot configuration uses original `deposit SW 3`
(DRM root), disables the DSK270 controller and detaches its four
DSK images before MINIT. Physical Type-167 `DP` and Type-236
DR0..DR3 remain present; four original 8192-sector DRM D6FSR2
members retain their preboot sector-zero descriptors, four distinct
member IDs and common real geometry `000001+017377+000400`
octal. The source-linked original KCORE addresses are:

```
vfs_mount_limit        023703
vfs_mount_root_base    023702
vfs_mount_target_base  023701
vfs_namespace_root     024270
```

The actual post-HALT `vfs_mount_limit` is **2**, both MM-backed
mount bases are resident and distinct. Original FSINFO verifies
D6FS root provider 6 ID1, MEMFS `/TEMP` provider 4 ID2, and
invalid indices 2/3; genuine XIP RUN/WAIT/EXEC, `STATUS:0`,
normal HALT, and no simulator attach/bind errors. An independent
raw media/source-linked verifier rejects **10** false witnesses
(root members, controller presence, root selection, capacity,
mount-array memory, XIP and preboot/postboot integrity).

The first v1601 exploratory physical run predated strengthened
capture and is not accepted. Complete rerun PASS evidence:
`$TMPDIR/daimos-fsinfo-drm-nodsk-v1601.GUnDWg`.
Reproducible testkit entry:
`tests/validation/daimos-fsinfo-drm-no-dsk-physical-v1601.sh`.

The complementary v1597 full guest was rebuilt from the current
source and run again: **PASS**, four distinct original DSK270
members, absent DP/DR service, FSINFO capacity 2, original XIP
lifecycle, **4** mutated witness cases rejected. Retained rerun:
`$TMPDIR/daimos-fsinfo-dsk-nodrm-v1597.Ku8Zhn`.

## Source-linked and boundary regressions

The original native PDP-6 v1492 FSINFO assembly test passes, and
its deliberate capacity-comparison mutation is rejected. The
original `MOUNTS`/`DF` command implementations scan gap/error
cases and 1/4/8/26 capacity representations. Native v22 executes
original root-selection and MM slab-capacity planning in full and
disk-only PDP-6 profiles; the v1601 extended test rejects a missing
selected controller and incomplete READ/WRITE service pairs and
excludes incomplete nonroot controllers. Host capacity boundary/
invalid-mask tests pass. No fixed four-slot fallback was accepted.

**The model's present controller classes permit at most nine
actual potential mount slots with a one-unit root.** Twenty-six
is a representational limit, not a claimed physical installation.

## Closure and independent remaining work

**P1-D CLOSED**: dynamic FSINFO enumeration; DTC/DSK/DRM root
inventories; absent and asymmetric controller services; nonroot
published high-slot failure, protected physical identity, and
return. Only existing accepted physical and source-linked coverage
is claimed. New v1601 work changes only testkit, project status
and documentation, not production KCORE, controller drivers,
MRES, RAM, syscall ABI or disk/tape format.

**P1-C OPEN**: dirty/write-through error isolation, process/cache
ownership intersections, and loaded-simulator timing repeatability.
**P1-E HOLD**: KA10 PURE/MM/APR architectural review before any
implementation. VERSION 0.5; KCORE fixed low end `024364` octal;
strict 32K RAM-footprint/optimization gate HOLD.
