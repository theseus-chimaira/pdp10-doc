# PDP-6 K05 v1898: two genuine target unpin failures and safe quarantine

Date: 2026-11-23. Scoped **original 48K PDP-6 physical PASS**.
K05-K09 remain OPEN, RAW PURE and mode3 OFF by default. Original
fixed KCORE and optional PURE MRES code/RAM footprint unchanged.

## Original FULL unpin contract

The original `proc_pure_ram_restore_v1391` fully reconstructs a
temporary PROCESS APR while the authoritative PACKED mutable
suffix remains retained. Before commit, the original v1855
PI-atomic owner check requires exactly one live pin each on RAW
source and complete PROCESS target. The transaction first calls
`mm_unpin(target)`. If it fails, `fail_target` must not assume
target release or free it speculatively; it retries its own
target pin release and, if that too fails, returns BUSY while
keeping the child TRANSITION and physically unpublishable.

The original fixed KCORE `mm_unpin` is a real pin-counter
operation. Attempting to unpin an already-zero MM descriptor
returns `MM_ERR_INVAL=-4` and changes nothing. v1898 deliberately
forces that unexpected state at an original call boundary to
test both actual native failure paths. **This test-only debugger
deposit is inside a normally PI-disabled section.** The normal
OS scheduler and interrupt-side MM code could NOT remove a pin
there after the v1855 exact-one-pin guard. The scenario tests
defensive handling of an abnormal MM state, not an anticipated
natural PI6 race.

## Real original source-linked physical execution

The original source-built fixed KCORE/PURE v1855 and original
five-child RUN/PIPE/BRK pressure from root and secondary compressed
DTC3 files are used unchanged. The accepted v1892 test-only
parent fixture continues already-resident user `HB` instead of
waiting for a child deliberately retained in quarantine.

Original KCORE reclaim JRST physically locates installed PURE
base `032417` octal. The original compiled FULL function symbol
`.text+000504` and **four** actual `mm_unpin` relocations
`.text+000655`, `.text+000662`, `.text+000746`, `.text+000766`
are checked against original linked `PUSHJ 17,mm_unpin` opcodes.
Native runtime PCs (octal):

* first original target-unpin call `035271`, return `035272`;
* original `fail_target` target-unpin retry `035362`, return
  `035363`;
* original FULL service return `033067`.

At original first target-unpin entry, the actual physical MM
table has 18 records, including source PACKED `111200..120000`
pin1 and target PROCESS `120000..130000` pin1, both owned by
slot3. The test-only SIMH controller changes ONLY target MM
metadata at original physical word `024705`:

    from 000011000003 (PROCESS, slot3, pin1)
    to   000001000003 (PROCESS, slot3, pin0)

The unmodified original `mm_unpin(target)` REALLY returns AC1
`777777777774` (-4) at original PC `035272`, with source pin1,
target pin0 and all 18 extents still physically present. The
original FULL error branch enters the real `fail_target` retry
at `035362`, whose target STILL has zero pins and whose original
unchanged `mm_unpin` AGAIN returns genuine -4 at `035363`.
The original FULL service then returns `MM_ERR_BUSY=-5`, without
freeing either retained MM owner or ever publishing a valid
RESIDENT APR for the quarantined child. Slot3 process meta at
`044723=046031600401` remains TRANSITION; VM word at
`044724=010000000000` retains physical base0.

The complete original 18-entry MM descriptor table is identical
from the first actual unpin failure through both original
`mm_unpin` returns, original FULL service and two later genuine
user-mode parent `HB` outputs. The authoritative PACKED source
is pin1, temporary full target pin0, all unrelated owners,
pins, geometry and selected TSFS ROOT `070101000000` octal
unchanged. Parent slot1 remains independently resident PROCESS
`130000..140000` pin0 and SRUN; actual current slot is 1 at
both new HBs, with original PCLK advancing from `016325` to
`016425` octal between them.

## Original physical MM checkpoints

| Physical checkpoint | MM count | PACKED source | PROCESS target |
|---|---:|---|---|
| Pinned FULL / first target unpin entry | 18 | pin1 | pin1 |
| Private target zero-pin injection | 18 | pin1 | pin0 |
| First genuine native `mm_unpin` -4 | 18 | pin1 | pin0 |
| Second genuine native cleanup `mm_unpin` -4 | 18 | pin1 | pin0 |
| Original FULL returns -5, TRANSITION/VM0 | 18 | pin1 | pin0 |
| Two new independent parent user HBs | 18 | pin1 | pin0 |

The independent physical testkit oracle checks compiled/linked
native PCs, both original typed -4 returns, exact original MM
table and process TRANSITION/VM0, preserved executable backing
and ROOT, resident parent APR/SRUN/current slot, and true
PCLK advancement across two original user-mode HBs. **Thirteen
separately executed unsafe physical transcript mutants** are
rejected at their intended source/target pin, return, process,
current-user, PCLK and second-heartbeat checks.

Normalized original 48K PDP-6 physical trace SHA256:
`c7d5350098f69474e1fd2398c0822d3abf11bd21ad754c5a69d973a7ef3e215e`.
Unchanged original fixed KCORE DXR SHA256:
`33bf700218a73fefcb36ebb1f1d835c1817f3a291a642898ccb3cbd5d9039d8b`.
Unchanged optional PURE MRES DXR SHA256:
`a0fb25c51cf1c57f4bc70ca1dc83d8b9ea39b1ac3ddb3bba7b9ffcea17e5b3fd`.

Original simulator evidence:
`$TMPDIR/daimos-k05-target-unpin-refusal-v1898-real/`.
DAIMOS design and physical state:
`design/kcore-k05-physical-target-unpin-refusal-v1898.md`.
Executable testkit source and read-only independent audit:
`tests/validation/pure-k05-target-unpin-refusal-v1898/`.
Targets `make test-pure-k05-target-unpin-refusal-physical-v1898`
and `make test-pure-k05-target-unpin-refusal-audit-v1898`.

The original private controller exits after two new parent HBs;
this is NOT a successful child WAIT/HALT or terminal MM cleanup
proof. Source-unpin failures, naturally occurring PI race,
foreign MM owner, permanently quarantined terminal teardown,
other PI6 timings, 64K/KA10 and K05-K09 release remain OPEN.
