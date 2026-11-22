# KCC FLOAT TO SIGNED DIMODE ICE (2026-11-22 V1)

Status: fixed and tested for the original `950613-1.c` crash.

## Root cause

KCC's `CAST_FP_IT` path in `ccgen2.c` used scalar `P_FIX` for
native single-precision floating point even when the destination was
signed 71-bit `long long`. The conversion returned a one-word VREG,
but the DImode assignment path expected a two-word register pair,
leading to a segmentation fault in `vrstoreal`.

## Repair

For KA10, KI10 and KS10, the signed SF-to-DImode cast now calls the
existing freestanding `__fixsfdi` runtime helper. The compiler saves
the input, marshals it to AC1, and receives the full AC1/AC2 result.
It removes the ABI sign-copy bit from the low word for KCC's internal
high36:low35 representation. PDP-6 retains its separately validated
software D-format path.

## Targeted verification

* Original upstream `950613-1.c`: compiles on PDP-6, KA10, KI10, KS10.
* Reduced float-to-long-long assignment: compiles on all four CPUs.
* Registered `kcc-float-to-dimode-v1`: passes guest execution on KA10,
  KI10 and KS10 with optimization on and off (six runs).
* Runtime cases include positive, negative and fractional inputs,
  2^24, and positive/negative 2^35 with explicit two-word readback.
* All four testkit audits pass.

## Scope and limitation

The helper comes from the installed PDP-10 freestanding libgcc
archive. A standalone KCC environment must supply that helper when
linking this conversion. This change addresses signed single-float
to DImode; other floating conversion variants were not changed.

The KCC long-long literal comparison for 2^35 in an exploratory
test did not match the correct two-word representation. Explicit
two-word readback of the converted value passed. The literal issue
is separate and remains for investigation.

Only previously failing source tests and targeted regressions were
rerun; no full compiler matrix was executed.
