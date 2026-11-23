# GCC/KCC floating and natural-byte aggregate ABI, KCC GAS literals (V1843)

Date: 2026-11-23. Source and testkit coverage only; no DAIMOS kernel or
production libc code changes.

## The original cross-compiler ABI question

`daimos-testkit/tests/validation/daimos-kcc-gcc-float-struct-v1840.sh`
compiles separate GCC and KCC translation units and executes both mixed
caller/callee directions under PDP-6, KA10, KI10 and KS10 at GCC -O0,
-O1, -O2 and -Os. A KCC callee is built without optimization at O0
and with -O for the other variants.

The fixture verifies float- and double-containing structures occupying
2, 3, 4, 5 and 8 native 36-bit words. It exercises direct function
calls, indirect calls through function pointers, mixed float/double and
integer arguments, and hidden result-buffer returns for objects larger
than four words. Exact binary-fraction inputs make value comparisons
unambiguous for the tested PDP-10 formats.

The repaired toolchain passes all 32 directions/CPU/optimization cells.
The test covers those layouts and arguments, not every floating-point
structure, union, or ABI extension.

`daimos-testkit/tests/validation/daimos-kcc-gcc-natural-byte-aggregate-v1842.sh`
independently checks ordinary C89 structures with a five-character
9-bit array, an intervening 36-bit word and a trailing three-character
array. The same 32-cell matrix verifies cross-compiler sizeof, direct
and function-pointer structure returns, by-value arguments, pointer
mutation and preservation of adjacent byte fields. All 32 cells pass.
This does not imply support for GNU packed/aligned attributes or
cross-word bit-field layouts.

## KI10/KS10 compiler-assembler failure

The first floating-point matrix passed all PDP-6 and KA10 cells,
then failed when assembling a KCC-built KI10 caller. The original KCC
GAS output contained two-word double literals split across physical
lines, for example:

    dmove 6,[0202400000000
             0]

The line-oriented DAS parser already supports a two-word literal
when the words are comma-separated on one source line, but not this
historical multiline form. DMOVE and DMOVEM instructions themselves
assemble correctly; the issue was the GAS dialect's emitted operand.

An initial host DAS reader change assembled these source lines and
passed its fast test suite, plus the KI10/KS10 literal regression.
Its effect on the 64K-word native DAS is unacceptable: the phase-1
image grew from octal 035216 to 035743, while the existing resident
budget is octal 034000. A separately rebuilt **unmodified DAS**
baseline also exceeds that budget (035216 with current KCC,
035205 with the pre-v1830 KCC). The baseline size issue therefore
predates this exercise and was not caused by the AC16 indirect-call
repair. All experimental DAS code and manual edits were reverted;
the original DAS source and installed binary were restored.

## RAM-neutral KCC production correction

The actual production correction is in KCC `ccout.c`, not DAS.
For GAS-output *instruction literals* in PTA_DCONST, KCC emits the
two mapped or unmapped words on one line, separated by a comma:

    dmove 6,[0202400000000,0]

`outmpdbl()` distinguishes a code literal from separately emitted
static data. In GAS mode only, a code literal gets a comma, while
static double data remains two complete assembler data words.
Non-GAS historical output retains its original formatting.
No resident DAS code, ABI adapter, runtime helper or permanent
data allocation is introduced.

`daimos-testkit/tests/validation/daimos-kcc-gas-double-literal-v1843.sh`
compiles native KCC GAS code for KI10 and KS10, with and without
optimization. All four cases assemble using the **original DAS**
and execute correctly on the target simulators, including initialized
double data. The script also checks that KCC emits comma-separated
DMOVE literals without a split line. The floating-aggregate matrix
passes 32/32 with the repaired KCC and original DAS.

## Installed toolchain and exclusions

The installed KCC compiler at `$PDP10_PREFIX/bin/kcc` has SHA256:

    d40828f32210ab6128614b10e35780e6588675d7524f612d4af28b43e1aaa6ba

The previous compiler was preserved under
`~/backups/kcc-gas-double-literal-v1843-20261123/`.
The restored original DAS has SHA256:

    43fd76e4489e29c0065ad2d7155f59d81ce6f66662ea21b7deb3c02b109d4127

The verified subset does not include compiler-private va_list objects,
floating-point variadic calls, GNU-specific packed attributes, or
cross-word bit fields. The independent native DAS phase-image budget
regression remains open, and was neither hidden nor expanded by the
accepted KCC-side correction.
