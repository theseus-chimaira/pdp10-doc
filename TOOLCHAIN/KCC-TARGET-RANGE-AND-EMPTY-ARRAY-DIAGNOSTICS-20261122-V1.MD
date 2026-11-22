# KCC TARGET FLOATING RANGE AND EMPTY-ARRAY ICE DIAGNOSTICS (V1)

Date: 2026-11-22.
Status: fixes implemented, built, installed, regression-tested,
committed and pushed.

## NONFINITE AND OUT-OF-RANGE FLOAT CONSTANTS

The GCC upstream testcase
`tests/semantic/upstream/generic/960405-1.c` uses a long-double literal
with exponent 4931.  KCC's host floating representation cannot encode that
value into the PDP-10 F or D formats.  Previously,
`host_pdp10_double_words` in `ccout.c` treated the resulting host
nonfinite value as a compiler internal error.

The D encoder and shared F encoder (`ccoututil.c`) now report normal
floating-range source diagnostics for nonfinite host values or target
exponent overflow.  No PDP-10 NaN or infinity bit pattern is invented.
Compilation with a diagnosed out-of-range source remains unsuccessful,
rather than being counted as a pass.

The original upstream testcase was rerun through 22 applicable KCC
profile cells.  All 22 produced ordinary `compile_fail` with zero
`compiler_ice`.  Two explicitly checked profiles (`base` and `pdp6`)
produced `Floating constant overflow` without an internal error.
As a compatibility check, three established, representable
floating-point regression programs passed all 54 selected KCC cells.

Source commit: KCC `d4004d6`.
Registered diagnostic test: pdp10-testkit `808b468`,
`misc/validation/test-kcc-fp-range-diagnostics-v1.sh`.

## GNU EMPTY INITIALIZER WITHOUT ARRAY BOUND

GCC's upstream torture source
`tests/codegen/upstream-gcc/compile/920501-22.c` contains
`int y[]={};`.  In KCC, array bound zero also denotes an incomplete
array; the compiler does not distinguish a complete GNU zero-length
array extension from an incomplete array.

Previously an attempted zero-word aggregate copy caused
`simsmove: bad size 0`, while static initializer emission could cause
`gizlist: bad size: 0`, both reported as internal compiler errors.

A zero-word `simsmove` is now an exact no-op rather than an invalid
BLT.  When an empty initializer cannot determine an array bound,
`gizlist` issues a normal source diagnostic:
`Empty array initializer cannot determine array size`.
The unsupported source is rejected, not silently accepted with
an incomplete array layout.  The original upstream testcase now
produces 22 `compile_fail` cases and zero ICEs across the selected
KCC matrix profiles.

Source commit: KCC `daa6670`.
Registered diagnostic test: pdp10-testkit `ea854dc`,
`misc/validation/test-kcc-empty-array-diagnostic-v1.sh`.

## REPRODUCTION AND ACCEPTANCE LIMITS

Both tests operate on the original upstream C sources, without editing
their contents or test expectations.  Normal strict matrix policy still
classifies diagnosed unsupported sources as compile failures, not PASS.
The special diagnostics assert that the compiler exits unsuccessfully
with the expected source diagnostic and no internal compiler error.

Testkit inventory, script ownership, raw p10run caller ownership,
and registered-fixture integrity audits all pass with zero issues.

This removes the internal-error classification from 44 previously ICE
cells.  The remaining GCC and KCC ICEs, other linker failures, and
unrelated timeouts require independent repairs.  No full-matrix
reclassification is claimed.
