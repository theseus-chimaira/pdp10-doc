# Freestanding PDP-10 String Search (2026-10-19, v1)

## Implemented ABI

The shared KCC/GCC PDP-6-compatible freestanding archive `libp10c-v2.a`
contains `strstr`, `strpbrk`, and `strspn`. Each function has a separate
archive member so the linker does not include unrelated string routines.
The 9-bit `char *` calling convention uses accumulator 1 for the first
argument/return value and accumulator 2 for the second argument. Byte
pointer operations use `LDB` and `IBP` on the PDP-6 instruction subset.

The three hand-written assembler routines require no stack or writable
library data. The implementation contains 28, 17, and 15 instruction words,
respectively; equivalent KCC-compiled C routines contain 38, 27, and 29.
This is a static instruction-word comparison, **not** a cycle measurement.
The algorithm retains linear constant-space set scans for `strpbrk` and
`strspn`, and a no-allocation first-byte-filtered candidate scan for
`strstr`, keeping typical searches compact and the resident RAM minimal.

Both KCC and GCC compiled and executed the same PDP-6 test vectors; tested
profiles are KCC optimized/unoptimized and GCC `-O0`. Both distribution
archives are byte-identical, with independent member extraction verified.
The regressions reside only in `pdp10-testkit`.

## Confirmed GCC initialized-data repair

GCC's GAS-style PDP-10 emitter previously left a partial 36-bit word
pending when completing a 9-bit initialized object. The pending word could
be emitted after the section switched from `.data` to `.text`, corrupting
data layout. The compiler now flushes subword initializer state at the
object boundary. A PDP-6 execution regression passed at GCC `-O0`, `-O1`,
`-O2`, and `-Os` using the rebuilt compiler.

The distinct GCC `-O1` optimization ICE involving address-of cleanup of
PDP-10 subword stack slots is **not resolved** by this data-layout fix.
Do not interpret the passing string-library API tests as proof of full
optimized GCC builtin transformations.
