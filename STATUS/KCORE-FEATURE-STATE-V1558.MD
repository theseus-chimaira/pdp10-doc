# KCORE v1558: fail-closed short physical DTFS admission reads

Date: 2026-11-14. Production DTFS MRES correctness fix and
PDP-6 source-linked native acceptance. **P0-B remains OPEN**.

The v1557 unique DTVOL2 unit-owner feature was followed by
a physical Type-551 admission source sweep. The shared
0200-word directory/identity workspace was reused by the
real `dtfs_mount_unit` and poisoned-volume administrative
`dtfs_verify_replacement_v1531` without clearing its contents
before an uncached physical RBN0144/RBN0 READ. Invalidating
BCACHE evicted the old tag, NOT the old 128-word contents.
An incomplete transfer whose controller falsely reports
success could therefore accept stale directory metadata.

**Critical distinction:** all-zero physical RBN0 is valid
unstamped legacy DTFS. Simply clearing the identity workspace
to zero before a raw RBN0 read would cause an empty successful
transfer to be misidentified as a complete legacy read.
v1558 now uses one shared assembly buffer-initialization leaf:
it zeroes all 128 words before each directory RBN0144 read,
but fills all 128 with **nonzero all-one sentinels** before
RBN0 mount/replacement identity reads. A full genuine legacy
zero block overwrites the sentinels and is still allowed;
short or zero-byte transfers leave poison and fail admission.

The v1551 independent physical WRITE_INTENT arming helper also
clears its entire pinned MM scratch BEFORE its first raw RBN0
READ, not merely before final physical readback. This closes a
case where a previous fully checked CLEAN header left in reused
MM scratch could authorize a DIRTY write without a new complete
physical transfer. The initial and final clear use the same
source leaf; the helper remains deliberately UNLINKED from
live DTFS writes.

Executed tests:

- Actual native PDP-6 V2 physical arm/readback with reused valid
  CLEAN scratch and false-successful zero/short transfer:
  **17 cases PASS, ten unsafe mutations rejected**.
- Actual production PDP-6 mount admission with real independent
  host `mkdtfs` DTVOL2 and V1 RBN0/RBN0144 bytes:
  **28 cases PASS, 13 unsafe mutations rejected**, including
  a mutation incorrectly prezeroing RBN0 and accepting zero-
  byte media as genuine legacy format.
- Actual native PDP-6 poisoned-volume replacement preflight with
  authentic V1/V2 physical media bytes: **20 cases PASS, six
  unsafe mutations rejected**, including skipped directory
  initialization and unchanged-media replacement refusal.
- Legacy v1519 physical directory and v1520 72-bit ID/MKFS
  suites PASS; existing v1555 V2 central write veto and v1556
  identity recovery suites PASS; complete KCC full-profile
  DTFS MRES assemble/fold/link PASS.

No new fixed KCORE RAM, persistent scratch table, MM allocation,
syscall ABI, default media format or optimization. No new DTFS
DATA/DIR writing is enabled. This is native PDP-6 source-linked
acceptance with controlled Type-551 controller completion, NOT
a full-guest actual physical partial-write or cold-boot test.

P0-B still needs provider-retained MM scratch ownership, mandatory
v1551-v1554 DIRTY write-ahead/verification before every actual
DATA/DIR transfer, exclusive owner-safe CLEAN/unmount across
sleeping Type-551 I/O, reinsertion/replacement and cold boot,
and a proved handling of manual tape exchange between physical
RBN0 verification and the subsequent data write.

P0-A complete; **P0-B OPEN**; P1-C separate. VERSION 0.5;
strict 32K and optimization HOLD.

Source: `DAIMOS/design/kcore-dtfs-raw-read-sentinel-v1558.md`.
Tests: `daimos-testkit/tests/validation/
daimos-dtfs-raw-read-sentinel-v1558.md`.
