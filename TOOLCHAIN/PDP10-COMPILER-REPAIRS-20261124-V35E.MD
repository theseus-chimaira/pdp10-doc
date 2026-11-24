# Historical PDP-10 compiler repair verification, v35e (2026-11-24)

## Implemented repairs

- GCC (4a71edc): convert packed 9-bit alias-name metadata to host
  identifier spelling, emit a delimited `.set` directive, and reduce
  local address offsets to the 18-bit effective-address field. Do not
  emit global indirect words for local addressing. All 44 original
  `20020527-1.c` GCC cells pass; extended memory is not required.
- DAS (0a3542a): retain the section-relative value of local assignments
  and resolve `.set` snapshots before exporting their DOBJ definitions.
  The integration regression checks forward text/data/BSS aliases and
  a data `.equ` at nonzero offsets. External-symbol aliases and weak
  symbol binding are not added by this change.
- KCC (178dac0): implement C99 control scopes with existing symbol
  scope/VLA machinery, preserve ordinary `#` tokens in object macros,
  classify compound literal lvalues correctly, and implement GNU void
  return, alignment, and void-pointer arithmetic. Hosted decimal
  literal parsing uses one `strtod` conversion instead of repeated
  digit rounding. The native decimal parser remains unchanged; the
  hosted 53-bit intermediate does not yet supply all 62 target bits.
- Testkit selects the original strict C99 flags for the three historical
  batch06 runtime sources. The original sources are unchanged.

## Native floating-point regression

The retired `960405-1.c` semantic test assumed an exponent far outside
native floating-point range. Its new regression, `pdp10-long-double-fold-range-v35c.c`,
uses X = 2^126 - 2^99, exactly representable even at 27 fraction bits.
Both static initializer expressions X+X and 2*X must equal the
independently specified 2^127 - 2^100. The oracle also rejects failure
to increase the value. No new runtime floating format is introduced.
The original out-of-range source remains an overflow-diagnostic fixture.

## Evidence

Installed exact original-v28 replay: 126/126 PASS (64 GCC, 62 KCC).
Original classification: 45 GCC compile_fail, 19 GCC run_fail, 62 KCC
compile_fail. The KCC scope/VLA/macro guard replay passed 88/88 in
staging; DAS DOBJ/DARC/DLINK integration and overflow diagnostics pass.
Installed hosted binary SHA256 values:

- GCC cc1: 386ae2060f92b631e07b8e40593e0e6cc2421f3e06a2a62a734a17a234e885bf
- DAS: 2a22c152088a9cee6d3085ce787c0796d46b1b9e903217dc760b343c4997cfa1
- KCC: 61784964a4044ffdc7fca2906f070fdebe44ea207ddd51c45cba7c70950f3a9c

Exact results are recorded in pdp10-testkit's
`manifests/PDP10-V28-REPAIR-RESULTS-20261124-v35e.tsv`.
No full sweep, native compiler-phase execution, or native DAS build is
claimed. Three original feature gaps remain active: weak linkage (4 GCC
cells), GNU zero-size aggregates (22 KCC cells), and complete wide
character/string semantics (22 KCC cells). They are not hardware-based
exclusions and must not be counted as passes.

## KCC-only repair scope (2026-11-24, v36a)

The user has explicitly excluded GNU zero-sized aggregates from KCC's
supported language subset. The 22 historical KCC cells of
`20000801-3.c` are now unsupported; all 44 GCC cells remain active.
This is an optional compiler-feature policy, not a PDP-10 hardware
limitation and not a successful test. Retain the original source and
historical records. Four additional historical C99 scope cells passed
with the installed v35e KCC. The remaining KCC cohort is 147 cells:
22 wide literal compilation failures and 125 runtime failures.

KCC already supports `_KCCtype_char6`, `char7`, `char8`, `char9`,
`char16`, `char18` (with the `_KCCtype_` prefix), `_KCCtype_int32`, and
native 36-bit `int`. No new scalar representation is necessary for
wide literals: reuse native `int` as `wchar_t`, consistent with GCC.
The current lexer uses the ordinary narrow literal type and refuses
mixed-width concatenation. Wide literal storage, typing, emission and
serialization remain to be implemented while preserving compact narrow
literal storage. No compiler implementation changes were made in v36a.

