# GCC PDP-10 __builtin_abort link fix (2026-11-22 V1)

Status: fixed and validated in 16 guest configurations.

## Root cause

GCC 3.2's builtin registry did not include `__builtin_abort`.
The frontend consequently emitted a call to the literal external
symbol `__builtin_abort`, which the C runtime correctly did not
provide. This also prevented the compiler from knowing the call
was non-returning, retaining unreachable code after the call.

## Repair

`gcc/c-common.c` now declares `__builtin_abort` as an external
function with assembler name `abort` and marks its declaration
non-returning and side-effecting. This uses the existing C runtime
`abort` implementation, without adding a redundant symbol or a
test-only shim.

## Targeted verification

* GCC `cc1` rebuilt successfully.
* `pdp10-noreturn-reachability-v1.c`: 16/16 selected PDP-6, KA10,
  KI10, KS10 guest configurations PASS; previous result 16 LINK_FAIL.
* Generated assembly at O0/O1/O2/Os calls `abort` and does not
  reference `__builtin_abort`.
* No full compiler matrix was executed.

## Related runtime scope

GCC already provides `libp10stdio-v2.a` for memory formatting and
`libp10stdio-daimos-v1.a` for `printf`, `vprintf`, `putchar`, `puts`
through the DAIMOS `dsys_writechar` backend. The latter is not a
bare-machine freestanding console. The existing hosted `malloc`
dependencies still require a real allocator and heap backend.
