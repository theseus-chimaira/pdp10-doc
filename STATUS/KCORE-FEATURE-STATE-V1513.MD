# DAIMOS KCORE v1513: terminal critical media, recoverable non-root media

Date: 2026-11-12. Supersedes earlier speculative *root or
BACKSTORE media reconstruction* tasks. This is a settled
feature **policy** plus a source-linked native non-root recovery
gate; it does not claim physical detach/reattach acceptance.

## Critical media loss is fatal, not an owner-recovery project

Confirmed loss of the selected boot root or BACKSTORE medium
must stop KCORE. There is no requirement to reconstruct lost
critical images, forcibly terminate suspended swapped owners,
complete foreign private FILE cleanup, invent a ZOMB or free
backing whose authoritative bitmap cannot be verified.
An inexpensive fatal reason (distinct root versus BACKSTORE)
is desirable but secondary to correct fail-stop operation.
No new quarantine state, permanent retry table or process
state is required to implement this decision.

The *confirmed* qualification matters. Both PDP-6 Type-270 and
Type-167/236 drivers report temporary PI errors or expired
300-tick watchdogs as ordinary negative I/O results. They do
not yet expose a distinct reliable offline/permanent-media-loss
classification. The physical v1511 and v1512 tests deliberately
exercise and survive two such errors on real BACKSTORE requests,
then restore a separate healthy swapped process. Panic-on-first
`-1` would regress those established guarantees and is forbidden.

**Still OPEN:** reliable controller/media-loss detection distinct
from ordinary negative I/O; a minimal fatal HALT/diagnostic
at the critical owner boundary; full physical critical-loss
verification. Until classified, the existing bounded rollback
and exact DISK ownership remain authoritative, but this is not
to be described as permanent-media recovery.

## Secondary filesystem loss must be recoverable

An unavailable *non-root* mounted filesystem must fail I/O
without tearing down the kernel, discarding the mount or
silently publishing a clean cache entry. Root and other mount
identities must remain intact. When the **same** medium returns,
normal reads can resume; replacing media requires checked
identity/geometry/superblock revalidation and BCACHE
invalidation before stale blocks become visible.

Testkit `daimos-nonroot-media-return-v1513.{s,sh}` uses the
real PDP-6 D6FS `d6fs_reader_get_block` and `fs_backing_read`
source. Native PASS: two negative backing reads from secondary
mount ID 2 return NULL without publishing a new cache entry;
independent root reader mount ID 1 succeeds. When the same
secondary callback becomes available, clean reads succeed;
a second failure and return also behave correctly. Three
executed source mutations (reversed error condition,
non-NULL failure, fabricated callback success) are rejected.
The old v1 D6FS multi-mount source test now links again after
a test-only missing callback stub was supplied, and PASSes.
D6FS v674 remount source and VFS v9 mount mutation PASS.

This is a **source-linked, cache-miss native test**, not
verified full-system device hot-unplug, volume rescan, media
replacement, dirty-block policy or open-FILE reattachment.
These remain OPEN. The existing VFS remount helper is not a
replacement-media validation interface; full recovery must
handle BCACHE and ownership correctly rather than assuming a
successful retry certifies a different medium.

Testkit revision `8ab7648`; DAIMOS source policy revision
`282979ad` (comments only; no runtime changes). Detailed
policy and test report:
`DAIMOS/design/kcore-media-loss-policy-v1513.md` and
`daimos-testkit/tests/validation/daimos-media-loss-policy-native-v1513.md`.

## Remaining prioritized KCORE feature gates

1. P0: reliable permanent root/BACKSTORE media-loss
   classification; minimal fatal stop, small diagnostic if cheap;
   actual physical proof. **No restoration or reconstruction.**
2. P1: non-root removal/reappearance, identity validation and
   BCACHE invalidation on replacement, dirty-data/pin/open FILE
   safety, full physical acceptance.
3. P1: DEVICE/VOLUME, BCACHE/DTFS, LOGSTORE/MEMFS mixed failure
   handling without promoting secondary loss to global fatal.
4. P1: F1 swapless PURE and checked MM/APR restoration.
5. P1: F3 dynamic FSINFO mount enumeration 1/4/8/26.
6. P2: compression and final RAM optimization only after
   functional gates and measured memory benefit.

Previously accepted KCORE v1506-v1512 physical ownership,
retry, PI2 and PI6 error-path tests remain accepted. VERSION
0.5 and 32K HOLD unchanged; MRES unload/reload post-1.0.
