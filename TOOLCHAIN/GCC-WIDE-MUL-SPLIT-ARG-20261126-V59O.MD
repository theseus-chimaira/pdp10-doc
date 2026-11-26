# GCC native wide multiplication and split argument repair (v59o)

GCC aa3a0505 and ceae113d; testkit 1c58eb61. Follow-up to
v59m/v59n. The ten remaining original priority cells of
920501-2.c were runtime assertion failures after the divider
optimization removed their timeouts.

Two distinct compiler defects were found. The PDP-6/KA10
general multiply converted its product to raw 36-bit limbs;
native division consumes high36:low35. KI10/KS10 selected
generic expansion instead. All native TARGET_71BIT general
multiplication now uses the same radix-2^35 MUL sequence.
It strips the redundant low sign bit, computes the low product
and extracts the full low36 of each cross-product using MUL.
Both high-word parities contribute bit70; restore the redundant
sign position. The raw72 pattern is unchanged. The first
experimental candidate omitted the high-high contribution and
regressed 16 signed arithmetic cells; it was rejected. The
second candidate includes the parity correction and passes all
16 signed arithmetic and all 16 powmod/GCD companion cells.
The sequence has 24 instructions and the existing scratch
pair constraint. No extra persistent target storage is needed.

Factorization still failed on that candidate. An independent
277-iteration host oracle produces factors 7, 73, 262657.
The scratch phase trace on PDP-6 O0 failed at iteration one.
In facts the outgoing split modulus high word came from 1(16),
but its low word was incorrectly read from 11(16), which holds
the low word of y. emit_push_insn in expr.c advanced the MEM
by GET_MODE_SIZE(mode) native words, not the number of words
in the stack portion. Correct scalar displacement is
(GET_MODE_SIZE(mode)/UNITS_PER_WORD - partial)*REAL_UNITS_PER_WORD.
The fixed call reads 2(16); existing stack/register ordering
and callee convention are unchanged. The supplied-size path
uses the already deducted stack portion in native units.

Final primary replay: 92/92 pass, including the original
920501-2.c 44/44, powmod/GCD companion 16/16, arithmetic
operators 16/16, and 36-bit multiplication companion 16/16.
The two new fixtures cover native physical product limbs and
memory/constant split arguments. Both reject the previous cc1
at their first assertion (PDP-6 O0); 88/88 repaired cells pass.
Guard replay: 108/120 pass; all 108 previous passes preserved,
including 920501-6.c 44/44, isqrt/prime 16/16, add overlap
44/44 and shifted accumulator 4/16. The other 12 shifted
accumulator cells retain existing arithmetic failures.
Total final selected 300 cells: 288 pass, 12 pre-existing
run_fail, no new failure. Neither original source nor limits
changed. All 440 original priority classifications have now
been individually addressed. Earlier ordinary diagnostic and
link-failure reclassifications are not semantic passes.

Evidence: `$HOME/tmp/gcc-wide-mul-v59o.GTzYKiWp`, replay-v1
and replay-v2 (experiments), replay-v3, regression-v1,
guards-v1, before-regression, phase-trace-v1, build-v3.log.
Final installed `install-v3/local.tar.gz` SHA256:
`16de9fb7bb1f58c35ee8d926a5eb9798627309b18ea80f986a839826c63f3732`.
Installed cc1 SHA256:
`4d26b5c11f9dfc32e87d7ec47995836a8c07607af8b805881331a47b1a769a7f`.
Original sweep is unchanged. The repair does not establish
consistent wide constant representation, general carry/sign-copy
semantics, complete argument ABI coverage, raw72/extended mode
correctness, or whole-matrix results. Target timing not measured.
