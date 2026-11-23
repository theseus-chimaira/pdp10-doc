# GCC/KCC aggregate-return ABI and GCC BLT frame correction (V1836)

Date: 2026-11-23. Status: PASS in 32 mixed-compiler execution cells;
16 additional GCC word-copy cells; three existing toolchain suites.

## Original ABI question

`daimos-testkit/tests/validation/daimos-kcc-gcc-struct-return-v1836.sh`
compiles separate callers and callees with GCC and KCC, links opposite
compiler combinations, and executes real target code on PDP-6, KA10,
KI10 and KS10. The caller exercises both direct and indirect calls.
It verifies independently returned objects of 1, 2, 3, 4, 5 and 8
whole 36-bit words. The 1-4 word cases use return ACs. Larger returns
require the caller-supplied hidden result buffer. All returned word
values are checked, not merely linker success.

At `-O0`, both compiler directions passed on PDP-6. With an `-O2`
GCC caller, the four-word direct-return check failed with decimal ID
8 (the simulator displays octal `000000000010`). The same check
failed with a GCC callee, while KCC caller -> GCC callee and KCC ->
KCC both passed at the same optimization setting. This isolated a GCC
caller error, not an incompatible KCC return ABI.

## Instruction-level diagnosis

GCC stored AC1-AC4 into four consecutive frame-relative words. Its
BLT setup then incorrectly used a source stack displacement of octal
`-037` instead of `-7`. The GCC RTL dump shows source
`virtual-stack-vars+32`, destination `virtual-stack-vars+3` and a
16-C-unit (four-word) copy. Source offset 32 was *already* a word
offset of the just-materialized return temporary.

`gcc/config/pdp10/pdp10.c:pdp10_normalize_block_word_address()` had
inferred C storage units whenever a positive offset was an exact
multiple of the block's C-unit size. That inference is invalid for
frame slots: FP+32 can be a genuine native word address. The erroneous
division by `UNITS_PER_WORD` changed the source to FP+8.

The compiler now preserves word-aligned BLKmode frame-relative
addresses as native word addresses. Existing normalization for
symbolic/global C-unit array offsets remains active. This fixes the
actual address-representation error, without a BLT instruction
workaround, a caller adapter, or a separate memory-copy routine.
GCC production commit: `395ec75` on `master`.

## Acceptance and regression protection

The final mixed KCC/GCC matrix passed 32/32 cells: four CPUs, four GCC
levels (`-O0`, `-O1`, `-O2`, `-Os`), both calling directions. KCC was
compiled without optimization for `-O0` and with `-O` for the other
profiles. Every cell checked direct and function-pointer returns
of all six structure sizes.

The independent `daimos-gcc-blt-word-frame-v1837` regression passed
16/16 GCC cells. It checks whole-word copies from global arrays to
global arrays, globals to stack arrays, stack to stack, and stack to
globals, plus independence after mutating the original source.

Existing installed-toolchain regressions also passed: the aggregate
return ABI compile check, 32/32 packed-record pointer cells, and
PDP-6 byte-BLT copy/fill. New suites were tested first against the
uninstalled rebuilt `cc1`, then both were repeated successfully
against the installed compiler. The GCC-to-KCC byte/word varargs
matrix and the separate wide varargs suite also passed with the
installed toolchain. None of these tests claims an interchangeably
laid-out compiler-private `va_list`.

Installed compiler SHA256:

    fd42b3acb75f07b9dc2aa8e3d34b05db60309d711bfaf7fa59e4e1467db24114

The prior installed `cc1` was saved under
`~/backups/gcc-cc1-blt-frame-v1836-20261123/` and had SHA256
`28b881b3d59ed4d335cf92865eff223079031229244cf8e69ba6913a1cc2b4cf`.

## Boundaries

These regressions use word-only structures. They do not close older
mixed character/float aggregate-return failures, packed-structure
alignment questions, or compiler-private `va_list` layout. No DAIMOS
kernel or production libc source was changed.
