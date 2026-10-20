# KCC PDP-6 Signed DImode and D-format Corrections (2026-10-19)

## Signed 71-bit automatic variables

The internal KCC signed 71-bit integer representation is a two-register
high36:low35 pair. In memory and in the external calling convention the
sign bit of the high word is also copied into bit 35 of the low word.

The direct fixed-automatic load path in `ccgen2.c` previously used
`DMOVE` without clearing that extra low-word sign bit. This caused
negative values reloaded from memory to compare unequal to numerically
identical register values. KCC now emits `TLZ` of the duplicated sign
bit following this direct load for signed DImode objects. The testkit's
`kcc-dimode-local-load-v1.c` checks zero, positive, negative, and
multiple-word values, passing under optimized and `-n` KCC on SIMH PDP-6.

## PDP-6 D-format doubleword negation

Unlike later machines, the PDP-6 has no native `DMOVN` instruction.
KCC emulated D-format negation with two independent `MOVNS` operations,
which do not implement a 71-bit negate correctly. The continuation word
contains 35 fraction bits and a replicated sign bit.

The PDP-6 expansion in `ccout.c` now masks the low 35 fraction bits
after negation, subtracts the appropriate borrow from the high word,
and restores the continuation-word sign bit. It uses no temporary AC
and no stack or writable data. The testkit `kcc-dformat-negative-v1.c`
checks both signs and a double-negation round trip with both compiler
optimization profiles on SIMH PDP-6.

## Completed signed 71-bit/software D-format conversion (2026-10-20)

The original `930622-2.c` ICE is now fixed in `-x=pdp6` KCC. Its
signed 71-bit integer is already a high36:low35 register pair and must
not be widened as though it were a scalar. KCC calls a new independently
loadable, integer-only assembler conversion member instead:

- `__kcc_floatdidf`: high36:low35 signed DImode to two-word software
  D-format, with nearest/even rounding and normalized exponent
- `__kcc_fixdfdi`: software D-format to signed DImode, truncating
  fractional values toward zero

Both routines live in `kcc-libc-dimode-float-pdp6-v1.s`, require no
internal stack or writable data, and do not call libgcc. The generated
KCC call uses two temporary stack words for the AC1/AC2 pair. A compiler
and library build has been installed under `$PDP10_PREFIX`; the KCC/GCC
distribution archives are byte-identical.

The unchanged upstream test executes successfully on SIMH PDP-6 in both
KCC optimization modes. A separate regression checks 17 signed-integer
bit patterns (including signed extremes and nearest/even rounding) and
ten fractional D-format truncation cases. Both modes pass. GCC PDP-6
also executes the original source at `-O0/-O1/-O2/-Os`.

**The PDP-6 native floating-point representation is different from the
portable software D-format and from later PDP-10 CPU capabilities.**
Consequently only the PDP-6-native test-matrix cells are activated;
KA10/KI10/KS10/KL10 and other floating-point modes have neither an
implicit compatibility guarantee nor a passing result from these tests.
AAP PDP-6 execution remains unverified until a reliable post-removal
replacement for its old diagnostic `cpustatus` test oracle is established.
