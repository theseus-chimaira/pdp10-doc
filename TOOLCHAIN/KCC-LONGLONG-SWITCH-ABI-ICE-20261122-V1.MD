# KCC LONG LONG SWITCH ABI ANALYSIS ICE (V1)

Date: 2026-11-22.
Status: fixed and pushed; compilation regression validated.

## Root cause

Upstream `tests/todo/generic/930111-1.c` triggered
`sizetype: null type`.  A reduced source with one `switch`
on a `long long` argument reproduced the error, while a
corresponding `int` switch and a regular `long long` call
compiled.

The DImode ABI peer analysis in `ccdecl.c`,
`fn_abi_direct_peer_simple`, traversed a statement/control
node `Q_SWITCH` with `Ntype == NULL`.  Its default branch
called `sizetype(n->Ntype)` without a null check.  This was
not a defect in the switch dispatcher or long long arithmetic.

The fix rejects untyped nodes conservatively from the
scalar-peer optimization.  No runtime code or ABI layout
changes are introduced.

## Validation

* Rebuilt and installed KCC.
* Original upstream source and reduced long long switch
  compiled for PDP-6, KA10, KI10 and KS10.
* Additional one-case switch, int switch and ordinary long
  long call compiled on KS10.
* Registered test `kcc-switch-longlong-abi-v1`: PASS.
* Four testkit audits: PASS, zero issues.

No full guest execution was performed in this fix; the
result is a confirmed compiler ICE removal, not a claim
of runtime semantic coverage.

KCC commit: `dc7a9d0`.
Testkit commit: `ce70ed1`.
