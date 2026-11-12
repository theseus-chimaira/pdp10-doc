# DAIMOS KCORE v1514: non-root sync failure and return-to-unmount

Date: 2026-11-12. Incremental native feature acceptance over the
v1513 policy decision and source-linked D6FS read/retry test.
No production KCORE code, ABI, permanent RAM or filesystem
on-media format changed in this step.

## Native source-linked VFS acceptance

Testkit v1514 invokes the actual PDP-6 `vfs_unmount` and
`vfs_mount_retire_slot` instructions from `fs/vfs.s`, with
independent root (public mount ID 1) and secondary D6FS
(public mount ID 2) mounts. The secondary sync callback
simulates missing media by returning -1.

On failed sync, `vfs_unmount` returns -1 and leaves secondary
mount root, target and read-only policy intact. It does not
invoke D6FS provider preparation, FILE mount unlock or slot
retirement. The root namespace/mount remains unchanged.
When the **same** medium returns, the second call completes
sync, prepares D6FS unmount once, unlocks FILE ownership once,
and retires exactly the secondary root/target/RO policy.
The namespace root remains unchanged. Three actual source
mutations were rejected: ignoring failed sync, omitting slot
retirement, and omitting FILE unlock.

Native v1514 baseline and all three negative tests PASS.
Testkit `34fab36`; DAIMOS policy evidence
`design/kcore-nonroot-unmount-return-v1514.md`, revision
`b18becc7`. This extends v1513 `8ab7648`, which tested
non-root D6FS read failure/no cache store and eventual retry.

This is **not** physical hot-unplug acceptance. Provider
callbacks are controlled test fixtures, and the test does not
execute real BCACHE invalidation, revalidate replacement media,
discard dirty blocks, or preserve live open FILEs through a
device change. Those are still open non-root recovery gates.
Confirmed root/BACKSTORE media loss remains **fatal** per
v1513, with reliable permanent-loss classification and a small
HALT diagnostic yet to be implemented and physically tested.

## Remaining active feature gates

1. P0: critical root/BACKSTORE permanent loss classifier
   separate from transient PI/timeout error; terminal HALT,
   minimal distinct diagnostic if cheap. No reconstruction.
2. P1: actual non-root detach/reattach and replacement-media
   revalidation, BCACHE invalidation, dirty/open FILE ownership.
3. P1: mixed DEVICE/VOLUME/DTFS/BCACHE/LOGSTORE/MEMFS faults.
4. P1: F1 swapless PURE/MM/APR and F3 dynamic FSINFO 1/4/8/26.
5. P2: compression/RAM optimization only after feature gates.

VERSION 0.5, 32K HOLD; MRES unloading remains post-1.0.
