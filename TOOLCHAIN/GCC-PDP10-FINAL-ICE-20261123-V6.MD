# GCC PDP-10: Last Byte-Pointer and SF Comparison ICEs

Date: 2026-11-23.

## Problem and correction

The unextended PDP-10 pointer comparison lowering in
`gcc/config/pdp10/pdp10.c:pdp10_emit_cmp_and_jump_insns`
used `gen_xorsi3` to invert the byte-pointer S field before a
six-bit rotate and signed comparison. This was not legal RTL if a
pointer operand was an immediate constant: the instruction's
source is constrained to register or memory. It was observed as
an unrecognizable `XOR(CONST_INT,CONST_INT)` in original upstream
test `20010329-1.c` at `-O0`.

The correction uses `simplify_binary_operation` for a constant XOR
and rotate, then materializes the value in a register. For an
unfoldable or dynamic input it ensures a register-or-memory source
before emitting the normal XOR and rotate. Extended-pointer
comparison lowering is unchanged.

Separately, three special SF compare-to-zero patterns in
`gcc/config/pdp10/pdp10.md` were too broad. `*snesf`, `*sltsf`,
and `*sgesf` recognized any immediate floating-point constant, but
used instruction alternatives and templates valid only for zero.
This led to `reload.c` being unable to satisfy the constraints for
nonzero values. All three now use the existing
`pdp10_const_double_0_operand` predicate, so nonzero constants
are handled by the generic SF comparison patterns.

## Validation

The staged compiler rebuild regenerated the machine-pattern output.
The previously failing original GCC tests compiled successfully:
`tests/semantic/upstream/generic/20010329-1.c` and
`tests/semantic/gcc-historical-b01-v1/20020321-1.c`.

The official `pdp10-testkit` cell runner checked all 414 originally
recorded GCC ICE cells. All 14 remaining ICEs changed to passes,
with zero regressions in the other 400 cells. Final state:

- 0 compiler ICE
- 205 pass
- 176 diagnosed compile failures
- 33 guest runtime failures

The verified status file is
`pdp10-testkit/workdir/ice-retest-20261123-v9/results-v1.tsv`;
the detailed testkit report is
`history/GCC-FINAL-ICE-RETEST-20261123-v6.md`.

These observations apply only to the 414-cell original GCC ICE
baseline and the staged compiler. The installed compiler and
full testkit have separate acceptance gates; other priority
categories remain open.
