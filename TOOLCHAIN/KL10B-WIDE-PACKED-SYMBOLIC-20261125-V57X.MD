# GCC wide packed symbolic relocation constants (v57x)

GCC e54b014, testkit 014906f; based on v57v. The DAS and DOBJ2
linker are unchanged. No DAIMOS or production installation change.

## Representation

The previous compact symbolic RTL combined the six-bit pointer tag
and a signed 30-bit word displacement in one encoded word. v57v
rejected packed exact16/32 symbolic adjustments outside that range
rather than allowing an opposite-sign relocation. The new extended
CONST(UNSPEC_GLOBAL_RELOC) form keeps the base symbol, selector and
signed word displacement in separate operands. It is used only for
wide displacements; the compact form remains for in-range constants.
Exact-width arithmetic shares a host coordinate calculation with
absolute folding, which wraps only absolute pointer words. Symbolic
arithmetic retains the signed displacement and checks the full
architectural magnitude. Displacements outside +/- (2^30-1) receive
an explicit diagnostic. The assembler output is %GADDR(tag,symbol+offset),
using existing ADDR30 records. No new target word, table or runtime
helper is required. Constant exact32 subtraction also retains its
constant count when negating it for extended pointer adjustment.

## Acceptance

The dedicated tools/test-gcc-global-wide-packed-symbolic-v57w.py
runner generates packed exact16 and exact32 symbolic initializers
with independent linear bit coordinates. It tests displacements on
both sides of the old signed endpoint, near the ends of the full
address range, and chained adjustments crossing the former boundary.
Each of four optimization profiles checks 1836 positive and 1782
negative static relocations and the corresponding linked words,
for 14472 of each. Eight distinct code-literal relocations per
profile (64 total) are checked before and after linking. The positive
placements also run 24 function-return comparisons per profile on
SIMH and KLH10: eight successful executions. Negative placements
use high synthetic DATA addresses and are checked at object and
link level only. No high synthetic pointer is dereferenced.

Sixteen extreme displacement objects (both signs of 2^30-1, both
widths, four optimization profiles) retain the exact displacement;
sixteen impossible displacement cases (+/-2^30) are rejected. The
v57v baseline rejects both wide translation units at O0. Actual
relocated addresses must still fit in the address space. The earlier
packed endpoint runner now requires correct wide relocations at the
old signed-range boundary, and retains its v57u baseline probe mode.

All 17280 earlier packed relocation and linked-word checks, plus 16
formerly rejected wide relocation checks, pass. The 4944 native
symbolic relocation and linked-word checks and 171440 exact-width
absolute constant word checks pass again. The exact16 (40), exact32
(16) and void/mixed conversion (16) runtime replays pass, totaling
72 previous runtime observations. All 56 extended and 112 local
assembly guards are byte-identical to v57v. Inventory: 2400 unchanged
matrix sources, 448 special entries, zero unowned sources.

Evidence: pdp10-testkit/manifests/PDP10-GCC-GLOBAL-WIDE-PACKED-20261125-v57x.tsv
(8 link profiles, 16 extreme object checks, 16 rejection controls,
2 baseline controls). Staged GCC: TMPDIR/pdp10-gcc-wide-reloc-v57x/gcc.
cc1 SHA256:
9be1640af362c0580fcc25e11439c0dea7f84fd33cef163683d07bef90f01c7f.
Use staged DAS v56y and linker v56m. No installation performed.

## Remaining limitations

This acceptance covers packed exact16/32 symbolic adjustment and
returning the resulting pointer values. Native ADJBP, plain word
symbolic arithmetic, direct aggregate construction and cast consumers
of the old compact RTL have not all been migrated to the wide form.
Wide cast chains, indirect accesses and general addressing paths
require their own validation. The full range of addend magnitudes at
object level is not a claim that all placements link successfully.
Additional combined expressions, aggregate strides, resident-size
optimization, extended runtime, multiple TEXT placement and KCC
remain open. Preserve the historical compiler matrix.
