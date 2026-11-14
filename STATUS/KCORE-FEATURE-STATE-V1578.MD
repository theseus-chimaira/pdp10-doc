# KCORE v1578: cold-boot recovery of an uncommitted LOGSTORE orphan

Date: 2026-11-14. Accepted P1-C fault-recovery increment, but the
combined P1-C task remains OPEN. P0-A and trusted-operator P0-B
remain CLOSED. VERSION 0.5, strict 32K/optimization HOLD.

The previous v1577 test physically created an uncommitted LSREC1
record and retried it successfully before reboot. v1578 explicitly
**does not retry**. On the first real 128K KCC/PDP-6 cold boot, the
test's disposable LOGSTORE MRES writes one whole physical Type-236
LOGSTORE sector containing the expected 36-bit magic/sequence and
payload with final commit trailer zero. It then reports a synthetic
negative status to the unchanged production append path, which
keeps its cursor unchanged. An ordinary HALT persists the separate
MEMFS file/XIP snapshot using the same 8 MiB AUXSTORE drum.

Before boot two, independent host analysis locates LOGSTORE from
the AUXSTORE partition descriptor and decodes each original 128-word
physical sector. The invalid record exists at octal block `05374`,
sequence `3`, exactly once, with zero trailer and expected payload.
This is a physical written record, not a synthetic guest log marker.

The second full cold boot has entirely new CPU RAM and uses the
**same drum**. Production LOGSTORE KINIT recovery ignores the
invalid first-boot LSREC1. MEMFS snapshot XIP/data restores, then a
new unique LOGSTORE record is appended and verified. Independent
physical inspection shows that KINIT reused the old orphan's
physical slot and sequence, rather than counting the invalid event:
valid records in the fixed INIT/LOGIN test profile have consecutive
sequences `3,4,5`, and the new event occupies sequence `5`. The
AUXSTORE descriptor remains byte-identical on both boots. Both
physical guests exit their user probes with status zero and HALT.

Four executed negative physical-image variants are refused: falsely
commit the old event, skip the reused sequence while preserving a
valid trailer, skip the final new-event sequence while preserving a
valid trailer, and invalidate the new event's trailer. The existing
native source-linked LOGSTORE recovery/torn record/tape-drain suite
also passes unchanged.

No production kernel, optional MRES, MM allocator, BCACHE, service
ABI, format or fixed RAM modification. `__kcore_low_end` remains
`024364` octal. The test-controlled completion status is returned
**after** a genuine physical DMA of a deliberately invalid complete
sector: this is not proof of a natural Type-236 hardware failure,
nor of a sub-sector device tear. P1-C still needs concurrent active
process swap restoration during MEMFS commit, and real in-flight
DSK270/DRM236/DTC DEVICE-versus-VOLUME and PI-owner fault isolation.

See DAIMOS `design/kcore-p1c-logstore-orphan-v1578.md` and testkit
`tests/validation/daimos-p1c-orphan-reboot-v1578.md`.
