# GCC typed global exact32 pointers (v57d)

GCC b19c593, testkit 3549910, based on v56x/v56y. This is a
verified GCC repair for native and continuous packed exact32 pointers
in extended mode. It is not yet a frozen mixed GCC/KCC ABI.

## Representation and lowering

A native exact32 pointer is a global word address (tag zero). A
continuous packed exact32 pointer has a high-six-bit tag in 1..36
(decimal), representing physical bit position tag-1, and the low
30 bits carry the global word address. Symbolic tagged addresses
emit `%GADDR(tag,address)` with octal fields and the existing ADDR30
relocation. The compiler must never emit such a software pointer as
an architectural OWGBP. Native static pointers continue using GIW.
Static array element addresses use laid-out packed field positions.
Dynamic member addresses in word-addressed records, including
stack records and pointer parameters, retain the entire sectioned
word coordinate before installing the tag.

The exact32 type predicate now recognizes extended as well as local
compilation. Typed indirect loads/stores distinguish tag zero from
packed positions. Packed loads and stores reuse the physical scalar
extract/insert logic and access a second word only for crossing
fields. Signed loads sign-extend; native stores normalize the value
to declared precision. Volatility is retained. The local access
and S=1 pointer representation remain supported.

For a packed advance by n elements, let q=trunc(n/9), r=n-9*q.
Add n-q to the word coordinate, subtract 4*r from the tag, then
normalize the tag with at most one word carry or borrow. Constant
addresses fold without creating a pseudo during initialization.
Dynamic arithmetic uses a signed quotient, bounded residual and
ordinary integer operations, not ADJBP. Tag placement uses logical
LSH, since ASH cannot shift a tag into the sign bit correctly.
Native advancement is ordinary word addition. Packed subtraction
returns C storage units: (9*(word difference)+(tag difference)/4)/2
for same-array operands. The front end divides by four units per
element. Native subtraction returns four times the word difference.

Relational lowering rotates the tag into the low six bits and
biases the sign bit for signed ordering by word coordinate and
then physical bit position. Both branch and materialized boolean
paths use the same transformation. Equality compares the encoded
word. No helper or correction table is added. The inline operations
are relatively large; later common runtime/size work should assess
sharing, rather than claim a RAM or speed improvement now.

## Acceptance

The v56x compiler reproducibly fails compilation of the historical
`gcc-exact32-pointer-representations-v51a.c` at line 18, the packed
static pointer initializer, for O0/O1/O2/Os. The candidate compiles,
links and executes that unchanged fixture at all four optimization
levels on both SIMH and KLH10. The new dedicated fixture checks
native and packed pointers through shared noinline functions,
static pointer initializers, signed/unsigned reads and writes,
25 ordered subtraction/ordering pairs for each of two array forms,
pre/post adjustments, increment of a stored exact32 value, wrap at
32 bits, direct and indirect global member addresses and automatic
packed records on the global stack. It also passes all eight
extended executions. Neighboring fields remain intact.

The historical exact32 fixture passes 16 local runs (PDP-6, KA10,
KI10, KS10, four optimization levels), with byte-identical assembly
against v56x. A further 64 local codegen comparisons are identical.
Previous extended tests pass 82 runtime observations (16 record,
8 narrow, 10 linked/archive, 48 wide/direct-packed), and four
static-pointer object checks pass. Test inventory audit: 2400
matrix sources, 437 special entries, zero unowned sources.

Linked TEXT words in the new fixture: O0 2132, O1 1189, O2 1181,
Os 1174; DATA 2, BSS 18 for each. These are test-image counts, not
peak RAM or timing measurements. No functional extended baseline
exists for the same fixture. The historical local matrix is not
expanded or reclassified. No claim of historical matrix recovery.

Evidence: `pdp10-testkit/manifests/PDP10-GCC-GLOBAL-EXACT32-20261125-v57d.tsv`
contains 16 extended and 16 local passes plus four baseline compile
failures, with source, image, log and tool hashes. Dedicated manual
runner: `tools/test-gcc-global-exact32-v56z.py`.
Staged GCC: TMPDIR/pdp10-gcc-exact32-v57d/gcc; cc1 SHA256
472678f0592caaa1d8e163bcaa9a7a4c9a6ee563bb1205117895d7a39011aaff.
Use staged DAS v56y and linker v56m. Production binaries unchanged.

## Remaining limitations

Exact16 global pointers remain unresolved; native slots and packed
positions must be handled consistently. Cross-width pointer casts,
void-pointer round trips, additional aggregate layouts, typed arrays
spanning section boundaries and arithmetic-only full 30-bit range
need more compiler-level validation. The previous v56y feasibility
model is not a substitute for these tests. KCC extended support and
mixed-compiler ABI acceptance are still open. Multi-TEXT placement
and full extended runtime remain pending. No DAIMOS changes.
