# GCC PDP-10 constant `memcmp` and logical 9-bit strings

Date: 2026-11-23.

GCC represents 9-bit string-constant characters using two host bytes
per target byte. The ordinary `c_getstr`/host `memcmp` constant-folding
path cannot compare these characters as unsigned PDP-10 bytes.

The target-specific path in `gcc/builtins.c:expand_builtin_memcmp`
uses the existing `pdp10_builtin_string_data_init` and
`pdp10_builtin_string_char` helpers to read logical 9-bit values.
It folds a comparison only when both literal bounds and the constant
comparison length are known. It returns a negative, zero, or positive
integer, as required by `memcmp`; otherwise the normal path remains.

The testkit regression
`tests/semantic/regression/pdp10-gcc-memcmp-9bit-constant-v1.c`
uses a deliberately undefined `link_error` and checks unsigned
9-bit ordering, embedded NULs, and differences after the first byte.
All 33 registered GCC semantic cells passed with the staged compiler.
Results are in `workdir/memcmp-9bit-20261123-v1/results-v1.tsv`.

The unchanged GCC upstream source
`tests/todo/generic/generic/string-opt-16.c` was also rerun for all
44 registered GCC profiles. All 44 link and execute successfully,
including the **33 configurations previously failing** with an
undefined `link_error`. The symbol was a deliberate optimization
oracle: defining a test stub would hide the compiler defect. Its
33 original failure keys match the successfully retested cells.
Combined upstream and new-regression verification is recorded in
`pdp10-testkit/workdir/gcc-memcmp-9bit-20261123-v10/results-v1.tsv`
and `pdp10-testkit/history/GCC-MEMCMP-9BIT-LINK-REPAIR-20261123-v1.md`.

The 414-cell original GCC ICE baseline was rerun with the new
compiler and has identical outcomes: 205 passes, 176 ordinary
compile failures, 33 runtime failures and zero compiler ICEs.
See `pdp10-testkit/workdir/ice-retest-20261123-v11/results-v1.tsv`.

The test does not validate general runtime `memcmp`, and this compiler
build has not been installed in `$PDP10_PREFIX`.
