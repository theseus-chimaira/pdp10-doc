# KCC 71-bit multiply scratch-register liveness v25

Status: verified compiler correction, 2026-11-24.

## Root cause

PDP-10 hardware `MUL ACx,src` deposits a two-word result in ACx
and ACx+1. KCC's 71-bit multiply routine chose its result pair
using `rfree()`, which tracks expression VREG ownership only.
Callee-preserved register variables in AC10..AC14 are not tracked
as expression VREGs. One 71-bit arithmetic expression therefore
used AC7/AC10 for its product while AC10 still held a structure
result pointer. The product overwrote the pointer, causing a
subsequent store through invalid memory and an execution timeout.

## Correct accumulator allocation

`ccgen2.c` now limits scratch AC selection to the actual volatile
expression register set (normally AC1..AC7), respecting incoming
argument hold masks. `gdimodemul()` reserves the required adjacent
product pair *before* selecting its one-word cross-product scratch.
If the current allocation is too fragmented, the established
VREG spill/reload mechanism is used to reallocate a legal pair.
The common path has no new stack allocation or extra instructions;
preserved register variables cannot be clobbered by scratch usage.

The restoring 71-bit divider is an intentional exception: it requires
up to eight scratch ACs plus its four input words. A first global
volatile-only restriction triggered 22 KCC compiler ICEs in the
existing DImode div/mod regression and was discarded. The final
compiler has a separate scratch selector for this divider, limited
to AC1..AC14. It may borrow preserved AC10..AC14 only because the
divider already pushes and restores each such register explicitly.
Normal inline 71-bit multiplication and comparisons cannot borrow
those registers, and neither path touches reserved AC15/16.

The separate division scratch path was validated in a fresh,
20-source KCC long-long/DImode matrix: **440/440 PASS** under strict
exit policy, including all 22 div/mod cells that the discarded
volatile-only allocator variant had caused to ICE. No previously
green cell was skipped. This was checked with the installed compiler,
not a hypothetical register-allocation simulation.

The final four-source timeout acceptance was repeated after the
last rebuild: **88/88 PASS**, yielding **528/528** fresh KCC cell
executions across the two independent selections. The installed
`$HOME/cross/bin/kcc` SHA256 is
`1045034ddd9cc98b61232366bbbe291d3d1f2d5d2b4e48d3f5cfce578927a64f`.

## Validation

The reduced `pdp10-kcc-wide-multiply-preserved-ac-v25.c` retains
the original 71-bit multiply and live structure pointer. Old KCC
OPT loops; corrected KCC passes. The original upstream
`20001203-2.c` and new regression each pass all 22 KCC profiles.
Together with the v24 comparison fix, **88/88** selected target
execution cells pass under strict policy, with no skipped results.
These repairs eliminate the final 22 timeout checkpoints from
the 135-failure historical timeout snapshot, without treating
unrelated GCC packed-argument ABI failures as resolved.

Testkit evidence:
`history/PDP10-KCC-MULTIPLY-PRESERVED-REGISTER-20261124-v25.md`.
