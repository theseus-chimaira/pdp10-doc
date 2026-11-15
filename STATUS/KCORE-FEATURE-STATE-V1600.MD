# KCORE P1-C v1600: direct Type-236 DEVICE BACKSTORE with separate DSK root

Date: 2026-11-15. Full real 128K KCC PDP-6 guest, original
production DAIMOS VERSION 0.5. The **selected D6FS root** is a
four-member DSK270 disk volume with distinct physical D6FSR2
member images. DRM0 is a separate actual Type-236 physical
AUXSTORE provider with original direct-DEVICE BACKSTORE region
`05770+010000` octal and independent LOGSTORE `05370+0400`.
The original linked `device_region_map=400000005770`,
`volume_data_blocks=0` and `backing_region_count=010000`
octal prove this is **not** the previous v1599 DRM-root
interleaved VOLUME mapping. Actual RBN0 AUXST1 descriptor and
preboot inspect text agree; original four DSK member identities
and layout geometry are independently verified.

At MEMFS snapshot shutdown, the accepted disposable test-only
timer observer selects two distinct existing timed SLEEP/DISK
children with positive original backing ownership, slots 6 and 7.
The original PI6 timer service wakes both once, with no fabricated
RUN, swap-in or controller completion. Original slot-0 swap
service starts a real direct Type-236 BACKSTORE READ for **slot 6**
at DR0 physical sector **`006421`**, original relative logical
sector **`000431`**. Its real DR_DONE PI2 is test-selected through
the unchanged driver error branch to the original required
reason-2 critical BACKSTORE HALT **`004011`** octal.

At fatal HALT, the original driver accounts one selected error;
one physical swap READ and matching PI2 occurred, with zero
successful RESIDENT publication. Original process swap blocks
`01650` and all BACKSTORE blocks `01734` octal remain allocated,
MEMFS snapshot header is not falsely committed and 268 other
physical BACKSTORE sectors have real nonzero data. Original
root was usable before fault; its physical member descriptors
remain correctly formed after abrupt halt, but no postfault
ROOT service or clean shutdown is falsely claimed.

Independent exact KCORE-linked post-HALT and raw physical
oracle PASS. **27** deliberately false witnesses rejected:
8 independent two-owner PI6 witnesses, 15 original real
Type-236 PI2/driver/ownership/snapshot cases, and four new
direct-DEVICE/AUXSTORE/independent DSK-root tests. Both old
v1599 mapped VOLUME 23-mutant and v1586 15-mutant tests PASS
again with the revised explicitly mode-selecting oracle.

Historical v1599 documentation correction: its exact original
`daimos_v1584_last_slot` equals **6**, not 7. Both owners
genuinely woke; the v1599 strict verifier already required
the correct two-owner set and remains accepted. The inaccurate
v1599 prose is corrected in all three projects. Neither test
proves two concurrent DMA READs or recovered operation after an
owned-media failure: mandatory critical fail-stop forbids both.

P1-C still OPEN for dirty/write-through error isolation and
loaded repeatability. P1-D remaining final matched controller/
root FSINFO matrix, P1-E future KA10-aware VM architecture.
No permanent KCORE/MM/MRES/driver change, RAM/ABI/media growth;
strict 32K optimization HOLD. Fixed KCORE low end `024364`
octal; DAIMOS VERSION 0.5.

Source tests and physical acceptance:
`DAIMOS/design/kcore-p1c-direct-device-dual-owner-v1600.md` and
`daimos-testkit/tests/validation/daimos-p1c-device-two-owner-fatal-physical-v1600.md`.
