# KCC static packed pointer coordinates and exact32 word access (v59f)

KCC 6423263; testkit 473143dc. All 22 KCC profiles of the
November 25 sweep's `gcc-exact32-pointer-representations-v51a.c`
crashed in gizptr with SIGFPE. The member layout's P=077,S=0
marker is a C-byte displacement, not a hardware byte pointer.
The constant evaluator was dividing by its zero S field.

Decode packed member coordinates before ordinary P/S fields.
Preserve the pointer representation in &*p and during assignment
conversion, using the existing S=1 logical form when a packed
array's native 9-bit address enters the polymorphic exact-width
pointer ABI. Constant address arithmetic uses the corresponding
element stride. Native exact32 slots retain word addresses;
polymorphic access and pointer arithmetic distinguish the word
case from native byte pointers. The original static relocation
is now `POINT 1,$x+1,9`.

The native-word load path releases its address VREG after use.
Increment/decrement through polymorphic pointers must retain the
VREG returned by the store helper, which consumes its input.
Earlier experimental candidates produced runtime failures and
internal register warnings; neither is present in the final
66-cell replay. No new pointer format or runtime helper is added.

Verification: replay-v4 ran 220 cells, with 209 PASS and 11
unoptimized original-test runtime failures (exit 044 octal,
native exact32 increment/wraparound assertion). After fixing
the discarded store result, replay-v5 passed 66/66: original
v51a, new kcc-static-packed-pointer-v59f, and existing packed
pointer increment/decrement regression, 22 profiles each. All
154 remaining v4 passing cells produce byte-identical normalized
assembly with the final installed compiler (asm-compare-v5).
The new fixture reproduces the original SIGFPE on the baseline.
The final replay has no internal error diagnostics. No full
matrix, native compiler, alternate assembler dialect, or extended
addressing acceptance is claimed. No target size measurements.

Evidence: `$HOME/tmp/kcc-static-packed-v59f.aItDYIjK`.
Final bin/kcc SHA256:
`1d2512c5fe5c2e0dccd7c53b4e79efd240dba7a5885a2f9fccd4b813a7324e5f`.
Installed incremental `install-v5/local.tar.gz` SHA256:
`72672f731b6deabc4f91dcf60d0d1b31db0b1f2f7b0600d986bbb9ba06516346`.
Original full-sweep results are untouched. No DAIMOS source change.
