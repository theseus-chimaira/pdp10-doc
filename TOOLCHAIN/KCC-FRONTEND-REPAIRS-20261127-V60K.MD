# KCC frontend repairs and explicit extension scope (v60g-v60k)

Date: 2026-11-27. Installed host compiler: KCC 2fe79a0.
Testkit: 5493f50. No DAIMOS runtime, console, source oracle, GCC
matrix column or compiler-owned runtime archive was changed.

## Verified changes

- v60g (KCC ec6604f, testkit da0c997): complete external
  tentative incomplete arrays at translation-unit end, accept GNU
  qualifier spellings using existing tokens, provide __PTRDIFF_TYPE__,
  parse compound literals in sizeof, and repair parenthesized
  __extension__ expressions. Original replay: 176/242 PASS;
  92/92 regression guard cells PASS. A stale incremental isolated
  build was rejected; acceptance used a fresh build.
- v60h (KCC b310698, testkit a105bdc): fold full-width 71-bit
  constant multiplication with existing two-word hosted arithmetic
  and native wide arithmetic. Independent literal oracles exercise
  signs, low/high boundary, modulo wrap and zero. Original 22/22;
  total acceptance 110/110 PASS.
- v60i (KCC 31179ec, testkit 946de6f): preserve identifier start
  line for object-like macro replacement, including backslash-newline
  and split identifiers; use closing-parenthesis line for function
  macro replacement, including object-like aliases. Accept and
  validate GNU #ident without emitting loadable identification
  storage. PDP-10 object formats used here have no identification
  section. Original 26/26; total acceptance 100/100 PASS.
  Multiline macro argument token location is not fully implemented;
  an __LINE__ inside an argument may still use the closing line.
- v60j (KCC 5ec33a3, testkit ce8cba2): bound implicit nested
  array initializer lists to the current subobject; preserve the
  enclosing comma and designator boundary in implicit arrays and
  structures. Verify inferred bounds, zero fill, mixed aggregates
  and explicit excess-element diagnostics. Original 22/22;
  total acceptance 110/110 PASS.
- v60k (KCC 2fe79a0, testkit 5493f50): accept GNU member: value
  initializer spelling in extension mode through the existing
  designated initializer machinery, including promoted anonymous
  members and mixed designator forms. Preserve positional identifier
  expressions and diagnose unknown members/union positional excess.
  Original 66/66; total acceptance 136/136 PASS.

These are targeted repairs to existing frontend mechanisms, not
claims of complete GNU extension conformance. No persistent
compiler type, node or token structure was enlarged. Native
self-hosting, peak RAM and code size have not been independently
measured.

## Explicitly unsupported scope

Testkit c9c0bd2 records 47 source registrations and 1,000 previously
failing KCC cells as unsupported in the compiler matrix. Reasons
are listed in manifests/PDP10-KCC-UNSUPPORTED-SCOPE-20261127-v60g.tsv.
Excluded categories include nested functions/trampolines, complex
arithmetic, zero-sized aggregates/arrays, block-local GNU labels,
weak/alias linkage, GNU machine modes, variable-length structure
members and selected obsolete GNU extensions. The existing
unsupported matrix status is used; none of these cells is a PASS.
GCC selection remains 85,799 cells. Extended inline assembly and
potentially useful but incomplete features remain active.

## Cumulative verification

The imported full sweep contained 2,196 KCC compile_fail cells.
Exact cell-key reconciliation: 1,000 excluded, 312 recovered,
884 remaining from that original cohort. All 312 recovered cells
were replayed with the final installed compiler: 312 PASS, zero
failures, zero cached results. This does not constitute a fresh
full-matrix result or establish absence of other regressions.

Isolated evidence under TMPDIR:
kcc-frontend-v60g.TOtigf/installed-replay-v1,
kcc-constants-init-v60h.3r0hXt/mul-guards-v3,
kcc-pp-v60i.0J2tDH/guards-v2,
kcc-brace-v60j.1j89Xe/guards-v1,
kcc-designator-v60k.KtVf28/guards-v1,
kcc-cumulative-v60k.uQtWoZ/replay-v1.
The cumulative directory includes exact recovered/remaining cell
manifests and the installed compiler hash.

Final compiler-only install archive:
kcc-designator-v60k.KtVf28/install-v1/local.tar.gz
SHA256 1ec45e6ee8aa0b48284e4b624a7e3e27bf007fa36787216ec8d19e7f70b4c010
Installed bin/kcc SHA256:
18d87622dfb26dc6713a4e0426574a6b9513685094da3f7ea497fb597400392c

Integral bitfield extension experiments are not included: certain
sources compiled but did not pass runtime verification. Wide fields,
static address difference folding, typedef alignment and the remaining
GNU compatibility cases are not accepted by this series.
