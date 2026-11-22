# KCC SUBBP CONSTANT OPERAND ICE (V1)

Date: 2026-11-22.
Status: compiler ICE fixed; upstream guest runtime NOT validated.

The upstream test `tests/todo/generic/20001203-2.c` previously
triggered `simsubbp: bad adrmode: 6` on KCC KS10.  The pseudo-op
was `P_SUBBP`, with `Ptype=026` (PTA_RCONST | PTF_IMM),
`Pvalue=0`, and `Pbsize=9`.  The optimizer legitimately folded
the right byte-pointer operand into a numeric constant.  The
`simsubbp` addressing-mode validation accepted register, indexed
memory, and pointer-constant forms but not numeric constants.

KCC `ccout.c` now accepts PTA_RCONST, leaving operand loading to
the existing P_MOVE output path.  No new register allocations,
runtime helpers, or fallback arithmetic were introduced.

Validation: the original source compiles for PDP-6, KA10, KI10,
and KS10 using the installed rebuilt KCC; the registered
`kcc-subbp-constant-v1` test passes.  Four testkit audits
(inventory, script ownership, raw p10run ownership, fixture
integrity) pass.

The full upstream semantic matrix was attempted but its guest
execution timed out on the initial profiles; the run was stopped.
It must NOT be reported as runtime PASS.  The source includes
pointer arithmetic involving zero and GCC-specific constructs.
Existing byte-pointer-difference source passed 18 compile-only
matrix cells, not runtime cells.

KCC commit: `e846b11`.
Testkit commit: `cffd85f`.

Next work: independently diagnose guest timeouts and develop
defined-behavior runtime coverage for the folded constant case,
where such a case is meaningful.
