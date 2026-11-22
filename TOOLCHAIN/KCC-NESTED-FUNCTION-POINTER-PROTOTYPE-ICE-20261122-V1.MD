# KCC NESTED FUNCTION POINTER PROTOTYPE SCOPE ICE (V1)

Date: 2026-11-22.
Status: fixed, compile-tested on PDP-6, KA10, KI10 and KS10.

## Failure

The original GCC extension source
`tests/semantic/upstream/extensions/bcp-1.c` triggered
`[Internal error] dodecl: scope mismatch` at its
`int (*bad_t1[])(int x)` declaration.

The minimal reproducer is a file-scope array of function
pointers with a named parameter:
`int (*table[])(int named) = { 0 };`.
The equivalent unnamed parameter compiled.

## Root cause and repair

In `ccdecl.c`, `declarator()` allowed `fundecl()` to
retain a parameter symbol block whenever `paramok` was true.
That is correct for an actual function declarator, but wrong
for a function type nested beneath a pointer or array derivation.
The leaked prototype scope made the subsequent file-scope
declaration appear to be a block-scope declaration.

Only permit retained function-definition parameter scope
when the current declarator derivation `pp` is null.
A nested pointer/array function type uses `DF_INPROTO`,
which discards the temporary parameter symbol block.
Ordinary function definitions and function-returning-pointer
declarations remain supported.

## Verification

* Original `bcp-1.c` and reduced named-parameter forms
  compile on PDP-6, KA10, KI10 and KS10.
* Registered regression:
  `kcc-function-pointer-prototype-scope-v1`: PASS.
* Testkit inventory, script ownership, raw p10run ownership,
  and fixture integrity audits: PASS.
* No guest runtime semantic result is claimed for GCC-specific
  `__builtin_constant_p` behavior.

The separate `950613-1.c` float-to-long-long crash remains
open: its conversion produces an incorrect single-word
representation before a two-word DImode assignment. This
requires a semantic conversion fix, not merely a null guard.
