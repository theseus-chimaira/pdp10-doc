# Freestanding PDP-10 String Search (2026-10-19, v1)

## Implemented ABI

The shared KCC/GCC PDP-6-compatible freestanding archive `libp10c-v2.a`
contains `strstr`, `strpbrk`, and `strspn`. Each function has a separate
archive member so the linker does not include unrelated string routines.
The 9-bit `char *` calling convention uses accumulator 1 for the first
argument/return value and accumulator 2 for the second argument. Byte
pointer operations use `LDB` and `IBP` on the PDP-6 instruction subset.

The three hand-written assembler routines require no stack or writable
library data. The implementation contains 28, 13, and 14 instruction words,
respectively; equivalent KCC-compiled C routines contain 38, 27, and 29.
This is a static instruction-word comparison, **not** a cycle measurement.
The algorithm retains linear constant-space set scans for `strpbrk` and
`strspn`, and a no-allocation first-byte-filtered candidate scan for
`strstr`, keeping typical searches compact and the resident RAM minimal.

Both KCC and GCC compiled and executed the same PDP-6 test vectors; tested
profiles are KCC optimized/unoptimized and GCC `-O0/-O1/-O2/-Os`. Both distribution
archives are byte-identical, with independent member extraction verified.
The regressions reside only in `pdp10-testkit`.

## Confirmed GCC initialized-data repair

GCC's GAS-style PDP-10 emitter previously left a partial 36-bit word
pending when completing a 9-bit initialized object. The pending word could
be emitted after the section switched from `.data` to `.text`, corrupting
data layout. The compiler now flushes subword initializer state at the
object boundary. A PDP-6 execution regression passed at GCC `-O0`, `-O1`,
`-O2`, and `-Os` using the rebuilt compiler.

## Confirmed GCC 9-bit stack ADDRESSOF repair

The distinct GCC `-O1` optimization ICE involving address-of cleanup of
PDP-10 subword stack slots is now repaired in `gcc/function.c`. The compiler
shares the existing byte-pointer construction with the late ADDRESSOF
cleanup, rather than attempting to stack-allocate a byte that is already
represented as `SUBREG(ZERO_EXTRACT(MEM))`. The unchanged ICE reproducer
compiles at `-O0/-O1/-O2/-Os`, and the full libc regression executes
successfully in SIMH PDP-6 for all those profiles. Independent compilation
also succeeds at `-O1` on KA10, KI10, KS10 and KL10 (execution unverified).

## Memory and speed measurements

Three independent assembler members now require 55 instruction words
in total versus 94 for the equivalent KCC-compiled C reference (39 fewer
words, 41.5%). The routines allocate no stack or writable library data.

For 16 searches per paired workload, including identical CRT and caller
overhead, the executed SIMH PDP-6 instruction steps were:

| Routine | ASM, KCC -n | KCC C -n | ASM, GCC -O2 | GCC C -O2 |
| --- | ---: | ---: | ---: | ---: |
| `strstr` | 71,243 | 198,011 | 59,626 | 65,770 |
| `strpbrk` | 54,315 | 132,123 | 42,538 | 42,618 |
| `strspn` | 21,771 | 42,939 | 12,954 | 20,714 |

The optimized GCC `strpbrk` C reference is essentially tied in executed
steps; static code size and stack use remain valuable differences.
These are simulator instruction counts, not wall-clock time, PDP-6 memory
cycle timing or AAP logical ticks. The user-space benchmarks live in
`pdp10-testkit`. GNU `__builtin_*` optimization regressions remain a
separate compiler-feature problem rather than a missing-libc dependency.
