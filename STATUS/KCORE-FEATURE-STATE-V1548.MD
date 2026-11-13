# KCORE v1548: P0-A critical-storage fail-stop accepted

Date: 2026-11-13. KCORE feature completion, not optimization.

## Final selected-root/BACKSTORE policy

The selected ROOT filesystem and enabled or still-owned BACKSTORE
are essential services. An otherwise valid required read/write
that still fails after ordinary bounded driver recovery executes
a no-return PI-disabled HALT. The first failed logical service
determines the reason: fixed v1515 ROOT HALT `003743`, or
BACKSTORE HALT `003747` in the tested KCC PDP-6 link. The policy
does **not** wait for proof of permanent physical-media absence.
Independent secondary mounted filesystems return I/O errors and
keep their owners; invalid requests, EOF, inactive/unowned probes
and successful bounded physical recovery remain nonfatal.

## Physical and native acceptance

Previously accepted v1538/v1539 root TSFS full/single-tape
physical detachment, v1540/v1543 Type-270 disk/Type-236 drum
D6FS root outages, v1534/v1545 independent noncritical
mounted DTFS/D6FS failures, v1541/v1542 BACKSTORE native and
selected real PI2 completion, and v1546 combined native logical
ordering all remain valid. v1547 accepted **BACKSTORE-first**
error on a shared DRM236 root/BLOCKSET VOLUME with actual
octal `0320` owned swap blocks.

The final v1548 **ROOT-first** 256K KCC PDP-6 guest uses the same
mapped DR0..DR3 root/BACKSTORE media. It exercises ordinary
`/SYSTEM/INIT` FILE reads from a running process. A genuine
selected-root D6FS BCACHE miss is armed for the selected Type-236
PI2 descriptor error only while BACKSTORE is enabled and owns
blocks. The original driver error branch returns a failed I/O to
the production root classifier, which HALTs at octal **`003743`**
(not the swap HALT). Exactly one root arm and one selected PI2
error, five clean earlier physical samples, and octal **`0320`**
BACKSTORE blocks both at arm and at HALT prove first-failed-service
ordering and retained ownership. Published D6FS root is
`060101000000`; device map `400000000000`; mapped data region
`000000075774`. Thirteen corrupted evidence/source variants are
rejected.

The PI2 error branch was deliberately selected in a disposable
test guest; this is real physical-simulator request/interrupt/error
handling, **not** spontaneous hardware loss. Actual physical
media detach acceptance is supplied by the separate root and
noncritical filesystem tests. There was no additional production
KCORE code, RAM, ABI or media-format change for v1548.

## Status

**P0-A COMPLETE** under the agreed policy and its defined native/
full-guest acceptance matrix. Previous dated P0-A `OPEN` entries
in historical design or status documents refer to earlier versions
and are superseded by this acceptance.

Next: P0-B durable rejection of discarded partially written
DTFS media on reinsertion/cold boot. P1-C combined process swap
plus persistent MEMFS/LOGSTORE fault scenarios remain OPEN.
VERSION 0.5; strict 32K/optimization HOLD.

Source details: `DAIMOS/design/kcore-p0a-failstop-acceptance-v1548.md`.
Physical evidence SHA-256 and reproducer:
`daimos-testkit/tests/validation/daimos-shared-drm-root-first-v1548.md`.
