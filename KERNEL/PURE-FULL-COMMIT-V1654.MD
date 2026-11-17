# P1-E v1654: original PDP-6 atomic FULL MM commit

Date: 2026-11-17. Real production opt-in PURE MRES correction.
Source-linked native PDP-6 MM error matrix and newly rebuilt
original physical 48K secondary-DTC3 read-failure/recovery PASS.
No fixed KCORE, process, MM metadata or syscall ABI growth.

## Correct FULL source/destination ownership

The original v1391 FULL restore held the PACKED RAW source and
the separate reconstructed PROCESS destination pinned through
the bounded loader, but deleted the authoritative RAW source
and published RESIDENT before its final fallible target unpin.
If that unpin returned MM_ERR_BUSY, the process remained
TRANSITION-held RESIDENT with a pinned APR and no original
PACKED image to retry. That ordering was unsafe.

v1654 unpins the complete destination FIRST, while the RAW
source is still authoritative, intact and pinned. The short
PI-disabled MM commit then unpins and frees that source,
publishes the restored APR and RESIDENT storage, clears
backing and TRANSITION, and reenables PI. No I/O or
allocation happens with PI disabled.

A one-shot target-unpin refusal rolls back to exactly one
unpinned PACKED owner and unchanged saved executable backing;
retry succeeds with full immutable TEXT and exact private
writable suffix. Persistently refused source or destination
unpin retains TRANSITION, raw source ownership and PACKED
state; the safe refusal is NOT claimed to automatically
resolve permanently uncertain MM pins.

The original v1395/v1397 and v1648 native PDP-6 regressions
still PASS; two new native permanent-unpin cases PASS.
A separately compiled and executed test-only mutation
restoring post-publication target unpin is REJECTED.
Testkit:
tests/validation/pure-full-commit-v1654/,
branch test-kcore-p1e-full-commit-v1654-20261117,
commit 01a6932.

## Actual newly rebuilt physical 48K validation

Original source-built GCC PDP-6 48K/no-AUX, selected TSFS
root and independent native DTC3 DTFS compressed executable
produces six real PACKED children. At bounded FULL address
035261 octal, slot3 owns one RAW suffix 111200..120000
and a separately allocated full PROCESS APR 120000..130000,
both pin1. Physical removal of only DTC3 causes the original
bounded read to return -4; exact RAW unpins and full target
frees without losing selected ROOT. The same physical reel
reappears and another FULL succeeds, six original PIPE/
DATA/EXIT-057/PID/WAIT checks pass, no child MM owner is
left and the simulator HALTs normally. Two forged source pin
and loader-success physical witnesses are correctly rejected.

Only the exact changed MRES loader offset was updated in
the testkit's private original-v1643 replay: 003224 ->
003233. Its original entire physical state oracle remains.
Permanent GCC KCORE map low end is still 024253 octal
and the fixed KCORE stream remains byte-identical with
the previously accepted release test:

SHA256 88970ee633dc22094565172348bc1dcb9f49ba0417e104fb5608b71a87668d01.

The optional linked PURE-on transient KINIT end moved
113621 -> 113631 octal (+8 decimal words); no resident
KCORE RAM increase.

Complete DAIMOS report:
design/kcore-p1e-full-commit-atomic-v1654.md.
Actual logs, linker maps, DXRs and media identity:
$TMPDIR/daimos-p1e-v1654-evidence-20261117.tar.gz
SHA256 294ea909464d5d42226042296affeb8a7c1f768d2f035873b62354afe47d438d.

## Current completion boundary

Strict 32K boot/footprint acceptance is now explicitly
DEFERRED until after remaining PDP-6 KCORE functionality
and the matched-source functional freeze. Original v1653
6289-word KINIT overflow remains an unsolved future goal.
Do not remove required MRES, weaken stack reserve or call
the oversize build a 32K physical acceptance.

Active finish gates: permanently uncertain MM owner policy,
whole-machine physical peak including KCORE/MRES/BCACHE,
actual optional mutable-suffix compression net-RAM YES/NO
decision, current-source DTFS stale-fixture modernization,
complete original native/physical release freeze and RAM
budget report. KA10/KI10 ports are NOT accepted and are not
PDP-6 KCORE-release prerequisites. Final code-size
optimization is LAST, after feature freeze.
