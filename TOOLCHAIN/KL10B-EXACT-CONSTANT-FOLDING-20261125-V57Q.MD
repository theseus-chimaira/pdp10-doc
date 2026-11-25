# GCC absolute exact-width constant folding (v57q)

GCC f178a00, testkit f76bf93. Based on the v57n compiler and v57o
independent full-range coordinate oracle. No DAIMOS, assembler,
linker, emulator or production installation changes.

## Defect and repair

The C front end previously folded addition or subtraction of an
integer-cast absolute exact-width pointer and a constant element
count using ordinary scaled C-unit integer arithmetic. The encoded
pointer word is not a linear C-unit coordinate: this can change the
word coordinate incorrectly or destroy the software tag. The v57n
baseline produces an incorrect first word in the generated test.

The new scalar exact16/32 front-end hook operates before generic
C-unit scaling. The host coordinate calculation is shared with the
existing RTL constant-add paths. It distinguishes native slots from
continuous packed positions, handles signed counts and normalizes
negative remainders. Symbolic RTL operands retain their signed
address-addend interpretation. The final v57q hook explicitly
requires an integral scalar of precision 16 or 32, rather than
selecting pointer-to-array operations by their element byte size.
Dynamic pointer instruction sequences are unchanged by this repair.
No additional target runtime routine or table is introduced.

## Verification

The dedicated test `tools/test-gcc-global-exact-constant-v57p.py`
generates C static initializers from the 4286 independently calculated
v57o arithmetic rows. Each row contributes a forward addition and
an inverse subtraction of an element count. The assembler object
reader checks each actual 36-bit word and verifies that the absolute
pointer words carry no relocation. At O0/O1/O2/Os, all 8572 entries
per profile pass on the final v57q compiler: 34288 checked words.
The v57n baseline fails at its first exact16 entry at O0. This is
object-generation evidence, not 34288 runtime observations.

The v57p intermediate build passes 88 dedicated extended runtime
observations: 16 full-range/boundary, 40 exact16, 16 exact32 and 16
void/mixed-width conversion. Four previous static-pointer object
profiles also pass. The final v57q build has 48 identical extended
assembly comparisons against v57p covering these profiles, and v57p
has 128 identical local comparisons against v57n. The final scalar
predicate refinement is extended-only. No historical matrix cells
were added or reclassified. Inventory: 2400 matrix sources, 442
special tests, zero unowned sources.

Evidence: `pdp10-testkit/manifests/PDP10-GCC-GLOBAL-EXACT-CONSTANT-20261125-v57q.tsv`.
Staged compiler: TMPDIR/pdp10-gcc-exact-constant-v57q/gcc.
cc1 SHA256:
9bc4255dff6c6acb42867c9599411ba3a6351881cdf5ad6f08f0ba5e792135b8.
Use staged DAS v56y and linker v56m. No installation performed.

## Remaining limitations

Integer-cast addresses are target ABI probes, not portable C
pointers. The tested absolute scalar addition/subtraction does not
establish arbitrary constant pointer differences/comparisons,
relocation addends beyond existing signed representation limits,
aggregate strides, all cross-width cast semantics, or actual
high-address memory access. A probe of an absolute pointer-pointer
static difference was rejected as not computable at load time;
that separate case has not been repaired. Native architectural
byte/halfword/word boundary operations are the next acceptance gate.
Extended libgcc, multiple TEXT placements, KCC and mixed ABI remain
open. Preserve the existing local compiler matrix.
