# KCC wide floating conversion and fixture scope (v59r)

KCC d95b844 and 6bb6846; testkit 236e6bfe. The original November
25 sweep includes 24 KCC link failures in the two frozen GNU 71-bit
conversion stage wrappers. The included historical conversion.c
defines the invoked functions only inside #if __GNUC__. The
original evidence builder explicitly executes those wrappers with
GCC profiles only. Correct the 12 erroneous active KCC cells per
wrapper to unsupported (compiler-specific fixture, not missing
KCC language capability). Preserve the frozen original and wrapper
SHA256 values, and all applicable GCC profiles. Do not define
__GNUC__ in KCC or manufacture function providers. The raw libgcc
ABI fixture genuinely passes 12 KCC cells and remains enabled.
The host scope audit uses the actual runner's selection logic;
frozen hashes, correct scope and two invalid-admission controls pass.

The new kcc-wide-cast-oracle-v59r.c uses explicit high36:low35
limbs, volatile inputs, independent floating constants and raw
integer readback for signed casts in both directions. The original
KCC passes ten PDP-6-targeted cells but aborts compilation in 12
later-CPU cells with 'vrwiden: reg already wide'. Signed pair
conversion is not scalar FLTR or DFIX: gcastr must not widen a
pair again or narrow away its low word. Use the existing pair
marshal path with the installed libgcc __floatdisf, __floatdidf
and __fixdfdi on later targets; __fixsfdi was already shared.
The direct SF helper retains all 71 input bits until rounding.
The ABI-call prepass recognizes these new call sites so incoming
argument registers remain live across conversion. PDP-6 retains
its previous KCC software conversion path. No new runtime member,
ABI, or static target storage; later-CPU standalone links need
the corresponding real libgcc entries, not just KCC libc.

Candidate v2 eliminates the fatal errors but four unoptimized
KA10/pdp10 profiles fail the negative double-to-integer check.
The emitted DFN uses KA10's historical format rather than the
portable C D-format. Reuse the established scratch-free seven-
instruction 71-bit negate expansion on CPUs lacking DMOVN.
Candidate v3 passes the new oracle but regresses four older
KA10 scalar conversion cells because simdfix/simdsngl assumed a
single-word negate instruction. Candidate v4 corrects the skip
length and uses the existing full expansion and rounding path
when DMOVN is unavailable. All four regressions are recovered.

Final 134-cell KCC replay: 122 PASS, 12 pre-existing run_fail.
New signed wide cast oracle 22/22, full-word and narrow integer
conversion 44/44, signed 36-bit conversion and locator 32/32,
raw shared ABI 12/12, original conversion.c 2/8 and original
line companion 10/16. All original passing cells preserved.
The 80 GCC cells in the unchanged-compiler replay have 64 PASS
and 16 pre-existing run_fail; both original stage wrappers
16/16 each, raw ABI 16/16. Overall 214 applicable final cells,
186 PASS, 28 existing runtime failures. No oracle or limit
changed. This is not a full unsigned-wide conversion repair,
complete D-format verification, native self-host acceptance, or
target-size/timing measurement. The older PDP-6 helper uses
AC10 as sign scratch; audit its call-preservation contract and
direct SF rounding independently before claiming full coverage.

Original 404 link failures: 128 confirmed runtime PASS, 24
invalid KCC wrapper registrations corrected, 252 unresolved or
unverified. Do not label the excluded 24 cells runtime PASS.
Original sweep and historical evidence are unchanged.

Evidence: `$HOME/tmp/pdp10-conversion-scope-v59r.3AAK2H3b`;
replay-v1, native-v1/v2, final-v3/v4, scope-audit.log, kcc.before,
build-v4.log. Installed compiler-only install-v4/local.tar.gz
SHA256 `c03e1662ab840440d90faa8a0ac10024d7797eb5c8ad5789ecfbb8c7d91cca24`.
