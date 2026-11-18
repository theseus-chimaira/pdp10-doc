# DAIMOS libc formatted output, v33

## Representation

The PDP-10 C ABI uses 9-bit characters and 36-bit words.  Signed
long long carries **71 value bits** in two words, including a
duplicated sign bit.  This is distinct from double: the two-word
software D-format occupies **72 bits**, with a sign, an 8-bit
biased exponent, and 62 fraction bits (27 in the first word and
35 in the second).  The other bit of the second word copies sign.

The v33 assembler converter consumes both D-format words and
produces an unsigned 36-bit magnitude plus a separate sign.
It does not reinterpret the floating operand as a 71-bit integer.

The KCC source distinguishes PDP-6 and KA10 floating-operation
modes.  Static code generation of 5.0, -2.5, 0.5, 3.5,
33554432.0, -0.25 and 8589934592.0 produces identical two-word
constants with KCC targets pdp6, ka10, ki10 and ks10.
This establishes constant encoding agreement, not full runtime
operation equivalence.  KA10+ dynamic-float guest validation
remains open.

## Implementation

DAIMOS/userland/libc/ contains:

- sprintf-v33.s: KCC variadic assembler entry, preserving AC1/AC2
  named parameters and the stack-resident unnamed parameters.
- dformat-f0-v33.s: integer-only conversion and nearest, ties-to-even
  rounding of portable two-word doubles for zero decimal places.
- printf.c: shared printf/vprintf and sprintf/vsprintf formatter,
  with character-device and caller-buffer sinks.
- Makefile: archive rules for these objects.

The entry and converter use only PDP-6 integer instructions; neither
requires later processor opcodes or hardware floating arithmetic.
The implementation handles ordinary integer/string/character
conversions and width flags.  Floating formatting is currently
limited to explicit %.0f and %.0F for magnitudes fitting a 36-bit
unsigned integer.  Other floating precisions return -1 instead
of silently supplying incorrect text.  A full decimal formatter,
large magnitudes, and IEEE-style exceptional values remain future
work, not claimed functionality.

The earlier hosted-libc work reuses real malloc/calloc, tightens
allocation overflow checking, provides abort via the DAIMOS exit
syscall, adds isspace, and implements console printf/vprintf.
No test-only fake formatter or allocator is involved.

## Tests

The test code resides only in daimos-testkit:

- tests/validation/daimos-sprintf-float-v33.sh
- tests/validation/daimos-sprintf-float-v33.c

Both real DAIMOS/PDP-6 guest probes passed:

1. Unmodified upstream pdp10-testkit test 930513-1.c calls
   sprintf through a function pointer with %.0f and 5.0,
   checking that the two-byte result is 5 followed by NUL.
2. The v33 fixture checks zero precision, negative numbers,
   halfway ties, fixed width and sign, function pointers,
   return lengths, and larger values through 34359738367.

The expanded guest run was stored in versioned local directory
$TMPDIR/daimos-sprintf-float-v33-29055.

Do not claim native KA10+ floating-point correctness from these
PDP-6 executions.  If a target selects a distinct representation,
use a distinct conversion helper rather than duplicating the
entire formatting engine.
