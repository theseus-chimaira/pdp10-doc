# KCC restoring wide divider register pressure (v59h)

KCC 68e1d19; testkit 2a52fad1, based on the intervening v59g
absolute-pointer correction. The November 25 sweep reported 85
gdimodedivmod scratch-allocation ICEs. Spilling expression values
does not release incoming ABI argument reservations: GDB showed
mask 016 (AC1..AC3) in signext.c, leaving only nine legal ACs.
The original signed quotient needed twelve (four inputs and
eight scratch); signed remainder needed ten.

Reuse the shifting numerator pair for quotient bits, and use
the sign bit of a shifted canonical low35 word as the carry
into high36. Bias the dying divisor high word once for unsigned
comparison. Subtract the biased word and toggle bit 35 of the
result to recover the original subtraction modulo 2^36.
Quotient and remainder share one emitted restoring loop; only
quotient inserts the result bit. The small IDIV path moves its
result low before clearing the overlapping high destination.
Signed operations now use five scratch ACs, unsigned four;
existing preserved scratch save/restore and ABI masks remain.
No additional runtime helper or persistent target storage.

The first experimental reduction removed the separate quotient
pair but still failed 22 signext cells. The final reduction
passed the entire 136-cell selected replay (118 pass, 18
compile_only, no failures). All 85 original ICEs disappear:
76 pass the registered codegen policy and nine compile_only
(shifts.c has no executable main). These are not 85 runtime
passes. Existing wide division/arithmetics semantic regressions
pass all 24 selected cells. The new pressure fixture reproduces
the baseline ICE and passes all 22 runtime cells, including
negative signed quotient/remainder and unsigned bit-70 cases.
No timeout, source oracle or policy was relaxed.

Evidence: `$HOME/tmp/kcc-div-pressure-v59h.QQ1ASExs`.
Final replay-v2 and regression-v3; installed bin/kcc-only
`install-v2/local.tar.gz` SHA256:
`2a7ef27f93f9c4635a0123737d654369b86ff56426418bbfc4bbd1faf9e780ef`.
Original sweep is unchanged. No whole-matrix, extended addressing,
self-host compiler, or measured target timing/size claim.
