# Native KCC variadic test adapter integration (v61m)

Testkit b7da89de; compiler remains KCC v61l (15e9b87).
The shared support/kcc/insns.h header defined __builtin_va_list and
macro replacements for __builtin_va_start, __builtin_va_arg and
__builtin_va_end. These became obsolete when v61j implemented native
GNU variadic builtins. The duplicate typedef rejected otherwise valid
sources, while the macros masked native stack-slot rounding and
operand evaluation. Remove these definitions; do not introduce a
second compatibility mechanism. The current shared adapter requires
KCC native variadic builtin support (v61j or later in GNU/KCC mode).
GCC's adapter and all original imported tests are unchanged.

## Verification

Evidence: TMPDIR/kcc-native-va-integration-v61m.orJ7VB.
The same installed compiler was used before and after the header change.
- Fresh affected-source comparison: 54 compile_fail -> 54 compile_only
  (argument-offset.c, function-pointer.c and varargs.c, 18 cells each).
  These fixtures are not executable runtime oracles.
- New kcc-insns-native-va-v61m.c: 22/22 runtime passes. Tests the shared
  header's native builtins with 36-bit integers, 71-bit arithmetic,
  narrow promotions, packed 3/5-byte aggregates occupying 1/2 physical
  words, cursor copying and operand side effects. Rejects macro
  shadowing. Eight stale-header negative controls pass on four CPUs
  with and without optimization.
- Combined fresh 2402-cell guard replay: 2261 pass, 90 compile_only,
  44 run_fail, 2 link_fail, 5 policy_fail, zero compile_fail.
  Exact failure-key comparison against the union of the recorded
  v61j, v61k and v61l related guard runs: 217 -> 51, 166 removed,
  zero added. All 166 prior compile failures had the duplicate
  typedef diagnostic (54 argument cells and 112 literal/byte-pointer
  cells). This comparison uses recorded prior results, not a fresh
  baseline of the full combined selection. The remaining keys are
  the 44 historical wide-field runtime failures (bf64-1.c and
  991118-1.c), two unresolved vprintf link cases (va-arg-21.c), and
  five pre-existing code-generation policy failures (dimode-wide-literal.c
  and bitwise-literal.c). No test exclusions or status-policy changes.
- Fresh historical 1042-cell replay: 848 pass, 128 compile_fail,
  66 run_fail; all 194 failure keys and classifications unchanged.
  v61l remaining manifests are still authoritative.
- Four registered special guards for old-style prototypes (320 checks),
  literal differences (32), bitfield fragments (8), and void
  indirection (36) pass.

## Footprint and scope

No compiler or library source, binary, runtime archive or ABI change.
No additional target compiler text, data or BSS words. No new install
archive needed. Installed compiler SHA256 remains
f1f01ac47de106f831d9c5037890bc980b5dac899ad18958b171f0f4b9c29924.
No full compiler matrix sweep, native self-host, timing or peak RAM
measurement. No GCC, DAS or DAIMOS source changes.
