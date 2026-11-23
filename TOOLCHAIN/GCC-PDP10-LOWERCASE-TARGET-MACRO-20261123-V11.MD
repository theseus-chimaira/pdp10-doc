# GCC PDP-10: Lowercase Target Macro and Noreturn Linkage

Status: fixed in GCC and verified with staged `pdp10-dec-none`
compiler on 2026-11-23.

## Problem

The PDP-10 GCC driver defines `__PDP10__` and `__PDP10`. Historically,
software also checks the reserved lowercase form `__pdp10__`. GCC's
NetBSD PDP-10 target defines that lowercase spelling, but the generic
PDP-10 and TOPS-20 driver specifications omitted it.

The upstream source `tests/todo/generic/builtin-noret-1.c` excludes
unavailable `_eexit` handling under `#ifndef __pdp10__`. On generic
PDP-10, the missing definition caused compilation of the unsupported
branch and an undefined `_eexit` linker symbol. The original targeted
rerun recorded 16 GCC `link_fail` cells, one for every registered
native PDP-6, KA10, KI10 and KS10 optimization profile.

## Corrective change

`gcc/config/pdp10/pdp10.h` and `gcc/config/pdp10/tops20.h` now
include `-D__pdp10__` in their `CPP_PREDEFINES`, preserving all
existing uppercase and other target macros. The NetBSD variant
already included the lowercase spelling and needs no change.

This repairs target identification, not linkage: no fake `_eexit`
symbol or freestanding runtime shim was introduced. The target
code generator and assembler instructions are unchanged.

## Verification

The rebuilt staged GCC driver returns both macros as 1 under
`-dM -E`. The new registered test
`tests/semantic/regression/pdp10-lowercase-target-macro-v1.c`
enforces their presence for 44 GCC profiles; all 44 execute and
pass. The original `builtin-noret-1.c` was rerun without edits;
its 16 previously failing cells now link and execute successfully.

The full original 414-cell GCC ICE baseline still has zero ICEs and
identical results to the prior run (205 pass, 176 compile_fail,
33 run_fail). All 414 recorded keys were compared.

Evidence:
`pdp10-testkit/history/GCC-PDP10-LOWERCASE-MACRO-20261123-v11.md`,
`pdp10-testkit/workdir/gcc-lowercase-macro-20261123-v11/results-v1.tsv`,
and
`pdp10-testkit/workdir/ice-retest-macrodriver-20261123-v12/results-v1.tsv`.

The staged executable was `pdp10-dec-none` rather than a separately
built TOPS-20 toolchain; the TOPS-20 source change is structurally
parallel and remains unverified on that target. The installed GCC
in `$PDP10_PREFIX` was not replaced.
