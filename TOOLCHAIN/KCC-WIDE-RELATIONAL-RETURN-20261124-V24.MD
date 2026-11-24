# KCC wide relational conditional return correctness v24

Status: verified compiler fix, 2026-11-24.

## Problem

KCC's 71-bit signed and unsigned relational comparison generator
emits the final true-edge skip as `JUMPN flag,.+2` through a raw
assembler `codestr()` call. The false-edge `P_JRST` is emitted
afterwards by `gboolean()` into the buffered peephole representation.
The optimizer cannot see the raw conditional skip in the PCODE
buffer, so a following `return` under the condition could be
misclassified by `deadjump()` as dead code after an unconditional
JRST. This removed a reachable return statement and caused a
nonterminating interval-search loop.

## Correction and verification

The KCC `ccgen2.c:gboolean()` wide-relational path now flushes the
buffered false-edge jump before generating following statements.
This reconciles the raw comparison control flow with the optimizer's
dead-code view; ordinary single-word comparisons remain unchanged.
There is no additional guest scratch register or runtime module.

The old optimized KCC timed out with
`pdp10-kcc-wide-compare-return-v24.c`, while the repaired KCC
passes. The original upstream `20020402-3.c` now passes all
**22/22** KCC compiler/CPU/optimization profiles, as does the new
focused regression. The original 11 optimized timeouts are gone.

Testkit evidence:
`history/PDP10-KCC-WIDE-CONDITIONAL-RETURN-20261124-v24.md`.
