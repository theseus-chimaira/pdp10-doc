# KCC static label differences and DAS BYTE expressions (v61o)

KCC 26ca5e8, DAS c096e37, testkit 9dbb2aaf.
GNU/KCC-mode static integer initializers may contain a difference
between two labels of the same function. The parser retains the
existing subtraction tree and permits equal-width or narrowing
integer conversions through 36 bits. Identical label subtraction
folds to zero. Different labels remain symbolic until assembly.
The recognizer lives in ccnode.c, shared by KPARSE and KGEN; no
new KIR node, persistent metadata, or runtime initializer is needed.
The evaluator no longer interprets a label symbol reference as a
numeric pointer payload in binary folding or casts. Label addresses
require function scope and GNU mode.

Each label denotes the first 9-bit C byte of its instruction word.
Emit four times the assembler word-address difference. The existing
DOBJ same-section relocation algebra cancels the common text base;
no relocation survives in the resulting table entry. Ordinary word,
native byte/halfword, packed aggregate and single-word bit-field
initializers retain their destination layout. Cross-word packed
values use absolute expression fragments. No new runtime helper or
jump-table storage. General symbolic arithmetic, widening after a
narrowing cast, and distinct-label multiword integer initialization
are not implemented by this change.

## DAS parser

The BYTE field scanner formerly stopped at every opening parenthesis,
including those after arithmetic operators. Track expression nesting
and preserve the field delimiter so grouped arithmetic and adjacent
mixed-width markers work. Leading parenthesized fields retain the
existing width-marker grammar. Same-section differences resolve to
absolute values; independent relocation bases and surviving byte
relocations remain errors. No object-format change. Host DAS/DAS2
and native phase sources share the corrected scanner.

## Verification

Evidence: TMPDIR/kcc-label-diff-v61o.mFab7P. Final prefix: final/.
- Original labels-3.c and new static label difference oracle: 44/44
  pass. Baseline 44 compile_fail. Runtime oracle checks independently
  computed differences, negative values, native halfword tables,
  packed cross-word scalar/bit-field values and computed dispatch.
- Registered KCC special test: 72/72 GNU89/99 accepts with assembly,
  strict C89/C99 rejection, invalid initializer and scope checks.
  No .LINK runtime initializer for the original source.
- DAS fast input/DXR equivalence suite passed. New registered DAS
  test checked DOBJ1 and paired DXR payloads independently, including
  forward labels, nested arithmetic, adjacent width markers and
  negative fields, no residual relocations, four invalid expressions.
  Existing v1817 relocation identity guard passed. DAS special tests
  were invoked directly because run-id requires suite preparation.
- Final historical 1042-cell replay: 892 pass, 84 compile_fail,
  66 run_fail. Exactly 22 labels-3.c cells recovered; no new keys.
  Updated manifests PDP10-KCC-COMPILE-REMAINING-20261128-v61o.tsv
  and PDP10-KCC-COMPILE-RUNTIME-TRANSITIONS-20261128-v61o.tsv.
- Expanded 3422-cell guard replay: 3183 pass, 180 compile_only,
  44 run_fail, 2 link_fail, 13 policy_fail, zero compile_fail.
  Compare recorded v61n 3166-cell guard with a fresh 256-cell extra
  baseline (194 pass, 18 compile_only, 44 compile_fail). Combined
  baseline 3139 pass, 180 compile_only, 103 failures. Exactly 44
  compile failure keys removed (22 original, 22 new oracle), zero
  added. Remaining failures unchanged; no exclusions or reclassification.

## Native footprint

Baseline v61n cross-compiler builds both revisions for PDP-6;
DAS DOBJ text sizes in 36-bit words:

| Unit | Before | After | Delta |
| --- | ---: | ---: | ---: |
| ccdecl | 14200 | 14206 | 6 |
| cceval | 7316 | 7371 | 55 |
| ccgen | 5230 | 5453 | 223 |
| ccnode | 552 | 634 | 82 |
| ccstmt | 9372 | 9396 | 24 |
| Total | 36670 | 37060 | 390 |
| DAS phase 1 | 14403 | 14440 | 37 |
| DAS phase 2 | 12683 | 12725 | 42 |

Data and BSS unchanged in every unit. KGEN-specific ccnode and
ccgen objects also cross-compile and assemble. The first DAS phase
comparison accidentally selected the live das.c include for both
revisions; discarded. Correct isolated include paths in
 das-native-size-v2.log establish the figures above. No linked
native phase size, native guest/self-host execution, peak dynamic RAM
or timing measurements. No full compiler-matrix sweep, extended-section
label arithmetic, GCC or DAIMOS source changes.

## Installation

install-v61o/local.tar.gz contains matched bin/kcc, bin/das and
bin/das2, installed in PDP10_PREFIX and compared byte-identical.
Previous binaries retained in the same versioned directory.
The pdp10-dec-none-as alias remains a symlink to das.

- bin/kcc SHA256: e7dd52e0eca6b5d6518d31b8e7486959cafc085ce594ca71189dce4cca5327f6
- bin/das SHA256: 1314796d2fc99d41e74053afd52f496d7734ef64750607966c6d779f5c8977de
- bin/das2 SHA256: 473d90ce0b377f879317ad62e7e9f634e741ec34fb2f233193d3469c24cc968a
- local.tar.gz SHA256: 4230d727e0d485cde76e0a52ce991ed711a9cfc3def63cc6aaabf35f010f5fc1
