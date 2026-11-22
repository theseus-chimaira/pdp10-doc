# KCC NESTED COMPOUND INITIALIZER ICE REPAIR (V1)

Date: 2026-11-22.
Status: fixed, installed, regression-tested, committed and pushed.

## ROOT CAUSE

The upstream GCC compile testcase
`tests/codegen/upstream-gcc/compile/init-1.c` declares an
incomplete outer array initialized by a struct containing a pointer
to a file-scope C99 compound literal:

`T1 a[] = { { "", ((char *)&((T3) {1, (char *) 1})) } };`

The initializer parser `pizer` in KCC's `ccdecl.c` uses the global
`izsym` and `izautof` variables to identify the object being initialized
and whether it has automatic storage duration.  Parsing the compound
literal recursively invoked `pizer`, overwriting both variables.
When the outer unbounded array initializer subsequently inferred its
bound, it updated the nested compound literal's symbol instead of
the outer array.  The outer array retained `Tsize == 0`, causing the
static initializer generator to report the internal error
`gizlist: bad size: 0`.

The repair saves the two initializer-context variables on entry to
`pizer`, establishes the nested object's context, and restores the
outer context after the recursive parse.  It adds no persistent
compiler structures and does not suppress the generator's size check.

## VALIDATION

* The original upstream `init-1.c` compiles with correctly emitted
  outer array and compound literal storage.
* A new semantic runtime regression
  `tests/semantic/regression/kcc_nested_compound_initializer_v1.c`
  checks an array of two records, its inferred bound, string keys,
  and both separately initialized compound-literal payloads.
* Original and new regression together: 40/40 selected KCC cells PASS.
* Six existing initializer regressions: 56/56 KCC cells PASS.
* Testkit inventory, script ownership, raw simulator ownership, and
  fixture-integrity audits: zero errors.

KCC commit: `934437a`.
PDP10 testkit commit: `337ea50`.

This is a confirmed repair for this ICE cluster only.  Other compiler
internal errors, linker failures, and guest timeouts remain to be
investigated independently.
