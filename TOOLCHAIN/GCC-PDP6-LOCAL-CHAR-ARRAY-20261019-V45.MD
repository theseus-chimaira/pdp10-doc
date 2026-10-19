# GCC PDP-6 Local 9-bit Character Array Boundary Correction (2026-10-19)

## Fault and root cause

The GCC PDP-10 backend generated incorrect assembly for optimized reads of
some 9-bit characters from a stack-local array spanning 36-bit word boundaries.
At `-O1`, `-O2`, and `-Os`, an ordinary C comparison of `line[4]` to `?`
could instead read the second byte of the same 36-bit word, erroneously
reusing the `line[5]` byte position. `-O0` generated the expected access.

The compiler had already lowered an access to the first byte of the selected
word as an `SImode MEM` plus an RTL logical right shift of 27 bits.
`pdp10_output_lshiftrt_memory` then incorrectly interpreted the associated
`MEM_OFFSET` as a nine-bit *byte* offset rather than the word displacement
already present in the frame-relative memory operand. Thus GCC emitted an
incorrect `POINT 9,...,17` instead of selecting the first byte.

The correction in `gcc-pdp10/gcc/config/pdp10/pdp10.c` (commit `df27247`)
keeps the selected word address for word-aligned, frame-relative `SImode`
loads with an RTL shift of 27, and uses bit position zero. It does not add
runtime data, enlarge DAIMOS resident structures, or substitute a different
algorithm for reading characters.

## Verification

The direct target-executed C99 test in
`daimos-testkit/tests/validation/pdp10-gcc-char-word-boundary-v45.{c,sh}`
and its independent assembly writer checks characters 3 through 8 across
36-bit word boundaries. Before the fix, it passed `-O0` and failed `-O1`
on the PDP-6 execution engine. With the rebuilt compiler it passed `-O0`,
`-O1`, `-O2`, and `-Os`. The existing multidimensional-character test
also passed all three tested levels (`-O0`, `-O1`, `-O2`). The regression
is included in the testkit `target` target (commit `e24488e`).

Most importantly, a fresh 256K disk-root DAIMOS build using that compiler
passed **all 115** DSH conformance probes, including the previously failing
`HELP ECHO` and `HELP DAS` compact-help probes. The unmodified installed
compiler and the DAIMOS production sources were not changed for this run.
`DSH_IMAGE_LIMIT=037000` remained a diagnostic expanded image ceiling;
no 32K acceptance claim follows from these checks.

## Separate open defects

The GCC scheduler multiprocess group still fails the timer-preemption marker
ordering test with the corrected compiler. This is independent of the
char-array repair and requires investigating the clock/scheduler behavior
before changing its timing assertions. Other disk-suite failures (notably
live MonitorFS, DECtape/TSFS mounting, and D6FS recovery) remain separate
gates. The complete 30-group regression matrix has not been rerun after
the compiler correction.
