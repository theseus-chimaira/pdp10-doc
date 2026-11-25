# GCC native global pointer boundaries and switch dispatch (v57s)

GCC f94a349, testkit 8077ef0; based on v57r. No assembler,
linker, emulator, DAIMOS or production installation change.

## Defects and repair

The native-pointer fixture initially failed before its first arithmetic
row completed. An extended casesi dispatch used a local indexed JRST
through a table of 18-bit indirect words. Such an indirect word selects
section zero, not the current nonzero section. Extended GAS/DAS output
now emits GIW entries with ADDR30 relocations. Local output remains
unchanged. The indexed table and dispatch remain in the same code
section; this is not arbitrary cross-section table placement.

Native byte-pointer materialized boolean comparisons previously used
raw encoded words. Branch ordering rotated the tag behind the address
but did not bias the sign for the entire 30-bit coordinate range. Both
paths now share the address-first rotate-and-bias sequence already used
for exact-width pointers. Word pointers retain their normal address
comparison. An isolated six-bit comparison of the final slot of
5,,777777 and first slot of 6,,000000 reproduced a false boolean result
on both SIMH and KLH10 with v57r, while its branch result was correct.
Both results are correct on v57s. A diagnostic attempt to disable jump
tables with -fno-jump-tables was rejected by this older compiler;
no such option or workaround is used in acceptance tests.

## Acceptance

The dedicated tools/test-gcc-global-native-boundary-v57s.py runner
uses independent host slot coordinates and raw memory oracles. It
checks all native 6/7/8/9/18/36-bit representations, every slot, near
page and section boundaries, both sides of address bit 29, and near
the upper 30-bit coordinate limit. Each of 1835 arithmetic rows checks
addition, inverse subtraction, both pointer differences, materialized
and branch ordering, and compound adjustments. Integer-cast high
addresses are arithmetic-only ABI probes, not dereferenced C objects.

Each of 88 physical memory rows checks signed and unsigned loads,
stores, raw preserved bits and a preceding guard word. The final slots
also check pre/post increments and decrements across a page or section
boundary and preserve the next word. Physical mappings cover the page
boundary 5,,377777, section boundaries 5,,777777 and 036,,777777,
and a final-word guard in section 7 with section 010 unmapped.
All eight O0/O1/O2/Os SIMH/KLH10 runs pass: 14680 arithmetic rows
and 704 memory rows. No new CPU or emulator profile.

Earlier native narrow (8), record/switch (16), extended link (10),
void/mixed conversion (16) and exact-width full-range (8) runtime
observations pass on v57s, totaling 58. Local DXR link comparison
and link rejection controls pass. All 112 local assembly guards are
byte-identical to v57r. Inventory: 2400 unchanged matrix sources,
444 special tests, zero unowned sources.

| Optimization | TEXT words | DATA words | BSS words |
| --- | ---: | ---: | ---: |
| O0 | 2283 | 8574 | 5 |
| O1 | 1606 | 8574 | 5 |
| O2 | 1539 | 8574 | 5 |
| Os | 1521 | 8574 | 5 |

DATA is dominated by the test oracle. These are linked image sizes,
not production RAM or timings. No new runtime helper or correction
array is introduced. Evidence:
pdp10-testkit/manifests/PDP10-GCC-GLOBAL-NATIVE-BOUNDARY-20261125-v57s.tsv
(8 passes and one baseline wrong-section failure). Staged compiler:
TMPDIR/pdp10-gcc-native-boundary-v57s/gcc. cc1 SHA256:
b4ffd90bae11f2d78de30dfdb772c2674917fd7620d8c86451504c332ee2c3f8.
Use DAS v56y and linker v56m as before.

## Remaining limitations

Native absolute constant-folded pointer operations, symbolic relocation
edge cases, aggregate strides and remaining cast semantics are open.
The physical tests do not establish actual cross-section object placement
by the linker or general multiple TEXT section layout. The previously
accepted exact-width constant tests were not rerun in this stage.
Extended runtime/libgcc, size optimization, KCC and mixed-compiler ABI
remain open. Preserve the historical compiler matrix until the extended
work is finished.
