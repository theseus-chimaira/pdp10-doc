# v1320: PDP-6 swap-in through production DEVICE backing gateway

Date: 2026-11-07. Scope: native PDP-6 assembler integration test,
not a booted 256K full-system acceptance. No production kernel code
or resident memory changes.

The v1319 testkit driver extends the source-authentic v1317
`proc_swap_in`/MM/storage/bitmap transaction test by extracting
the actual `fs_mres.s` backing DEVICE gateway, including the
`backstore_read`, `backing_region_io`, `device_range_translate`,
and sector-loop blocks. Only the final `dsk270_read_sector`
hardware operation is supplied by the test fixture.

The backing transfer requests eight 0200-word blocks beginning
at logical block 1. With a DEVICE region starting at physical
sector 020, production code must perform reads of sectors 021
through 030, check the region bounds and advance the physical
destination pointer by 0200 words for each sector. The stub
validates all eight translated addresses and deposits two distinct
36-bit endpoint markers. The rest of `proc_swap_in` is production
code: MM allocation after a real capacity failure, pin/read/unpin,
storage-mode publication, backing bitmap release and one-shot
scheduler admission. The successful test checks final process
VM ownership, marker integrity and zero outstanding backing.

First-sector and third-sector simulated hardware I/O errors
exercise the production transaction's rollback after no transfer
and after partial transfer, respectively. Both preserve the DISK
record and backing bitmap, unpin/free the new MM extent and
produce exactly one fatal TERM event. A runtime mutation removing
the production gateway's destination-buffer advancement is rejected
because the fully transferred image fails its endpoint assertion.
The original v1317 transaction regression is also run before the
new gateway tests, and the v1319 target was run twice successfully.

This does **not** run a real DSK270 controller, the booted guest's
PI6 scheduling half, independent user-process resource release,
or genuine exchange candidate discovery. Those remain requirements
for the full-system physical no-partner liveness acceptance.
No speculative v1208/v1211 fallback or permanent RAM growth is
introduced by these tests.

Evidence archive on the Termux host:
`$TMPDIR/daimos-swap-device-gateway-v1320-evidence-r1.tar.gz`.
SHA256: `c63d3f14a09d0873bb4958bbf40de7dd61bff84c3a4cf7901d1c54e06348bbab`.
The archive includes the extracted production assembly, positive
and negative PDP-6 reports and maps, the failing mutation, input
hashes and a verified manifest.
