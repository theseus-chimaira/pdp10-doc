# GCC native global scalar constant folding (v57t)

GCC 71af84b, testkit bc31c69; based on v57s. No assembler,
linker, emulator, DAIMOS or production installation change.

## Defects and repair

The scalar constant-folding hooks introduced for exact16/32 in
v57q/v57r now also cover canonical native 6/7/8/9/18/36-bit
integral scalar pointer types. Generic C-unit arithmetic previously
operated on the encoded pointer word instead of its slot coordinate.
Absolute pointer differences could be rejected in static initializers,
and generic relational folding compared tags before word addresses.

The native selector validator accepts the canonical slots for the
requested width, or tag zero for word pointers. Before-first selector
encodings are not treated as canonical object addresses. Adjustment
adds the element count to the slot ordinal, then divides by the
number of slots per word with floor correction for negative remainders.
Difference multiplies the word-coordinate difference by the number of
slots and adds the slot difference. Comparison retains the existing
address-first ordering key. The scalar hooks are renamed to reflect
both native and exact-width support. Exact-width host coordinate
arithmetic is unchanged. All hooks are extended-only and integral
scalar-only. There is no new generated runtime routine or table.

## Acceptance

The dedicated tools/test-gcc-global-native-constant-v57t.py runner
uses the independent native slot-coordinate oracle from v57s. It
creates separate translation units for adjustment, difference and
comparison, covering 1835 rows across all six widths. Each row adds
forward/inverse adjustments, forward/reverse differences and all six
relations. Actual absolute object words must equal the oracle and
have no relocation. At each of O0/O1/O2/Os the candidate passes 3670
adjustments, 3670 differences and 11010 comparisons, for a total of
73400 words. The v57s O0 baseline has 3088 wrong adjustment words,
rejects the difference translation unit, and has 2432 wrong comparison
words. The runner's --baseline option records all three failures.

The exact-width constant adjustment and relation suites pass all
171440 previously accepted word checks again on v57t. The native
boundary suite passes 8/8 SIMH/KLH10 runtime profiles (1835 arithmetic
and 88 physical memory rows per run). All 56 extended assembly guards
and 112 local assembly guards against v57s are byte-identical. The
historical compiler matrix remains at 2400 sources; the inventory has
445 special entries and zero unowned sources.

Evidence: pdp10-testkit/manifests/PDP10-GCC-GLOBAL-NATIVE-CONSTANT-20261125-v57t.tsv
(12 passing object profiles and three baseline controls). Staged GCC:
TMPDIR/pdp10-gcc-native-constant-v57t/gcc. cc1 SHA256:
bb07138cf8e67f9f8a160b07f2e8b3055f788a8ec0c0e72cfab3e12bb1dd1b67.
Use staged DAS v56y and linker v56m. No installation performed.

## Remaining limitations

Integer-cast absolute pointers are target ABI probes, not portable C
objects. These tests establish canonical scalar constant operations,
not symbolic relocation edge cases, noncanonical pointer encodings,
mixed storage-form comparisons, arbitrary cast semantics or aggregate
strides. Symbolic relocation folding and additional aggregate layouts
remain open, followed by resident-size optimization, extended runtime,
multiple TEXT sections and KCC/mixed-compiler ABI acceptance. Preserve
the historical compiler matrix until the extended work is finished.
