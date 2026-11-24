# PDP-10 GCC 36-Bit Unsigned Division/Modulo, v17

Status: backend defect fixed; installed GCC and all specifically selected
regression profiles verified on 2026-11-24.

## Machine-level cause

The PDP-10 `DIV` instruction consumes a 70-bit dividend formed from
two ACs; the second AC contributes 35 payload bits. The old GCC
`udivsi3`/`umodsi3` lowering put a full 36-bit unsigned dividend in
that low AC while setting the high AC to zero. Bit 35 was silently
dropped, so high-bit unsigned division and remainder could be wrong
on PDP-6, KA10, KI10 and KS10.

## Correct representation

Before `DIV`, copy numerator into both result ACs, logically shift
the high AC right by 35 (octal shift count `-43`), and clear the sign
bit of the low AC. The existing pair then contains a positive 70-bit
representation of the entire unsigned 36-bit value. The backend
handles high-bit divisors via its existing unsigned comparison path;
the relative jumps there account for the two added instructions.

No extra temporary register or libgcc helper is required for the
positive path. The patch is in
`gcc-pdp10/gcc/config/pdp10/pdp10.md`, patterns
`pdp10_udivmodsi4_positive` and `pdp10_udivmodsi4_general`.

## Reproducible verification

The native PDP-6 reducer failed at octal case `0201`, the high-bit
division `0400000000000 / 2`. A scratch assembly correction established
the root cause before the machine description was changed.

After rebuilding and installing the compiler at `$HOME/cross`, the
official `pdp10-testkit` runner re-executed 108 previously failed
cells over three existing registered sources: unsigned quotient,
unsigned remainder and their combined operation. All **108 PASS**.

To cover immediate/constant-divisor lowering independently, testkit
added a C89 regression with a volatile numerator, registered across
36 GCC profiles. All **36 PASS**, including high-bit inputs and
non-power-of-two positive divisors.

The compiler runtime repair is complete for these tested cases.
The testkit's broader sweep remains unfinished; its latest local
inventory has 52,636 recorded cells and 2,927 other runtime failures.
Do not interpret this as a general GCC runtime pass.

Detailed source/checkpoint evidence:
`pdp10-testkit/history/PDP10-GCC-UNSIGNED-DIVMOD-RUNTIME-20261124-v17.md`.
