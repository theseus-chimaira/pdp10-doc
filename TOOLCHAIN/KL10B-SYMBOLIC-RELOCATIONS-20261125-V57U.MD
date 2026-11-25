# GCC native scalar symbolic relocation endpoints (v57u)

GCC 00463d2, testkit e7272e8; based on v57t. No assembler,
linker, emulator, DAIMOS or production installation change.

## Defect and repair

The symbolic ADJBP constant simplifier narrowed a signed target
36-bit element count to host int before calculating the word carry.
At large valid signed 30-bit symbolic displacements, six- and
seven-bit array pointer adjustments could therefore emit the wrong
selector and word addend. The extended path now decodes the full
36-bit signed count and shares the canonical native coordinate helper
with the absolute scalar constant folder. It uses a signed host
wide calculation for the slot, quotient and remainder, with floor
correction for negative remainders. The helper declines a symbolic
fold if the resulting addend exceeds the signed 30-bit RTL field.
The local ADJBP simplifier is unchanged. No target runtime code or
new object relocation kind is introduced.

## Acceptance

The dedicated tools/test-gcc-global-symbolic-v57u.py runner generates
positive and negative displacement initializers for external scalar
arrays of widths 6/7/8/9/16/18/32/36. The native slot oracle checks
all starting and ending slots with six word displacements per group,
including section carries/borrows and the endpoints -2^29 and
2^29-1. Large synthetic offsets are linker/ABI probes, not portable
C array accesses. The exact-width types in this runner use native
storage forms, not continuous packed fields.

Each group contains 618 pointer words. At O0/O1/O2/Os, all 4944
DOBJ2 selector/addend/target relocation checks pass, and all 4944
final linked P10X1 pointer words equal the independent oracle.
Separate low and high DATA placements keep the relocated addresses
within the unsigned 30-bit range. No high synthetic pointer is
executed or dereferenced. The v57t baseline has 61 wrong relocations
in each O0 displacement group (122 total); the --baseline option
records both failures. No assertion or diagnostic is bypassed in
the accepted candidate runs.

The earlier static-pointer suite passes four object profiles. All
73400 native absolute constant checks pass again. The earlier native
boundary (8), exact16 (40), and exact32 (16) runtime observations
pass on v57u, totaling 64 SIMH/KLH10 runs. The 56 extended assembly
and 112 local assembly guards against v57t are byte-identical.
The inventory passes with 2400 unchanged compiler matrix sources,
446 special entries and zero unowned test sources.

Evidence: pdp10-testkit/manifests/PDP10-GCC-GLOBAL-SYMBOLIC-20261125-v57u.tsv
(eight accepted object/link profiles and two baseline controls).
Staged compiler: TMPDIR/pdp10-gcc-symbolic-v57u/gcc. cc1 SHA256:
39d7464f4529cd130f02e3e3c60f6c765c71f7051ed7deccb7921cc4b7351167.
Use staged DAS v56y and dlink v56m. No installation performed.

## Remaining limitations

The current compiler symbolic pointer encoding has a signed 30-bit
word-addend field. The linker can accept larger signed displacement
magnitudes, but this compiler stage does not implement a wider
symbolic representation or establish that every out-of-range source
expression is diagnosed rather than folded incorrectly. Those cases
remain open, including packed exact-width symbolic endpoints and
combined symbolic expressions. The test does not establish arbitrary
cross-section object placement, additional aggregate layouts, mixed
cast semantics, or full extended runtime support. Continue with
these correctness gates before measured resident-size optimization,
multi-TEXT placement and KCC/GCC ABI acceptance. Preserve the
historical compiler matrix.
