# Dlink RH18 negative-displacement relocation (2026-11-22 V1)

Status: linker defect fixed; targeted guest regressions passed.

## Failure

The original `tests/todo/generic/index-1.c` computes `a[n - 100000]`.
GCC O0/O1/O2/Os and optimized KCC generated a relocation with a
negative addend of -100000 decimal (`-0303240` octal). Dlink rejected
it as `RH18 relocation overflow`. Unoptimized KCC already passed.

The assembler emitted a LOCAL_RH18 relocation (type 1), including
for the GCC O0 literal reference. A symbol-only relaxation would
therefore not fix this test.

## Repair

`daimos-tools/dlink.c` now permits a negative, representable 18-bit
result for both LOCAL_RH18 and SYMBOL_RH18 when the addend is negative.
The stored right half is masked to 18 bits, as required by PDP-10
effective-address arithmetic modulo 2^18. The accepted negative
result is bounded by -HALF_MASK. Positive overflow, LH18 relocations,
and out-of-range DOBJ addends remain rejected.

## Validation

* Original `index-1.c`: 6/6 PDP-6 profiles PASS, including all five
  previously failing links and their guest execution.
* Registered `dlink-rh18-negative-displacement-v1`: PASS; additionally
  verifies that a positive RH18 relocation at high image base is
  rejected with `RH18 relocation overflow`.
* Previously recorded KCC noopt `920909-1.c` linker failure: currently
  6/6 PDP-6 profiles PASS in a targeted rerun.
* `20020828-2.c`: currently 1 PASS and 5 RUN_FAIL; no LINK_FAIL in
  the six targeted PDP-6 profiles. Runtime correctness remains open.
* Testkit inventory, script ownership, raw p10run ownership, and
  fixture integrity audits: PASS.

No full test matrix was rerun. These results cover only the named
sources and selected PDP-6 profiles.
