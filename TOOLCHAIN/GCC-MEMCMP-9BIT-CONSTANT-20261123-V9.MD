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

The test does not validate general runtime `memcmp`, and this compiler
build has not been installed in `$PDP10_PREFIX`.
