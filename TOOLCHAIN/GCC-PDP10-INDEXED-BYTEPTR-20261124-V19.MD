# GCC indexed constant byte-pointer correction v19

Status: bounded compiler fix, checked on 2026-11-24.

## Problem

An indexed symbolic PDP-10 byte pointer is a 36-bit value: its left
half contains the P/S byte selector, and the right half contains the
word address. The GCC `*addsi3` backend had used `MOVEI` to combine a
constant `POINT` value with a register index. MOVEI computes only the
18-bit effective address, discarding the selector. This corrupted
9-bit pointer differences in the optimized compiler.

## Correct lowering

The backend now decomposes an actual symbolic byte-pointer constant
into selector and address components. For the local indexed-pointer
case it emits `MOVEI` of symbol+offset(index), followed by `HRLI` of
the original selector. No temporary AC is required. The pair is
safe when the destination and index share an AC, and the selector
cannot change through low-address arithmetic carry. Normal indexed
word-address additions continue to use a single MOVEI.

The machine-description attribute reports two words for exactly
this byte-pointer case, preserving correct instruction length and
avoiding global pessimization of word pointer expressions.
Implementation: `gcc-pdp10/gcc/config/pdp10/pdp10.c`, `pdp10.md`
and `pdp10-protos.h`.

## Verification and open problems

The reduced C89 char9 pointer-difference source passed 16 CPU and
optimization combinations. A new registered testkit regression for
indexed 9-bit and signed 6-bit pointers passed **36/36** GCC cells
under strict exit policy in an independent output directory.

The larger GCC O1 byte-pointer test still has **one runtime failure
and eight timeouts**. Eight former immediate failures at octal `0250`
now advance to later code and expire the simulator instruction limit;
they must not be counted as passed. The wider full matrix still has
independent known defects.

Detailed checkpoint evidence:
`pdp10-testkit/history/PDP10-INDEXED-BYTEPTR-REPAIR-20261124-v19.md`.
