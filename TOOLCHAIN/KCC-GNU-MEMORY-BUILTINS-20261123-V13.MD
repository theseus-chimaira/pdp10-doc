# KCC GNU Builtin Memory Functions and Static Linkage

Status: implemented and target-execution verified on 2026-11-23.

## Failure

Five GCC upstream semantic tests used `__builtin_memcpy`,
`__builtin_memset` or `__builtin_strcmp`. KCC compiled those GNU
spellings as ordinary undeclared external function names, and
`dlink` reported missing `__builtin_*` symbols for 110 KCC profile
cells in the original priority rerun. The compiler already had
the corresponding real freestanding `memcpy`, `memset` and `strcmp`
implementations, so adding new library routines or test stubs was
not appropriate.

## Compiler implementation

`ccstmt.c` recognizes only the three specific GNU builtins in
function-call position when they have not been separately declared.
It constructs standard pointer, size and integer parameter types
with KCC's own type constructors, so KCC can apply its usual
function-argument conversions to the PDP-10 byte-pointer ABI.

The builtin symbol remains distinct from the ordinary C library
identifier in the parser. `ccout.c` translates external references
to the real library names when emitting assembler calls. A symbol
that the source explicitly defines as `__builtin_*` is not renamed.

This required correcting internal-linkage output in long-identifier
mode as well: a source function declared `static strcmp` must not
produce the same public `strcmp` label as the real library call.
KCC now prefixes internal symbol names with `$` in long-identifier
assembly, while legacy SIXBIT internal mapping remains unchanged.
The original source `921007-1.c` directly verifies this distinction.

No runtime helper, DAIMOS resident code, test stub, or new ABI
translation layer was introduced. Target code uses the existing
freestanding library implementations.

## Verification

All **110 original KCC link-failure cells** in the five unchanged
upstream sources now compile, link and execute successfully on the
target CPUs. The new C89 regression
`tests/semantic/regression/pdp10-kcc-gnu-memlib-shadow-v1.c`
passes **22 KCC profiles**. It checks return values, actual byte
contents, and separation from locally defined static libc names.

Sixteen adjacent internal-linkage compilation cells have identical
statuses between staged and installed KCC. They are compile-only
checks, not claims about target execution.

The final compiler was built separately at
`$TMPDIR/pdp10-kcc-memlib-build-20261123-v16/kcc`; the installed
compiler at `$PDP10_PREFIX/bin/kcc` was left unchanged.

Sources and exact results are recorded in
`pdp10-testkit/history/KCC-GNU-MEMLIB-UPSTREAM-RETEST-20261123-v13.md`.
Target result checkpoints are under
`pdp10-testkit/workdir/kcc-gnu-memlib-20261123-v19/` and
`pdp10-testkit/workdir/kcc-gnu-memlib-shadow-20261123-v20/`.
The native DAIMOS KCC phase build is outside the completed checks.

Together with the preceding verified GCC linker repairs (93 cells),
this confirms **203 of the original 560 link failures** now pass.
The other 357 require separate retesting and diagnosis, especially
hosted functions such as malloc, printf and sprintf and unrelated
compiler/ABI issues.
