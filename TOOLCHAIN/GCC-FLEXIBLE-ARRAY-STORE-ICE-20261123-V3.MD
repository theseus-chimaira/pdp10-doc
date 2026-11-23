# GCC PDP-10: Flexible-Array Member Store ICE

Date: 2026-11-23.

## Root cause

`gcc/expr.c:single_component_in_word_p` tries to establish that
an assignment to one structure component can be reduced to a
whole-word store. The scan of adjacent structure fields read
`DECL_SIZE_UNIT` unconditionally. A legal C99 flexible array
member such as `char data[]` has no fixed-size extent, so
`DECL_SIZE_UNIT` can be null. Passing that value to `tree_low_cst`
crashed GCC while generating RTL for an earlier, ordinary field.

The compiler now declines the single-component whole-word shortcut
if any scanned field lacks a known size or field offset. That
conservative result is required for correct whole-word store
selection: the shortcut is only permitted when overlap can be
ruled out. Normal store expansion remains responsible for actual
field accesses. The change does not add target runtime helpers.

## Verification

The existing `pdp10-testkit` source
`tests/codegen/kcc-ported/flexible-array-members-semantic.c`
was used unchanged. It exercises two non-packed flexible member
layouts and a packed declaration, including writes to trailing
members. Before the repair, 16 GCC profile cells crashed.

After rebuilding staged GCC, all 16 cells pass the testkit harness.
The full retest of the 414 originally failing GCC ICE cells changed
only those 16 states (`compiler_ice` to `pass`): remaining ICEs fell
from 79 to 63, and passes rose from 126 to 142. Other results remain
176 diagnosed compile failures and 33 guest runtime failures.

The checkpoint table is
`pdp10-testkit/workdir/ice-retest-20261123-v6/results-v1.tsv`;
the detailed testkit history is
`history/GCC-FLEXIBLE-ARRAY-ICE-RETEST-20261123-v3.md`.
This is staged compiler evidence, not validation of an installed
compiler update or a general ABI guarantee for flexible arrays.