## KCC native-width literal implementation (2026-11-24, v36c)

KCC commit c9e688e implements `wchar_t` as the existing 36-bit `int`.
Wide character constants and wide string arrays use the native word
representation. Adjacent mixed-prefix literals produce a wide array;
array extent, address-taking, indexing and initialization retain the
correct element type and count. Wide universal escapes use Unicode
scalar values. Full multibyte source decoding and narrow universal
character conversion are not claimed.

The existing literal pool retains one octet per ordinary narrow
character. Ninth-bit narrow literals and wide literals use two and
five octets per element respectively. Promotion expands only the
current literal; no new permanent pool or node fields are needed.
KIR1 explicitly transports the storage flags and payload octets.
The existing fixed pool capacity is unchanged. The host byte source
reader no longer masks the high bit; packed seven-bit ASCII and C-SIX
storage paths remain unchanged.

Installed strict replay: 44/44 original KCC cells PASS (22 wide
literal compilation failures and 22 high-byte runtime failures).
The new `kcc-wide-literal-v36b.c` regression passes 22/22 KCC cells.
Hosted split-phase tests preserve exact wide words through KIR1 and
KPCODE2 in optimized and unoptimized modes. Four changed native
PDP-6 phase modules cross-compile; native phase execution and RAM
measurements remain unverified. C-SIX source decoder regression passes.
GCC was neither modified nor tested. No full sweep was run.
Remaining original KCC cohort: 103 runtime failures, no compilation
or linker failures. GNU empty aggregate policy is unchanged.

## KCC runtime repairs v37d

Hosted compiler commits b929249, f7d138c, c8618fb implement correct UL
suffix typing, GNU-mode undeclared memcpy/memset prototype selection,
floating addition constant checks, argument preservation across numeric
conversion helper calls, and double comparison without the redundant
continuation sign bit. Wide integer shifts consume the low count word,
multiplication includes the contribution at bit 70, unsigned division
compares the reconstructed remainder correctly, and scalar truth uses
both words. Constant logical RHS folding avoids a synthetic comparison
crash. No new permanent compiler data structures or runtime library ABI.

Installed hosted SHA256:
6cc10d24ce39ad7fe2b9362f63edafe06bc04ea9f9b4906e94266cd39fb5c2c4.
Strict installed exact replay: 97 of 103 historical KCC runtime cells
pass; all 66 new regression cells pass. Six remaining runtime cells:
PDP-6 opt/noopt enum bit-field signedness, wide Karatsuba union case 4,
and prime interval instruction budget. The optimized prime interval
v37c assembly passes a diagnostic 1-billion-step run, but the standard
100-million-step budget remains unchanged. No test exclusions, GCC
work or full sweep. Native execution and RAM measurements unverified.
See testkit's v37d results and six-cell remaining manifests.

## GCC exact-type cohort runtime verification (v40a)

The staged candidate 71d4100, on GCC branch repair/packed-ice-v40a,
was tested against all 852 historical exact-type compilation-failure
cells. 148 passed and 704 failed runtime assertions; zero compilation,
assembly or linker failures occurred. All 165 former packed-aggregate
ICEs reached execution, but none passed. Of the remaining 687 cells,
148 passed and 539 failed. The 32 existing packed-record pointer guard
runs passed. No test was excluded or weakened. The separate historical
2614 GCC runtime cells were not replayed.

The review candidate addresses union field-position recursion,
word-boundary aggregate bit transfers and byte-selector normalization
for word-oriented bit-field expansion. Packed aggregate semantics are
not yet accepted: a separate small packed-copy probe passes PDP-6 O2
but fails O0. GCC master and the installed compiler remain unchanged.
The staged cc1 SHA256 is
c1182aaf18f2dad471794e82f32c93ff249ea372e13e6af627cb02dcc604f07d.
Testkit commit edc3cfc records exact result hashes and the 704-cell
remaining manifest. This is verification evidence, not a claim of
complete packed aggregate support. No full sweep was performed.

## GCC packed aggregate runtime repairs (v41b)

