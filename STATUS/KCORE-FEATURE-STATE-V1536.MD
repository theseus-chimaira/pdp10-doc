# KCORE v1536: MEMFS PERSIST versus BACKSTORE allocation

Date: 2026-11-13. Feature-completion evidence, not optimization.

## Accepted narrow KCORE P1-C reservation gate

The current unmodified `proc_swap_boot.c` is compiled into a
source-linked host test with explicit test-only MM/backing-header
stubs. Nine independent boot cases assert correct v1/v2 MEMFS
snapshot header recognition and range reservation, invalid
header/refused device read behavior, one BCACHE workspace-tag
invalidation, exact 145-sector bitmap and accounted ownership,
one process record allocation and one slot-state table.
Committed snapshot sectors 107..144 must all be reserved
*before* process swap can claim other sectors; invalid headers
must not reserve guesses. Four deliberately unsafe production
source mutations are executed and rejected.

The actual `backstore_alloc`,
`backstore_verify_allocated_v795`, and `backstore_free`
PDP-6 instructions are separately extracted from the current
production assembly and run under `p10run`. In the same
145-block layout with high-end 38-block snapshot reservation
and another live owner at block 0, the allocator admits
temporary 100-block and 6-block process extents at 1 and 101,
refuses overlarge/extra requests, safely releases the temporary
runs and rejects unauthorized overlapping/duplicate release.
Both snapshot and older process occupancy remain intact.
Three deliberately unsafe instruction mutations are rejected.

The initial native test accepted a mutant that bypassed
complete-run free preflight because the test only asked for
valid frees. It was strengthened to exercise a partial
free across the start of the snapshot and double-free; all
mutants then rejected. This fixes the test oracle, **not**
production KCORE.

## Current-source real cold-reboot evidence

The 128K KCC PDP-6 v1477 MEMFS PERSIST physical guest
was rebuilt from the current KCORE tree and PASSed two
full boots. The first wrote the data and certified XIP
and committed at orderly HALT. The second used fresh
process/RAM state and unchanged physical AUXSTORE media,
restored exact contents, executed the saved XIP and HALTed.
The AUXSTORE descriptor was inspected before and after
each run and remained unchanged. This supplements earlier
v1484/v1485 acceptance; it is not a claim of redoing
every old profile or corrupt-snapshot test.

## Remaining KCORE feature gaps

These three validations are complementary but not a
single live guest with actual swapped-process restoration
*during* persistent MEMFS snapshot shutdown and
concurrent DTC/DSK/DRM or LOGSTORE faults. P1-C
full concurrency, MM/BCACHE/PI owner rollback and
mixed DEVICE/VOLUME identity remain open.

The authoritative current outstanding feature-only
checklist is at the top of `DAIMOS/TODO.MD`
(2026-11-13 v1536). Its open priorities are:

* P0-A: authoritative positive permanent TSFS-root/
  BACKSTORE loss detection and narrowly qualified HALT.
* P0-B: durable old WRITE_UNCERTAIN volume re-admission
  policy across reboots, with explicit administrative checks.
* P1-C: real live mixed physical swap/snapshot/LOGSTORE
  concurrency, storage mapping, BCACHE and PI rollback.
* P1-D: alternative boot-inventoried hardware/mount slabs
  beyond the already accepted seven-slot physical profile.
* P1-E: uncompressed swapless PURE/MM/APR scheduler
  integration and full rollback under pressure.
* Final: current-source matched full feature regression
  and accounting freeze; optimization remains deferred.

No fixed KCORE/MRES/data layout, on-media format or
permanent RAM change in v1536. VERSION 0.5,
strict 32K and optimization HOLD.

See `DAIMOS/design/kcore-snapshot-swap-reservation-v1536.md`
and `daimos-testkit/tests/validation/
daimos-snapshot-swap-reserve-v1536.md`.
