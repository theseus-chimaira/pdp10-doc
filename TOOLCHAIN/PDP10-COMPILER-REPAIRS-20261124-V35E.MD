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
