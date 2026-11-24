# PDP-6 K05 v1909: real clock PI6 with quarantined foreign FULL target

Date: 2026-11-24. **Scoped original 48K PDP-6 physical PASS.**

The optional RAW PURE v1906 recovery remembers the exact
original provisional FULL target base in the right half of
the original u-area word 067, with the existing swap-retry
marker in the left half. v1907 proved genuine user TERM refuses
against this still-live executive continuation; v1908 proved
ten parent HBs under an indefinitely held foreign owner, with
MM18 unchanged and the original shared retry backoff capped.

v1909 inserts exactly one actual 60Hz **PCLK PI6** observation
between the existing HB4 and HB5 checkpoints. It uses the
same original source-built compressed TSFS five-child guest,
fixed KCORE, optional PURE MRES and deliberately unrepaired
foreign owner of the full temporary PROCESS target.

Original source `mm_free(PACKED)` returns true -5, the target
owner is test-retagged owner3 -> owner2 without changing base,
size, type or pins, and original parent TERM(3) returns `TR`.
Parent HBs 1..10 keep source RAW, foreign full target, all MM18
extents, ROOT, child TRANSITION/VMbase0, executive CTX_PC/FDCTL,
and private u[067] escrow `040000120000` unchanged.

At the original physical PI6 breakpoint:

| Original witness | Octal value |
| --- | --- |
| Permanent `proc_sched_pi_tick` PC | `017225` |
| Actual instruction (`AOS 1,proc_timer_clock`) | `350040024530` |
| Hardware-saved `pdp10_pi_level6` address | `023714` |
| Actual saved PI6 return word | `340000017560` |
| Interrupted original executive PC | `017560` |
| Linked original slot-0 idle-loop range | `017552..017620` (exclusive) |
| Actual `proc_current_slot` | `0` |

This event is from the real original PDP-6 PCLK/PI6 entry;
no timer, kernel status, interrupt frame, ownership cleanup,
or MM return value was fabricated by the test controller.
The full MM18 physical table and saved private context are
identical before, during and after the interrupt. The parent
continues to HB10 and the original slot-0 retry reaches/holds
the existing `01000`-octal-tick (512 decimal) upper bound.
The simulator then stops on **operator command**; this is NOT
native completion of the intentionally unrepairable target.

The complete v1908 positive MM/TERM/backoff oracle re-PASSes.
Exactly three new independent physical trace mutants are
rejected: incorrect PI6 instruction, false saved interrupted
PC, and lost target-base escrow at PI6. Existing earlier
MM/TERM/backoff negatives are not redundantly rerun with
unchanged production binaries.

The original source-level PI6 path also limits the suspected
tag/escrow-store race. `proc_sched_pi_tick` permits a slot-0
context switch ONLY when the saved interrupted PC is inside
the original idle loop; ordinary executive code cannot switch
at an asynchronous PI6. Thus a clock tick while the PURE
arm helper executes cannot independently launch another
idle reaper before the same executive continuation stores
the target escrow. This structural argument is distinct
from v1909's physically observed idle-loop PI6 ordering.

Unchanged original SHA256:

- KCORE `a201a7fdcf481b436b06694e5dd55c9367036ebd9f64c630486f854b1a4f89be`
- PURE MRES `09f65d0ca272986b462c63580eae3c4006490004cd060529ce40f10a55949351`
- Accepted original physical trace `fbe72332fc0540331c9d4b131d0efded53e42d3da5b746b089ecc99f0f4cbb52`

No production C/ASM, KCORE/PURE IMAGE, BSS, process storage,
or default feature policy changed. Testkit:
`daimos-testkit/tests/validation/pure-k05-held-pi6-v1909/`.
DAIMOS design:
`DAIMOS/design/kcore-k05-held-pi6-v1909.md`.
Accepted physical evidence:
`$TMPDIR/daimos-k05-held-pi6-v1909-physical-v1/`.

**K05 remains OPEN** for other PI6 timings and terminal MM
ownership uncertainty; no forced FINALIZE on an unreleased
foreign target or live saved executive stack. RAW PURE and
mode3 remain default OFF; K06-K09 also OPEN.
