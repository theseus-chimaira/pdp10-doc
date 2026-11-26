# GCC wide addition/subtraction register overlap (v59n)

GCC ad48f37b; testkit 04fc7e38. Follow-up to the compact divider
v59m. The earlier decimal parser emitted a destructive pair copy
into a still-live operand, followed by addition of the overwritten
operand to itself. This is a machine-description allocation defect,
not an expensive division or an insufficient step limit.

The adddi3_71_no_dadd alternatives now explicitly tie the in-place
destination to operand 1 and use early-clobber for separate pairs.
The register/memory alternative also marks the in-place destination
early-clobber, as the low-word write precedes consumption of the
other high operand. Immediate alternatives retain their existing
instruction sequences; lengths distinguish tied and separate pairs.
Subtraction protects its in-place destination in the same way, and
the expansion materializes non-offsettable memory operands before
the two-word operation. No new helper, ABI, or scratch requirement.
The underlying carry algorithm is unchanged; this is not a general
full-width arithmetic representation correction.

The exact original 920501-6.c now passes all 44 GCC profiles,
recovering 21 remaining timeouts and nine other original runtime
failures in addition to the 14 timeouts resolved by v59m. The
136-cell replay passes 117, with 19 existing runtime failures:
920501-2.c 33/44, wide powmod/GCD 8/16, wide square-root/prime
16/16 and wide arithmetic operators 16/16. All v59m green cells
remain green. The ten original factorization timeout cells still
fail their oracle. The original priority timeout classification is
now exhausted, but those ten failures are not resolved.

The new gcc-wide-add-overlap-v59n.c fixture checks decimal parsing,
separate operands, live source values and immediate add/subtract.
The old cc1 fails the first parser assertion under PDP-6 O0; the
new cc1 passes 44/44. Another 88 guard cells include 72 passing
add/subtract codegen checks and 4/16 shifted-accumulator runtime
passes; the other 12 retain their original runtime failures.
Across all three replays: 268 selected, 237 pass, 31 existing
runtime failures. No acceptance limit or original source changed.

Evidence: `$HOME/tmp/gcc-wide-add-v59n.PCenBTMW`, replay-v1,
regression-v1, guards-v1, before-regression, build.log. The previous
cc1 is retained. The compiler-only local.tar.gz installed at the
operational PDP10_PREFIX has SHA256:
`b67874cd9791a864bc84fa91bd37e6e168b122babd99ec4e0fdb6f5ee9224e49`.
No complete matrix, raw72, extended addressing, target timing or
full-width carry/sign-copy correctness claim is made. The remaining
powmod failure and shifted-accumulator defects require further
arithmetic-lowering investigation. Original sweep unchanged.
