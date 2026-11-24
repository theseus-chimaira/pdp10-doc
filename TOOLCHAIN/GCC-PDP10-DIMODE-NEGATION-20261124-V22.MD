# PDP-10 doubleword negation control-flow correction v22

Status: verified GCC runtime/compiler correction, 2026-11-24.

## Fault

The old GCC PDP-10 backend negated a doubleword using a conditional
branch such as `JUMPE low,[AOJA high,.+2]`. Brackets place an
instruction in the assembler's literal pool, rather than inline
with the machine-description expansion. The AOJA target is relative
to the pool location, so a zero low word could execute into an
unrelated or uninitialized address. The result was a guest loop at
PC 0, not simply a computationally expensive wide-integer program.

In the baseline compiler matrix, all 36 GCC runtime profiles of
upstream test `950612-1.c` timed out. Extending one KI10 O2 attempt
to 200 million guest instructions did not resolve it. Reducing the
source isolated failures to extended signed/unsigned long-long
absolute value operations, while 36-bit integer versions passed.

## Correct code generation

The backend now implements the negation carry through an inline
`SKIPN` followed by `ADDI` of the high word when the low word is
zero. Negating the low word then preserves low zero or computes
the ordinary two's-complement low value. The 71-bit variant still
normalizes the low-word sign-copy bit. Separate-destination and
in-place patterns have consistent instruction lengths.

No scratch accumulator, runtime helper or literal-pool branch is
needed. Implementation is in `gcc-pdp10/gcc/config/pdp10/pdp10.md`.

## Verification

The corrected full upstream source passed staged O2 PDP-6, KA10,
KI10 and KS10 execution. After GCC was rebuilt and installed,
the official PDP-10 testkit runner executed the wide-integer source,
both corrected v21 timeout sources and the v19/v20 byte-pointer
regressions. **284/284 GCC/KCC cells PASS** under strict policy,
with no previous PASS statuses skipped.

Combined with earlier repairs, **86/135 original timeout records**
are now confirmed repaired or correspond to a corrected undefined
test oracle. The remaining 49 belong to GCC packed aggregate
by-value (27) and two KCC optimizer cases (11 each); none is
accepted as a false positive without independent evidence.

Further source-linked details:
`pdp10-testkit/history/PDP10-WIDE-INT-NEGATION-TIMEOUT-20261124-v22.md`.
