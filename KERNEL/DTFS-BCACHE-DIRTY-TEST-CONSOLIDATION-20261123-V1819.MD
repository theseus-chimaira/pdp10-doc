# DTFS and BCACHE historical test consolidation, v1818-v1819

**Scope:** testkit-only recovery and repair. No kernel source code or
production memory placement changed during this consolidation.

## DTFS physical write ABI (v1818)

The untracked v206 source-linked probe checked whether the DTC raw
write adapter preserved the original physical UNIT, BLOCK and BUFFER
arguments. It exercised an old `bcache_reclaim` write path that is
no longer present. Current `dtfs_dtc_write` first checks the mounted
intent gate, restores all three raw arguments from its process stack,
then invalidates clean cache identities with `bcache_invalidate_all`.
The latter routine is specified to use AC4, preserving AC1..AC3.

In daimos-testkit, `tests/validation/daimos-dtfs-write-preserve-v1818.sh`
extracts the *actual current source* for both routines. The PDP-6
test substitutes a mount-intent gate that overwrites the three
argument accumulators and a raw write sink that records exact
physical UNIT=3, BLOCK=0144 and BUFFER pointer. Production passes;
two native simulator negative controls reject (1) write-time
reclamation and (2) AC2 corruption during invalidation.

Command: `make test-dtfs-write-abi-v1818` in daimos-testkit.
Source test commit: `4cc73b1` on testkit main. The original two
files were preserved under
`~/backups/daimos-dtfs-v206-promotion-v1818-20261123/` with verified
SHA256; their detached worktree was cleaned.

This checks ABI and register preservation, **not** full device I/O,
mounted DTFS persistence, or media-fault handling.

## BCACHE LOW allocation regression (v1819)

DAIMOS production has allocated its reclaimable 0201-octal-word,
MM-owner-6 cache slab from LOW memory since v1636. LOW placement
avoids obstructing the highest aligned process APR. An existing
testkit assembly fixture still expected MM_ALLOC_HIGH and therefore
failed even against correct production code.

`tests/validation/daimos-bcache-allocation-v1.s` was corrected to
require preference zero (LOW), retaining all existing slab geometry,
fetch/reclaim and MM-owner assertions. A new
`tests/diagnostics/bcache-low-v1819/run-v1819.sh` builds a negative
source mutation restoring the historical HIGH allocation argument.
The actual production source passes the PDP-6 simulator fixture,
and the HIGH mutation is rejected by the native exit assertion.

Command: `make test-bcache-low-placement-v1819` in daimos-testkit.
Source test commit: `b12bb77` on testkit main. Nine stale v1636
tracked/untracked historical candidate files, including duplicate
six-child physical-pressure scripts, were archived under
`~/backups/daimos-bcache-pure-six-v1636-retired-v1819-20261123/`
with verified SHA256 and a retained tracked-file patch; that
detached worktree is clean. The later committed original-guest
`pure-cache-low-v1636`, `pure-cache-pressure-v1637` and
`pure-six-matched-v1639` suites remain the physical evidence owners.

**Not claimed:** fresh physical 48K/32K/64K PURE acceptance, a
whole-system memory measurement, or an end-to-end DTFS guest run.