GCC review commit d4867fe, based on 71d4100, preserves physical
36-bit memory references for aggregate copies and uses the target
extraction/insertion patterns directly. Constant packed integer
references consume the declared field positions and array strides.
Integer-only packed static constructors stream target bytes from
that layout without a full aggregate buffer. Other constructor
kinds retain the existing emission path.

Staged hosted SHA256:
60fed2b3a4ff38f83418333207f666c3e88b82b5c43d8e27031013bdff3b7c2a.
Exact acceptance: 324/324 PASS, including 176 previously failing
cells and 148 previously passing guards. All 165 former ICE cells
now pass. The four recovered sources are packed unions, chained
assignment, nested by-value, and nested whole-copy. New packed-slice
regression 44/44 PASS; existing packed-record pointer guard 32/32
PASS. Testkit commit 12ec30c records source, assembly and report
hashes. 528 of the v40a runtime failures remain. No full sweep,
installation or merge to GCC master. Native execution and RAM cost
have not been measured.

A separate diagnostic candidate 61b50b5 (v41c) prioritizes constant
packed stores over the older exact16 logical-pointer expansion.
The array-members test advances from assertion 2 to 7 on PDP-6 O0
and O2, but all six focused probes still fail. It is not accepted;
indirect packed pointer representation and arithmetic remain open.

## GCC packed pointer subtraction (v42)

Review commit 3c547c4 handles local exact16 native S=16 and logical
S=1 pointer subtraction separately. Logical subtraction uses the
physical word and P-field difference, converted to C storage units.
Constant logical adjustments scale element counts to bit counts.
No persistent target storage was introduced. The staged hosted cc1
SHA256 is bbceffb8a867162b0165edd2658159aed37c53fef23cb09541a19824a20bc3f8.
All 412 historical acceptance cells passed (88 recovered and 324
previous passes), plus 44 packed-slice regression cells. Testkit
9e0b826 records exact hashes and the remaining 440-cell manifest.
The separate 2614 historical runtime cells were not replayed.
Extended pointer semantics and native RAM cost remain unverified.
No master merge, installation or full sweep.

## GCC packed element counts (v42a)

Review commit d30e21b separates explicit element-count adjustments
from the older C storage-unit adjustment path. Direct packed array
indexing and dynamic address expansion no longer divide already
normalized counts. The one-word pointer representation is unchanged.
Staged cc1 SHA256:
3a978d119a1cf40ff6dbd0c7390dda2bcba1598a043e981c923d71eeb7be7c0e.
All 544 historical acceptance cells passed, including 132 recoveries
and 412 earlier passes. Testkit b099337 records the exact result
hashes and 308 remaining cells. The separate earlier 2614 runtime
failures remain untested. No master merge, installation or full
sweep; native RAM and extended addressing remain unverified.

## KCC historical runtime closure (v43)

The installed compiler based on cdcadb0 still failed all six remaining
historical PDP-6 cells. The uncommitted v38 patch was rebased onto that
revision and committed as b57fa3e. Integral signed and unsigned 71-bit
values use the duplicated sign bit at memory/ABI boundaries and a
canonical low35 register representation. Nonnegative enum types use
the unsigned compatible word type; enumerator constants remain int.
The inline wide division path uses native IDIV for small magnitudes
and retains the restoring divider for the general case.

All six repaired cells and 169 staged regression guards passed, as did
seven mixed-compiler ABI and language compatibility scripts. Installed
recheck: 6/6 PASS at the original step budgets. Installed hosted SHA256:
ec336dcde92a22311a2dba69d6940daea466a7296d4fc8668aebfef5106c6602.
Testkit de81287 records exact source, assembly and report hashes.
The tracked historical v28 KCC cohort has no active failures remaining.
No full sweep or native execution/RAM measurement was performed.

## GCC small repairs and target string reads (v44/v44b)

Review 994fef8 selects a physical low word before extracting a narrow
part of a wide memory value. All eleven historical conversion cells
passed. Four historical alias cells passed with the previous .set
correction, but weak linkage itself is still unsupported. Sixteen
arithmetic failures were test-width selection errors: generic PDP-10
target detection fixes the reference values without weakening the
arithmetic assertions. All 198 arithmetic cells passed, including
66 KCC guards. Testkit commit 1275250 records those repairs.

