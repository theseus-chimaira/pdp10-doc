# GCC extended pointer conversions (v57n)

GCC a8b2545, testkit b7b8150; based on v57i. The completed stage
covers tested extended object-pointer conversions and typed packed
exact32 dynamic references. It is not a frozen GCC/KCC ABI.

## Carrier and conversion rules

Pointers retain one 36-bit word and the low 30-bit global address.
The ordinary C void/character representation remains canonical
9-bit OWGBP: native word addresses become selector 070, and native
18-bit selectors 075/076 become 070/072. Native exact32 word slots
also become selector 070. Explicit 6/7/8-bit pointers and software
exact16/packed exact32 tags retain their representation when passed
through void*, because a canonical 9-bit pointer cannot retain all
of their positions and storage forms. Null remains zero. The local
address model and the explicit no-convert-voidp option are unchanged.

The extended conversion path recognizes exact-width and void types
before the legacy local P/S and architectural-selector conversion
paths. Static conversions preserve signed ADDR30 relocation addends;
constant conversions use the same tag mapping as runtime conversions.
Runtime conversions preserve the low address bits and map the source
tag to the destination representation. Word targets require only
masking; ordinary native to void conversions use simple instructions.
A dynamic conversion needing a tag translation uses one shared
64-entry six-bit table per destination representation, emitted only
when referenced. Each table occupies eleven 36-bit words; lookup uses
ADJBP and LDB through an actual global 6-bit pointer. No new library
routine or relocation kind is required. Tables are emitted in data
with explicit octal constants.

The tested mixed exact16/exact32 casts retain packed physical tags.
For native word-aligned objects a cast to another exact width selects
the native destination slot. Native architectural byte destinations
select a canonical slot from the physical starting position. This
mapping is not a promise of lossless misaligned casts: an arbitrary
packed bit position may not exist in the destination architectural
representation, and a native slot may lose its original stride
provenance when cast to another type. Same-type void round trips do
not lose that information. Further validation is required before
freezing mixed-width alignment and representation rules.

Void comparisons normalize the physical coordinate without changing
the stored pointer. For ordering, the address comes before the bit
position. Both direct branches and materialized boolean comparisons
use this mapping. The lowering protects queued operands and flushes
the side-effect queue before the comparison branch.

## Dynamic packed exact32 access

The conversion fixture exposed a separate preexisting defect: a
variable-index direct store into a packed exact32 member array used
a generic word-relative field-store path, rather than the packed
physical bit stride. The exact-width reference-address helper now
supports the tested packed exact32 forms as well as exact16. Both
reads and assignments use the typed physical access path. Existing
native exact32 and local address paths are retained. Constant-position
specialization and sharing the arithmetic/access sequences remain
resident-size optimization work.

## Acceptance

The v57i compiler reproducibly reports an internal compiler error in
bytesize_for_ps for the static packed void-pointer initializer at all
four optimization levels. The v57n candidate compiles, assembles,
links and executes the comprehensive new fixture and the unchanged
historical pdp10-void-pointer-roundtrip-locator-v1.c at O0/O1/O2/Os on
both SIMH and KLH10: 16/16 runtime observations pass. The new fixture
checks static and dynamic exact16/32 and native 9/18/36-bit round-trip
identity and values, native explicit 6/7/8-bit pointer identity,
null, indirect calls, physical equality, mixed exact-width casts,
ordinary C void raw representation, post-increment side effects,
and direct dynamic packed exact32 reads/stores. The historical test
checks independent scalar values and identity/value masks.

The v57l predecessor passed 128 earlier extended runtime observations
(16 exact32, 40 exact16, 16 record, 8 narrow, 48 wide/direct-packed)
and four static-pointer object profiles. The final v57n build has
68 byte-identical extended assembly guards against v57l covering
those profiles, and 112 local assembly comparisons against v57i.
These are dedicated guards, not changes to the compiler matrix.
Inventory: 2400 matrix sources, 439 special entries, zero unowned
sources. Production tools and DAIMOS remain unchanged.

Final linked TEXT/DATA/BSS counts, in 36-bit words:

| Fixture | O0 TEXT | O1 TEXT | O2 TEXT | Os TEXT | DATA | BSS |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| Comprehensive | 2825 | 1779 | 1830 | 1735 | 96 | 28 |
| Historical | 148 | 129 | 122 | 121 | 23 | 9 |

The comprehensive case references eight conversion tables (88 DATA
words); the historical case references two (22 DATA words). These
are linked image counts, not peak RAM or speed measurements. Later
optimization should specialize known-source conversions, remove
unneeded dynamic tag work, and share exact-width operations where
measured total resident words are reduced.

Evidence: pdp10-testkit/manifests/PDP10-GCC-GLOBAL-VOID-CAST-20261125-v57n.tsv.
Runner: tools/test-gcc-global-void-cast-v57j.py. Staged compiler:
TMPDIR/pdp10-gcc-global-casts-v57n/gcc; cc1 SHA256
1693fecb58f48085202922f70375e33fe7e648f2110aab2a422409ddba94255d.
Use DAS v56y and linker v56m as before.

## Remaining work

Compiler-level full-range pointer arithmetic and physical accesses
across section boundaries; further aligned/misaligned cast semantics,
GNU void-pointer arithmetic on extension tags, additional aggregate
layouts, and mixed compiler ABI validation remain open. No claim of
full arbitrary cross-width reversibility, full C extension coverage,
or KCC compatibility. The GAS/DAS target is tested; other assembler
output modes are not. Extended libgcc and freestanding runtime,
multi-TEXT placement, and KCC remain pending. The historical matrix
remains out of scope until the extended work is complete.
