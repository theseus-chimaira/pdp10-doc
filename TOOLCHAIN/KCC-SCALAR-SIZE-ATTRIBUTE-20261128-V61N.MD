# KCC native scalar size attribute (v61n)

KCC 1cd3147, testkit 85d4f28d. GNU/KCC mode accepts the PDP-10
`__attribute__((size(N)))` and `__size__` spelling with a single
integer literal width N in 6, 7, 8, 9, 16, 18, 32, 36. It selects
an existing integer storage/code-generation representation, preserving
signedness and qualifiers. The attribute applies to integer scalar
base declarations or individual scalar declarators, including typedefs,
parameters and members. The lexer validates the full literal and
requires consumption before advancing beyond the associated token;
invalid widths, conflicting pending widths, malformed or misplaced
attributes are diagnosed. General integer constant expressions in the
attribute argument are not implemented. Strict C89/C99 still reject
the GNU spelling.

The native word and default integer remain 36 bits. Narrow storage
values promote as before; no new arithmetic mode or runtime helper.
For GNU size(16), use the existing short type family with precision 16,
so sizeof is two 9-bit units. The historical _KCCtype_char16 spelling
continues to have sizeof 1; this is intentionally not an ABI migration.
The GNU size(32) spelling uses the existing exact-32 integer type,
not a new default. Unbound char6 in a type position denotes unsigned
six-bit char, matching the historical PDP-10 GCC builtin typedef.
Explicit user declarations (including the test adapter's signed
char6 typedef) take precedence; no unconditional typedef is inserted.

## Verification

Evidence: TMPDIR/kcc-size-attribute-v61n.H8hw7x, candidate-v3.
- Original ptr-arith-2.c and independent all-width oracle: 44/44
  runtime passes, baseline 44 compile_fail. Signed limits, unsigned
  maxima, array extent, physical first word, all pairwise element
  differences in 13-element arrays, declaration scope, parameter
  conversion, aggregate member, qualifiers, char6 fallback/override.
- Registered special test: 192 syntax/scope checks, 24 language-mode
  checks (8 GNU89 accepts, 16 strict C89/C99 rejects), and eight
  byte-identical assembly comparisons between old and new type
  spellings (loads, stores and pointer differences across all widths
  and signednesses, four CPUs, opt/noopt).
- Fresh historical 1042-cell replay: 870 pass, 106 compile_fail,
  66 run_fail. Exactly 22 cells of ptr-arith-2.c recovered; zero new
  failure keys, unchanged runtime keys. New authoritative manifests:
  PDP10-KCC-COMPILE-REMAINING-20261128-v61n.tsv and
  PDP10-KCC-COMPILE-RUNTIME-TRANSITIONS-20261128-v61n.tsv.
- Expanded 3166-cell guard replay: 2945 pass, 162 compile_only,
  44 run_fail, 2 link_fail, 13 policy_fail, zero compile_fail.
  Baseline comparison uses v61m's recorded 2402-cell guard replay
  plus a fresh baseline of 764 extra cells (640 pass, 72 compile_only,
  44 compile_fail, 8 policy_fail). Union of exact failure keys:
  103 -> 59, 44 recovered, zero added. The original source and the
  new oracle account for the 44 removed cells. No exclusions or
  classification changes. Remaining policy failures and the two
  historical wide-field runtime sources and vprintf link cases
  are preexisting.

## Native compiler footprint

Baseline v61l cross-compiles both source revisions for PDP-6 with
-Pgnu99 -O -m=gas -DHOST_DAIMOS=1 -DHOST_UNIX=0; DAS DOBJ headers:

| Unit | Before text | After text | Delta |
| --- | ---: | ---: | ---: |
| ccdecl | 14021 | 14200 | 179 |
| cclex | 3616 | 3779 | 163 |
| ccstmt | 9377 | 9372 | -5 |
| Total | 27014 | 27351 | 337 |

Lexer BSS increases from 1061 to 1062 words, for the pending size
attribute. Data unchanged, all other BSS unchanged. No new fields in
TYPE, SYMBOL or NODE. No claim about final linked phase size, peak
RAM, native execution time or native self-host execution; no full
compiler matrix sweep. No GCC, DAS or DAIMOS source changes.

## Installation

Compiler-only install-v61n/local.tar.gz installed into PDP10_PREFIX
and compared byte-identical with candidate-v3. Previous compiler
retained in install-v61n/previous-kcc-v61l.

- bin/kcc SHA256: 370e5a106373c8f885d81fcfd0c14e31461fd43899e3e1ed30278e54ee72d4d8
- local.tar.gz SHA256: 456e4cb757dc890ea7160a8261aca74c603956090193400ed6ff7afa44916ad6
