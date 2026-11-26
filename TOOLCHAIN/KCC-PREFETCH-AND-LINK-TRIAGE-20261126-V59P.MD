# KCC prefetch and link-failure triage (v59p)

KCC 7e37fff and 09da707; testkit d986cc0b. The original November
25 sweep contains 404 link_fail cells (190 GCC, 214 KCC).
Exact first unresolved symbol or relocation classification:

| Classification | Cells |
| --- | ---: |
| malloc | 113 |
| RH18 overflow | 94 |
| printf | 74 |
| sprintf | 54 |
| alloca | 22 |
| test_longlong_integer_to_float | 12 |
| tst_longlong_integer | 12 |
| __builtin_prefetch | 8 |
| isprint | 4 |
| putchar | 3 |
| func | 2 |
| stderr | 2 |
| stdout | 2 |
| vprintf | 2 |

The exact 94 RH18 cells all pass with the already installed
integrated v1977 linker; four isprint cells pass with v59k.
No additional linker change was needed. The 22 alloca cells
remain link_fail: 20010122-1.c explicitly declares alloca and
also calls __builtin_return_address. The previous undeclared
direct alloca builtin repair is not sufficient. No fake helper
or hosted runtime was linked.

KCC now recognizes undeclared direct __builtin_prefetch calls
in ccstmt.c. A pointer address expression is converted to void;
existing discarded-value analysis preserves calls, increments,
assignments and volatile evaluation without dereferencing the
hinted address. Optional controls must be integral constants;
out-of-range controls warn, and missing/extra arguments or bad
types diagnose. No external symbol or hardware instruction
is emitted for the hint, and no new persistent target storage.

The first candidate passes all eight original builtin-prefetch-6
cells but reveals 11 noopt failures in builtin-prefetch-4 and
11 optimized failures in the native invalid-address companion.
A scratch numbered-assertion trace identifies preinc_arg_idx
as the first noopt failure. gincdec returned the live register
object itself via r_preserve, so forming &p[++i] destructively
modified i after incrementing it. Use a distinct result value
as the normal VREG allocation does; keep the variable's AC
unchanged by subsequent expression arithmetic. The independent
new kcc-register-preinc-value-v59p regression fails the old
compiler on PDP-6 noopt (assertion 1) and passes 22/22 repaired
profiles. This is a general prefix-update repair, not a bypass.

Final companion replay: 121/132 runtime passes. The original
builtin-prefetch-4 now passes all 22 profiles. Four existing
observable companions pass 88/88, and the invalid-address
companion passes 11 noopt profiles but fails 11 opt profiles
at assertion 9 (integer data check after both address passes).
The scratch equivalent void-address-call variant reproduces
assertion 9 on PDP-6 opt; keep this independent optimizer
failure open. Newly enabled tests retain their real oracles.
The negative prefetch-control fixture passes 22/22 diagnostic
cells on candidate v1. Final guard replay 106/106 includes
the new prefix-update regression, alloca lifetime, packed
pointer callincdec and existing byte-pointer updates. Final
recheck of the eight original prefetch and four isprint cells
is 12/12 pass. No acceptance limits or original sources changed.

106/404 original link-failure cells are now confirmed PASS;
298 are not resolved (including the 22 explicit alloca cases).
This census does not include subsequent ICE-to-link-failure
reclassifications or a complete new matrix. GCC's four sprintf
cells emit MOVEI AC,sprintf, not a direct call; selection misses
that address materialization. The original requires floating
formatting, which must be tested separately from selection.
KCC's 50 sprintf cells require a genuine compiler-owned
formatter or explicitly chosen hosted environment. Do not
silently pull OS libc into bare-machine tests. The conversion
wrappers and missing func symbol need source/feature ownership
investigation, not invented providers.

Evidence: `$HOME/tmp/pdp10-link-triage-v59p.B2h7PTX3`;
exact symbol TSV cohorts, rh18-replay-v1, existing-replay-v1,
prefetch-replay-v1, prefetch-companions-v1/v2,
prefetch-negative-v1, guards-v2, before-preinc,
scratch-diagnose-v1 and recheck-v2. Installed compiler-only
`install-v2/local.tar.gz` SHA256:
`314fc4652a22739e223f2c1f2d81e9f755ad35cda3398252ca5b2b7c7bc2d773`.
Original sweep unchanged; no full-matrix, extended-mode,
self-host compiler or target timing/peak RAM claim.
