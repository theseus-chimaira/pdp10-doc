# PDP-10 GCC LABEL RETENTION AND KCC BYTE-POINTER ADJUSTMENT REPAIRS (V1)

Date: 2026-11-22.
Status: implemented, tested, installed to PDP10_PREFIX, committed and pushed.

## GCC 3.2 PDP-10 BACKEND: REFERENCED LOOP LABELS

The PDP-10 backend's `maybe_remove_label_and_following_code` in
`gcc/config/pdp10/pdp10.c` previously deleted instructions from an
unreachable entry label all the way to the next BARRIER.  An intervening
live CODE_LABEL, such as the self-loop label in an `else { for (;;) {} }`
branch, could therefore be deleted even though its jump reference remained.

The deletion scan now stops at an independently referenced, preserved, or
named label, as well as BARRIER and NULL.  It does not remove the live label
or the loop instructions following it.

Reproduced with the original upstream `961125-1.c`: optimized GCC emitted
`jumpe 6,%L15` without defining `%L15:`; the corrected compiler emits
the referenced label.  A reduced independent regression is registered at
`pdp10-testkit/tests/semantic/regression/gcc_live_inner_loop_label_v1.c`.

Acceptance: 88 selected GCC profile cells, 88 PASS and no compile, link, or
guest runtime failures.  The original upstream testcase also passed direct
KA10 simulator execution at `-Os`.

Source: gcc-pdp10 commit `f72c890`.
Regression: pdp10-testkit commit `317dc2a`.

## KCC: BOUNDED SIGNED BYTE-POINTER ADJUSTMENTS

For positive constant byte-pointer offsets larger than three, `simadjbp`
in `ccout.c` previously emitted one IBP per byte.  The upstream GCC
`HIcmp.c` torture testcase includes a very large constant byte-pointer
offset: KCC completed after roughly 22 seconds but emitted approximately
1.2 GB of assembly (168,890,920 lines).

Large positive offsets now use the shared signed ADJBP simulation path;
only offsets 1-3 inline IBP.  The shared implementation divides the signed
element count by floor(36 / S), where S is the byte-pointer size, adjusts
the pointer's word-address half, and performs fewer than floor(36 / S)
remaining IBPs.  Negative remainders are converted to floor quotient and
nonnegative remainder.  The PDP-6/KA10 helper marshals registers and IDIV
divisors through actual stack memory, because PDP-6 accumulators cannot
be used as ordinary low-core memory operands.  AC13-AC16 are preserved.
The KS10 standalone runtime uses its native ADJBP instruction.

The corrected `HIcmp.c` generated approximately 26 KB of assembly in less
than one second, instead of 1.2 GB in approximately 22 seconds.  This
addresses the two original interrupted PDP-6 and KA10 cells that reproduced
as 20-second timeouts; it does not imply that all other timeouts are false
positives.

Acceptance:
* 40/40 selected KCC cells PASS for `HIcmp.c` and the new large-offset
  C regression.
* Eight existing pointer regression sources: 142 cells, 70 runtime PASS,
  72 compile-only by applicability, zero failures.
* Direct signed ADJBP guest execution on PDP-6, KA10 and KS10 PASS for
  positive/negative 7-, 9-, and 18-bit pointer adjustments and preservation
  of scratch accumulators.
* Testkit inventory, script ownership, raw simulator ownership, and
  registered fixture integrity audits PASS with zero issues.

Source: KCC commit `a46594d`.
Regression and simulator suite: pdp10-testkit commit `6e920ec`.
The regression fixture is installed in the testkit and registered in
`COMPILER_MATRIX.tsv` and `SPECIAL_TESTS.tsv`; it uses the existing
compiler-only runtime prefix isolation policy.

## LIMITS OF ACCEPTANCE

The original broader compiler matrix remains partly incomplete and has
other real link failures, compiler internal errors, and guest timeouts.
These fixes must not reclassify unrelated failing cells as PASS.
In particular, GCC and KCC crashes in other source tests and PDP-10 RH18
linker relocation overflows remain separate work items.