Review f11b292 decodes the two-host-byte representation of target
9-bit string elements at constant read sites and uses logical element
bounds. Dynamic aggregate indexing retains the byte selector. The
staged compiler SHA256 is
 a56b1ff680229d1477819eeaca6bf0413d21b3d9823918700a5d3bffb93f1f09.
All 132 string cells and 132 additional GCC guards passed, plus
22 KCC string checks. Testkit d102d2c records exact hashes. The string
repairs recover 77 historical cells, for 108 recoveries in v44/v44b.
Staged backlog: 2726 cells. No merge, installation or full sweep;
previous cumulative guard acceptance is incomplete.

## GCC floating conversion repairs (v45b)

Review 5c89d46 corrects unsigned SF conversion by reading the operand
once, shifting right for high-bit inputs, retaining a sticky rounding
bit, and scaling only that path. Integer-derived D-format values
require normalization even under optimization; a dedicated RTL
operation preserves it. The unsigned D-format exponent bias is
corrected and a second shift of the 71-bit low continuation is
removed. Negative SF-to-DF extension duplicates the sign bit.
No persistent target storage is added. Hosted staged cc1 SHA256:
3214735aa60ff2d270ddbb6c2e2a74b509eceb2784a0a568d404122ff6bd64a7.
All 30 historical floating cells reproduced on v44b and passed on
v45b; 44 new boundary regression cells and 76 existing guards pass.
The 16 packed-unaligned complex cells still fail and remain in the
aggregate backlog. Testkit 27b9a7b records the exact result hashes.
Staged backlog 2696 cells. No full sweep, master merge or installation.
G-format and native RAM/speed remain unverified, and the earlier
cumulative acceptance run is incomplete.

## GCC exact-type recheck and void-pointer repair (v47a)

All 176 remaining exact-type runtime cells reproduced on v45b.
Review eef83fe converts an incoming native record byte pointer to
its selected member's byte size before applying the member-sized
position adjustment. The dynamic address helper previously advanced
using the old record selector, e.g. an 18-bit record pointer for a
9-bit array member. No pointer-width or permanent storage change.
Hosted staged cc1 SHA256:
3bee8f496b13c3b7b5484ea78e6cdd9adb4558235a24267853198b7fc755518b.
All 44 void-pointer roundtrip cells passed, along with 32 existing
packed-record pointer guards and the mixed KCC/GCC pointer ABI suite.
Testkit 2c914d2 records the exact 176-cell baseline and 44 recoveries.
The three mixed packed-record sources still fail; 132 exact-type
cells remain. Their noncanonical record and member positions require
further work. Staged backlog 2652; the 2520 other historical runtime
cells remain separate. The v46 memory/string investigation is deferred.
No full sweep, master merge or compiler installation; native RAM
and cumulative acceptance remain unverified.

## KCC optimization quality (v49a/v49b)

57dbc60 removes an unreferenced shared epilogue after a proven
non-fallthrough instruction and preserves the lexical return identity
before evaluating the return expression. 44 affected profiles and
169 historical runtime guards pass. The 12 assembly-policy failures
in the separate epilogue suite also reproduce with installed v43.

edff620 prevents unprofitable double-constant operand folding for
PDP-6 long floating arithmetic, avoiding two scratch pushes and pops.
82 targeted cells and 169 historical runtime guards pass. All 41
size comparisons improve, including the 12 historical KCC size
regressions. The PDP-6 wide-literal case decreases from 23 to 20
words (unoptimized 22); the floating division case decreases from
41 to 34 words (unoptimized 38). Staged hosted KCC SHA256:
c62d1f2406d08f96e8ea6f2456daa8e00e02c27b6bb54f6cabd819944ea2a29d.
No installation, native RAM measurement or full compiler sweep.
Testkit edefe4d records the exact results and new regression.

The archived GCC speed audit now has 336 measured execution cells:
85 O1/O0 improvements, 56 O2/O1 improvements, 14 ties and 13
regressions, 83 Os/O0 improvements. Simulated ticks are not physical
CPU timing; historical compiler revision provenance is incomplete.
The audit and GCC optimization regression repairs remain open.
