# GCC packed exact-width symbolic relocation validation (v57v)

GCC 3460733, testkit f3e2e68; based on accepted v57u (00463d2).
No assembler, linker, emulator, DAIMOS or production installation change.

## Defect and repair

The packed exact16/32 symbolic constant-adjustment helper previously
masked its signed word displacement without checking representability.
The RTL encoding uses six tag bits and a signed 30-bit displacement.
The first positive overflow (+536870912) became -536870912; the first
negative overflow (-536870913) became +536870911. The assembler could
produce a structurally valid relocation carrying the wrong addend.
The common exact-width adjustment helper now checks the signed range
before masking and reports an error. This does not change absolute
coordinate folding or generated dynamic arithmetic. It does not
introduce a wider symbolic representation. The diagnostic applies to
constant symbolic adjustments through this helper, including constant
expression paths in functions; it is not a general runtime range check.

## Acceptance

`tools/test-gcc-global-packed-symbolic-v57v.py` uses independent linear
bit coordinates for external packed records of widths 16 and 32.
Four prefix lengths and nine element indices cover all starting bit
positions. Each reachable position in the selected displacement word
is checked through addition, subtraction of the opposite count and
split chained adjustment. Both signed displacement endpoints and
section carries/borrows are included. Synthetic large displacements
are ABI/relocation probes rather than portable C array accesses.

All four optimization profiles pass 2133 positive and 2187 negative
relocation checks per profile, for 17280 object relocations and 17280
final linked pointer words. Both low and high DATA placements are
used without executing these high synthetic addresses. The v57u
baseline also passes the in-range cases. The new rejection controls
cover both widths and both first out-of-range displacements at four
optimization profiles. The baseline accepts all 16; independent
object inspection confirms all 16 wrong-sign addends. The candidate
rejects all 16 with the explicit range diagnostic. A separate O0
probe confirms the same diagnostic for direct indexing at those four
endpoints. The --probe-overflow mode reproduces baseline acceptance;
the manifest records the independently inspected baseline objects.

Earlier native symbolic relocation and linked-word checks (4944 each)
pass again. All 171440 previous exact-width absolute constant checks
pass. All 56 extended assembly guards and 112 local assembly guards
are identical to v57u. No additional emulator runtime regression was
run in this stage. The inventory passes with 2400 unchanged matrix
sources, 447 special entries and zero unowned sources.

Evidence: pdp10-testkit/manifests/PDP10-GCC-GLOBAL-PACKED-SYMBOLIC-20261125-v57v.tsv
(8 accepted object/link profiles, 16 rejection controls and 16
baseline wrong-addend objects). Staged GCC:
TMPDIR/pdp10-gcc-packed-symbolic-v57v/gcc. cc1 SHA256:
237f1576f26ebfba1e369323dd7c4c9fe50339bc7df97e49c7f30e00d508457c.
Use staged DAS v56y and linker v56m. No installation performed.

## Remaining work

Wider symbolic addends require an RTL representation that retains
selector and signed displacement independently. Do not reinterpret
signed 30-bit overflow as a legal relocation or silently narrow
counts. Other native and direct aggregate symbolic paths have not
been exhaustively checked for overflow. Combined expressions with
out-of-range intermediates are not accepted by the current helper.
Additional aggregate layouts, casts, resident-size optimization,
extended runtime, multiple TEXT placement and KCC/mixed-compiler ABI
remain open. Preserve the historical compiler matrix.
