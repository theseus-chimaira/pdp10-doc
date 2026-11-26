# GCC compact wide divider (v59m)

GCC 9687f281; testkit b85edff1 and 6311fa28. The original
November 25 sweep has 45 remaining GCC priority cells after
v59l: 35 prime-search and 10 factorization timeouts.

The previous libgcc1.s divider always performed 71 restoring
iterations, even for small operands. Its unsigned wrappers
saved eight ACs. The core now recognizes the divisor-one case,
uses IDIV when numerator high36 is zero and divisor fits low35,
and uses two DIV operations for other low35 divisors. The first
DIV divides high36 as a nonnegative pair; for divisors >= 2 its
quotient fits 35 bits. The second DIV divides remainder:low35,
with quotient strictly below 2^35. The general restoring path
reuses the numerator pair as quotient, propagates low35 carry
through the temporarily set low-word sign bit, and biases the
dying divisor high word for unsigned comparisons. The signed
wrappers preserve the sign flag without a second stack save.
The core's established high36:low35 representation, signed
sign-copy convention and deterministic direct zero-divisor
result are preserved. No new runtime ABI or helper dependency.

Unsigned calls save four ACs instead of eight; signed calls
save five rather than eight plus an inner sign word. The
installed DARC libgcc.a shrinks from 42024 to 41448 host bytes.
This is not a resident target size or timing measurement.

The registered direct assembly oracle generates independent
integer quotient/remainder expectations and checks both result
words, all AC5..AC16 and stack balance after each call. 300
calls per CPU pass on PDP-6, KA10, KI10 and KS10, 1200 total.
The initial direct run used the default prefix, and the final
direct-v2 replay uses compiler-only prefix isolation to exclude
generic OS libc. It also passes all 1200 calls. Boundary and
seeded full-width operands, signed cases and unsigned zero
divisors are included.

Selected GCC replay: 136 cells, 77 pass, 38 run_fail, 21 timeout.
All 63 original passes are preserved; 14 original prime-search
timeouts now pass. The other 21 prime-search timeouts remain.
All ten original factorization timeouts now terminate with
failed arithmetic assertions. These are not successful repairs.
The other original run_fail cells remain run_fail. The decimal
parser assembly exposes a distinct overlapping destination/source
problem in the 71-bit addition pattern; broader arithmetic
representation and carry semantics remain to be verified before
changing the backend. No source or acceptance limit was relaxed.

Evidence: `$HOME/tmp/gcc-wide-div-v59m.NlZXIgkZ`, replay-v1,
direct-v2.log, build.log, and libgcc.before.a. Installed through
local.tar.gz containing libgcc.a and the matching installed
libgcc1.s source. Archive SHA256:
`23ed7a192dd6d929793d08e059b5abe1088c5f760ac011affe0aec2f2bb60fae`.
Installed libgcc.a SHA256:
`dde44fe5ad2034d06a276d65ce6258355e9ee48712aa4a97618c204e767a1627`.
Original sweep is unchanged. No full-matrix pass, extended
addressing or raw72 helper validation is claimed.
