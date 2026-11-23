# GCC PDP-10 JRST Output: Cyclic Labels and Safe Folding

Date: 2026-11-23.

## Problem

The PDP-10 GCC 3.2 final-output optimization inlined a jump's
unconditional target instruction by recursively calling
`final_scan_insn`. Its existing check excluded only direct self-jumps.
A cyclic jump chain across multiple labels caused infinite recursion
and a host stack overflow. This was reproducible with the unmodified
upstream GCC `goto-1.c` torture source under `-Os`.

Simply breaking the recursive call was not sufficient: the target
operand printer (`follow_label`) also rewrote jumps and used
`maybe_remove_label_and_following_code` to delete labels. The first
candidate generated a `JRST` to an undefined label. DAS rejected it.

## Corrected invariants

The target now walks eligible unconditional jump-label chains and
uses Floyd's two-pointer cycle test before inlining. It requires
constant additional RAM and does not depend on an arbitrary maximum
jump count. Cyclic chains are emitted as ordinary `JRST` branches.

The same cycle check is applied before the operand printer rewrites
or removes a label. A label participating in, or leading into, a
cyclic chain remains addressable. Acyclic tail-jump optimizations
remain enabled.

## Verification

`pdp10-testkit/tests/codegen/upstream-gcc/compile/goto-1.c`
now compiles successfully for all 11 previously ICEing configurations.
The `-Os` assembly was also assembled with DAS across `base`, PDP-6,
KA10, KI10, KS10 and generic PDP-10. All assembled successfully with
resolved local labels. The full original 414-cell GCC ICE rerun
recorded 191 passing cells, 14 remaining GCC ICEs, 176 ordinary
compile failures and 33 runtime failures, without changed results
outside the 11 repaired `goto-1.c` cells.

Testkit checkpoint table:
`workdir/ice-retest-20261123-v8/results-v1.tsv`.
This is staged compiler acceptance; the full historical testkit and
installed-toolchain validation remain separate tasks.
