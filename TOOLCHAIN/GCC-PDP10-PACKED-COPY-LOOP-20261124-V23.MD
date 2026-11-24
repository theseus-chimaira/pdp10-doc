# GCC PDP-10 packed argument copy-loop termination v23

Status: verified bounded compiler repair, 2026-11-24.

## Cause

PDP-10 packed BLKmode by-value arguments are copied by an inline
LDB/DPB/IBP loop. The original GCC expander emitted a separate
`count -= 1` RTL operation before its conditional backward branch.
In a function-argument/libcall sequence, the decrement could be
moved into the preheader even though the loop still tested the old
count. This made the byte-copy loop unbounded.

## Fix

The backend now emits one existing combined decrement-and-branch
RTL operation, lowered as the PDP-10 `SOJN` instruction. It keeps
the counter update and branch inseparable; the code is shorter,
faster, and needs no additional scratch accumulator. Source:
`gcc-pdp10/gcc/config/pdp10/pdp10.c`.

## Verification boundary

The original 44-cell GCC packed-byvalue test was rerun after a
compiler rebuild. The 27 old timeouts disappeared, replaced by
normal runtime failures from a separate packed-field/argument ABI
defect. The original source is **not** passing: all 44 cells still
fail their field-value oracle. No test was weakened.

A new permanent C89 testkit regression for the counted argument
copy with 3, 6, 11 and 20-byte packed records passed **36/36 GCC**
cells. It checks loop termination and call progress, not the
independent field extraction semantics. Prior v19-v22 tests are
checked separately: 284/284 GCC/KCC PASS under strict exit policy,
with no skipped old PASS statuses.

The historical 135 timeout cells now comprise 113 confirmed
repaired/invalidated and 22 still unresolved optimized KCC cases.
Details: `pdp10-testkit/history/PDP10-GCC-PACKED-COPY-LOOP-TIMEOUT-20261124-v23.md`.
