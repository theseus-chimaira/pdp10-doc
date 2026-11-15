# KCORE feature state v1594: P1-C exact owner publication and P1-D DSK/DRM roots

Date: 2026-11-15. Production DAIMOS VERSION 0.5, PDP-6 KCORE and
MRES unchanged. No installed code, dynamic RAM, ABI, filesystem
format or optimization/32K placement changed during these tests.

## P1-C v1593: exact second owner RESIDENT publication PASS

Both real 128K KCC PDP-6 cold boots on four-member Type-236
DR0..DR3 D6FS root VOLUME, mapped BACKSTORE, MEMFS/XIP and
LOGSTORE pass. The genuine PI6 timer wakes two distinct original
SLEEP/DISK slots 6 and 7 during MEMFS snapshot. A source-linked,
read-only observer at original successful
`proc_storage_publish_v30` confirms slot **7** (specifically,
not an aggregate anonymous slot) publishes RESIDENT once per
boot, with aligned physical VM `0220000` octal, original
nonzero image record, stable u-area, no transition and cleared
old disk-backing owner pointer. Slot 6 also has one successful
publication from another transaction; that does not invalidate
its separately selected post-DMA checked BACKSTORE release
refusal and original MM/bitmap/BCACHE/TERM rollback.

EACH boot: five actual independent mapped BACKSTORE swap READs,
1,120 matching original Type-236 PI2, four successful RESIDENT
publications, no unexpected device error. Both original privileged
HALTs; second boot restores prior MEMFS/XIP and LOGSTORE before
committing its own snapshot. Raw v2 header DR3 `017777`, 3,648-word
pool and 280 nonzero other physical sectors. Oracle rejects
10 new owner/publication/VM/FSINFO mutants, 16 inherited v1592
owner/PI6/rollback and 13 v1584 physical/scheduler/PI2 mutants:
**39** deliberately unsafe witnesses. P1-C per-slot question CLOSED.

## P1-D v1593 and v1594: two different D6FS root classes PASS

The same v1593 *four-member DRM236 root* physically proves original
FSINFO capacity **6**: D6FS provider 6/root ID1, MEMFS provider
4/`/TEMP` ID2, four spare DSK270 positions, slots 6/7 invalid.

Separately v1594 boots a full 128K PDP-6 from four physical DSK270
DSK0..DSK3 root members and proves the corresponding capacity **6**
with four spare DRM236 positions; the same D6FS/MEMFS entries and
empty/out-of-range behavior pass. Normal XIP EXEC/RUN/WAIT also
passes. A preliminary DCS/GE TCP port collision produced SIMH
attach errors and was rejected, not counted as acceptance. The
corrected private-port re-run passes with no simulator errors.
Independent physical oracle validates four real 45,056-sector
D6FSR2 DSK members, unique identities, matching layout and
rejects three deliberate physical/FSINFO mutations.

Together with accepted v1532/v1533/v1591 Type-551 DTC/TSFS roots
(capacity 7 or 8, real secondary DTFS public ID 5), all three
currently supported root controller classes now have physical
mount-inventory evidence. Native maximum 26 is only a validated
representation, NOT 26 physical devices.

## Still open

P1-C: mixed late DEVICE/VOLUME/DTFS errors, dirty/write-through
behavior and repeated simulator-loaded stability. Real owned-media
controller errors remain fatal under the original fail-stop rule.

P1-D: missing/failed controller service inventory, per-mount
provider-failure/error classification and high-slot retirement
behavior under unavailable physical media. Existing root selection
and FSINFO are implemented; do not add speculative permanent RAM.

P1-E: implementation ON HOLD for KA10-aware process/VM design.
Strict 32K and all optimization on HOLD, VERSION 0.5.

DAIMOS design notes:
`design/kcore-p1c-exact-owner-publication-v1593.md` and
`design/kcore-p1d-dsk-root-inventory-v1594.md`.
Testkit physical acceptance:
`tests/validation/daimos-p1c-owner-publish-physical-v1593.md` and
`tests/validation/daimos-fsinfo-dsk-root-physical-v1594.md`.
