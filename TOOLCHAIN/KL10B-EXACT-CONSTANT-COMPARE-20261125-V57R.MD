# GCC absolute exact-width pointer differences and comparisons (v57r)

GCC 683be66, testkit 4531f14; based on v57q. No assembler, linker,
emulator, DAIMOS or production installation change.

## Defects and repair

The previous C front end rejected absolute integer-cast exact-width
pointer differences in static initializers as not computable at load
time. The generic constant comparison path compared encoded words
rather than address-first coordinates, so ordering could disagree
with the already validated dynamic lowering.

The new difference hook folds scalar exact16/32 operands before the
common-base optimization and C-unit division. Native exact32 slots
use word difference, native exact16 slots use twice the word
difference plus the slot difference, and packed operands use the
linear bit difference divided by declared precision. Packed operands
must have an integral element distance; distinct storage forms are
not treated as interchangeable same-array coordinates. The result
is a signed ptrdiff_type constant. The comparison hook runs before
the generic folder strips pointer type information, and compares
unsigned word-coordinate-first keys with the tag in the low six
bits. This matches the dynamic rotate-and-bias path; equality still
compares encoded words. Both hooks are extended-only and restrict
operands to matching scalar integral precision 16 or 32. They add
no target runtime code or table.

## Acceptance

`tools/test-gcc-global-exact-compare-v57r.py` uses the v57o independent
host oracle. Separate generated translation units contain 8572
forward/reverse signed differences and 25716 comparisons (all six
relations for each of 4286 rows). Each assembled absolute word is
checked against the expected value and must have no relocation.
All four optimization profiles pass: 34288 differences and 102864
comparisons, 137152 object-word checks in total. The v57q O0
baseline rejects the difference translation unit and produces 8064
wrong comparison words out of 25716. These are target ABI probes,
not portable C arithmetic on actual objects at high addresses.

The earlier constant-adjustment test passes all 34288 words again.
The dynamic typed full-range suite passes all eight O0/O1/O2/Os
SIMH/KLH10 observations on v57r. All 48 extended assembly guards
covering earlier exact-width, conversion, boundary and static
profiles are byte-identical to v57q, as are 128 local assembly
guards. No new local runtime matrix runs. Inventory: 2400 matrix
sources, 443 special entries, zero unowned sources.

Evidence: pdp10-testkit/manifests/PDP10-GCC-GLOBAL-EXACT-COMPARE-20261125-v57r.tsv.
Staged compiler: TMPDIR/pdp10-gcc-exact-compare-v57r/gcc.
cc1 SHA256:
fe145ddf2db412361eae8b319f7dd2aa3419cddb3b412b33c0e593a8cc73a0d2.
Use staged DAS v56y and linker v56m. No installation performed.

## Remaining limitations

The tested constant relations are same-array scalar exact16/32
coordinates. This does not validate arbitrary void or mixed-type
constant comparisons, symbolic relocation edge cases, aggregate
strides, or native hardware byte/halfword/word boundary operations.
Those operations, code-size optimization, extended runtime,
multi-TEXT placement and KCC remain open. Preserve the compiler
matrix until extended work is finished.
