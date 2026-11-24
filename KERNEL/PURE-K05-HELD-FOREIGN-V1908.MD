# PDP-6 K05 v1908: unrepaired foreign FULL target and shared retry cap

Date: 2026-11-24. **Original PDP-6 48K physical PASS, bounded scope.**

The v1906 PURE FULL-recovery fix records the original temporary
PROCESS target base in the right half of the *existing* private
u-area word 067; its left half keeps the existing retry bit.
The v1907 physical guest then demonstrated genuine user TERM
refusal while a foreign owner claims the full unfreed target.

v1908 asks only whether a longer unrepaired foreign owner
remains quarantined while the already-existing slot-0 idle
retry backoff reaches its cap. This is a ten-heartbeat bounded
test, not an assertion about unlimited time or a new terminal
cleanup mechanism.

The exact original v1907 source-built 48K PDP-6 KCORE and PURE
boot the original compressed TSFS five-child PIPE/BRK/RUN
pressure workload. Native first `mm_free(PACKED)` returns -5,
the external RAW source pin is released, and one test-only
deposit changes the full target PROCESS at `120000..130000`
from owner3 to foreign owner2. Original parent user `PROCCTL`
TERM(3) returns `TR` (refused). Unlike v1907, the controller
NEVER repairs the foreign MM owner, releases the target,
opens the typed parent WAIT gate, or forges FINALIZE.

Ten independent resident user `HB` checkpoints physically
preserve exactly MM18 and all MM owner/type/pin/extents,
including RAW `111200..120000` owner3 pin0 and the full
PROCESS `120000..130000` owner2 pin0. Slot3 remains
TRANSITION-held, VMbase0, with original private executive
CTX_PC, FDCTL and escrow `040000120000` in u[067]. The
original parent remains active (slot1); ROOT, unrelated
owners and WAIT gate remain unchanged. Actual PCLK increases
from `015624` to `016724` octal across ten heartbeats.

The **one existing fixed BSS word**
`proc_finish_last_tick_v881` holds the shared pending bit,
last epoch and capped exponential delay. Physical delay
readings at ten checkpoints (all octal ticks):

    0100 0200 0200 0400 0400 0400 0400 1000 1000 1000

The original `01000` octal maximum (512 decimal ticks) is
reached and remains constant across three consecutive HBs.
Original parent progress continues; there is no new per-owner
retry allocation and no forced MM free.

**The simulator is operator-stopped at HB10.** No native guest
EXIT, typed WAIT completion, recovery or HALT is claimed for
this deliberately unrepaired owner. An observed finite interval
cannot prove the safety of an infinite lifetime.

The source-backed physical audit checks exact original MM
tables, real source-free -5 and refused TERM, all ten saved
CPU/FDCTL/escrow/TRANSITION/VM0 snapshots, ROOT, parent
progress, exact shared retry pending, rising hardware clock,
nondecreasing delay and `01000` plateau. Three focused unsafe
physical transcript mutations (false TRANSITION unlock,
delay above cap, stalled parent PCLK) all reject. Existing
v1906 16 MM and v1907 four TERM controls are not redundantly
rerun because production binaries have not changed.

Source-built v1908 uses unchanged production binaries:

- KCORE SHA256 `a201a7fdcf481b436b06694e5dd55c9367036ebd9f64c630486f854b1a4f89be`
- PURE MRES SHA256 `09f65d0ca272986b462c63580eae3c4006490004cd060529ce40f10a55949351`
- Accepted original physical SHA256 `6d5a43084392f9a32442dc7cb706d8d0e4e451aac0cc029784d35ba810954736`
- KCORE IMAGE `024167`, BSS `000415`; PURE IMAGE `004761` octal, unchanged

Testkit:
`daimos-testkit/tests/validation/pure-k05-permanent-owner-v1908/`.
DAIMOS analysis:
`DAIMOS/design/kcore-k05-long-held-owner-v1908.md`.
Accepted physical evidence:
`$TMPDIR/daimos-k05-permanent-owner-v1908-physical-v1/`.

**K05 remains OPEN:** terminal cleanup of a permanently unknown
or foreign MM owner cannot discard its still-live saved
executive continuation; additional allocator/PI6 ambiguities
remain. No new production code, RAM or policy was added.
RAW PURE and mode3 remain disabled by default; K06-K09 open.
