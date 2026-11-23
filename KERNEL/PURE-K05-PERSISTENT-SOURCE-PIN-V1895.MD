# PDP-6 K05 v1895: persistent native MM source-free refusal and parent progress

Date: 2026-11-23. **Scoped original 48K full-system physical PASS**.
K05-K09 OPEN. RAW PURE/mode3 remain OFF by default. No production
fixed KCORE or optional PURE MRES code/RAM growth.

## Three authoritative source-free failure cases

The original v1839-v1855 PURE FULL transaction must never free
the last authoritative mutable suffix or publish an incomplete
PROCESS APR merely because `mm_free(PACKED)` reported failure.
The real original source-linked MM extents provide the only
authoritative decision:

* Sole exact unpinned full PROCESS successor: physical source
  deletion committed even if the reply is -5. Publish target.
  v1893 original physical PASS.
* Both exact original PACKED and PROCESS owners exist and BOTH are
  unpinned: source free genuinely refused; free only provisional
  target, preserve PACKED source. v1894 original physical PASS.
* Original source still pinned, with complete target unpinned:
  uncertain physical owners. **Do not release either owner or
  publish an APR.** v1895 original physical PASS with two new
  independent resident parent heartbeats.

This last case is distinct from v1890-v1892 target overpin: here
the genuine original `mm_free(PACKED)` itself executes and
returns its OWN `MM_ERR_BUSY=-5`, not a forged AC1 error, and
the external pin belongs to the PACKED RAW source.

## Physically executed original PDP-6 FULL transaction

Use the original source-built fixed KCORE and removable PURE
MRES of v1855, unchanged. The original 48K user fixture uses
the accepted v1892 testkit-only INIT, preserving all original
five-child RUN/BRK/PIPE pressure, writable sentinel checks and
secondary compressed DTC3 backing; only the already-resident
parent emits repeated new user-mode `HB` after the normal `L`
pressure milestone instead of blocking in WAIT for the
deliberately quarantined child.

The original installed PURE base is physically measured from
the original KCORE reclaim JRST. Compiled source-linked FULL
DOBJ symbol/relocation entries and linked MRES instructions prove
the actual runtime call/return in original PDP-6 physical core
(octal):

* bounded original FULL loader entry `036202`;
* actual original PACKED-source `PUSHJ 17,mm_free` `035310`;
* original native `mm_free` return consumer `035311`;
* original FULL service return `033067`;
* original source metadata physical word `024703`.

The original FULL pinned checkpoint has 18 MM extents, including
slot3 RAW PACKED source `111200..120000` pin1 and provisional
full PROCESS `120000..130000` pin1, as well as separately
resident parent slot1 PROCESS `130000..140000` pin0. The loader
completes and both original transaction pins are successfully
removed. At original source-free entry, BOTH slot3 descriptors
are present pin0, and original source meta is
`024703: 000004000003`.

Only the private SIMH controller adds ONE external physical pin
at that exact native call boundary:

    024703: 000004000003 -> 000014000003

Original unmodified fixed-KCORE `mm_free` genuinely refuses the
pinned source, returning AC1 `777777777773` = `MM_ERR_BUSY=-5`.
The native MM table retains all 18 descriptors, including source
pin1 and target pin0. **Unlike v1894, the external source pin
is NEVER released.** The original v1839 zero-pin owner scan
cannot establish exact unpinned source/target pair, so it
correctly returns -5 at original FULL service `033067` without
freeing either reservation or publishing a runnable APR.

The actual original slot3 process word `044723` is
`046031600401`, with TRANSITION set, and VM word `044724` is
`010000000000`, physical base zero. Original selected TSFS root
`070101000000` and the **entire** 18-entry original MM owner/pin
table remain unchanged throughout the native -5, the original
quarantined FULL service return and two later new user `HB`
checkpoints.

The parent slot1 actually executes both user-mode `HB` syscalls
after the genuine failed FULL, with `proc_current_slot=1`,
its valid scheduler state and original resident PROCESS APR
`130000..140000` pin0. The original PCLK is `016640` octal
at the failed service and first HB, then **`016740`** at the
second new HB. Do not misreport a clock advance BEFORE HB1:
it is the two newly emitted independent HBs that prove the
original later clock actually advanced. The first testkit-only
verifier incorrectly demanded a tick between service and HB1.
That check was corrected; the original unchanged physical
trace then PASSed all original hardware owner/clock assertions.

## Source-backed original physical audit

| Physical checkpoint | Slot3 PACKED | Slot3 PROCESS | MM extents | PCLK |
|---|---|---|---:|---:|
| Original pinned FULL | pin1 | pin1 | 18 | -- |
| Actual original source-free call | pin0 | pin0 | 18 | -- |
| Genuine native original MM -5 | pin1 | pin0 | 18 | -- |
| Original FULL -5 / TRANSITION held | pin1 | pin0 | 18 | `016640` |
| First real parent user HB | pin1 | pin0 | 18 | `016640` |
| Second real parent user HB | pin1 | pin0 | 18 | `016740` |

The independent source-linked physical audit verifies original
KCORE/PURE binary hashes, real MM call/return PCs, genuine -5,
original source/target MM exact owner/type/size/base/pin,
slot3 TRANSITION/VM0, full unchanged MM table and TSFS ROOT,
parent slot1 resident APR/dispatch/state, TWO new real HB
outputs, and original PCLK progress. TEN separately executed
unsafe owner/pin/process/return/heartbeat/clock transcript
mutations are rejected at intended code-path checks.

Original normalized physical PDP-6 transcript SHA256:
`8f2bb94995d3522b076c845f5e784cb7f682b5cb565ae553b24b717342f62c92`.
Unchanged original fixed KCORE DXR SHA256:
`33bf700218a73fefcb36ebb1f1d835c1817f3a291a642898ccb3cbd5d9039d8b`.
Unchanged optional PURE MRES DXR SHA256:
`a0fb25c51cf1c57f4bc70ca1dc83d8b9ea39b1ac3ddb3bba7b9ffcea17e5b3fd`.

Original private simulator evidence:
`$TMPDIR/daimos-k05-persistent-source-pin-v1895-real/`.
Design and precise original owner table:
`design/kcore-k05-physical-persistent-source-pin-v1895.md`.
Testkit implementation and executable acceptance:
`tests/validation/pure-k05-persistent-source-pin-v1895/`, targets
`make test-pure-k05-persistent-source-pin-physical-v1895` and
`make test-pure-k05-persistent-source-pin-audit-v1895`.

## What remains

The private simulator exits after TWO real new parent HBs. This
is deliberately NOT original five-child EXIT/WAIT or HALT.
Original quarantined slot3 cannot have its PROCESS/RAW extents
released without an independently verified owner transition.
Actual eventual source unpin, terminal cleanup of permanently
quarantined owners, other PACKED children restoring under 48K
pressure, permanently refused target rollback, unknown allocator
or foreign owner outcomes, arbitrary PI6 timing, 64K/KA10 and
K05-K09 release acceptance all remain OPEN.
