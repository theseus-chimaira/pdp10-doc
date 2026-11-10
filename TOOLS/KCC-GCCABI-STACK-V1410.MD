# KCC GCC ABI outgoing stack-argument lifetime v1410

Date: 2026-11-09.  Compiler correctness on native PDP-6; does not
change the DAIMOS kernel or installed toolchain.

## Root cause and correction

The direct GCC PDP-10 ABI call generator in KCC `ccgen2.c` accepted
one-word arguments in AC1..AC4 and pushed later arguments onto the
outgoing stack.  When an argument in an ABI register required a nested
call, compiler-generated virtual-register spills used the same stack.
An outgoing fifth or sixth argument could be mistaken for temporary
state and discarded or overwritten before the outer PUSHJ.  For
example, the old compiler corrupted a call with an object pointer,
name, computed hash, section and value, making the native DAS global
symbol refactor appear to have a symbol-table bug.

The v1410 compiler reserves the outgoing stack-only area before any
argument is evaluated.  Register-valued stack arguments are stored
before nested calls can overwrite their source ACs; other one-word
stack arguments are evaluated once into their reserved slots.
Offsets are computed using the current `stackoffset`, which includes
live VREG spills, rather than an assumed fixed offset from SP.  The
existing register-argument VREG snapshots and parallel ABI copies
remain in use.  The call site releases precisely the reserved number
of stack arguments.  There are no new heap allocations, wire-format
changes or runtime calling-convention changes.

An attempted fallback to the old general argument-block path did not
correct incoming parameter lifetime; simply reserving slots without
accounting for live spills also failed.  The final correction fixes
both causes and does not silently turn off the optimized ABI path.

## PDP-6 acceptance

The testkit fixture is `daimos-testkit/tests/validation/
kcc-gccabi-stack-nested-v1407`, extended for v1410.  It builds the
staged host compiler, independently compiles/assembles/links three
C89 executables against accepted CRT0 and syscall objects, stages
them into a complete isolated grouped D6FS root and executes them
on the native PDP-6 simulator.  The same source exercises five and
six arguments, a nested hash call and a pointer/unsigned-long
record-writing function.

Old optimized KCC exits with status 5 because an outgoing argument
is corrupt.  Corrected optimized KCC exits 7.  Corrected KCC built
using lowercase `-n` (the actual frontend no-optimization switch)
also exits 7; its assembly differs from optimized output.  The
strict testkit regression passes **3/3**.  A separate full v1401
rebuild of KCC, KCPP, KPARSE, KGEN and KOPT passes **16/16** native
phase, return-value and host-equality probes with the new compiler.

The DAS v1410 spill-backed symbol refactor subsequently passes the
v1405 **5/5** native global tests and v1402 **17/17** native DAS
compatibility tests, including a real runnable image.  All work
uses disposable test media; no bootstrap binaries were installed.

## Limits

The accepted correction applies to KCC's direct *one-word argument*
path.  Mixed two-word structures, hidden aggregate return pointers,
variadic named/unnamed slots and recursive call combinations need
additional targeted execution tests before claiming the entire GCC
ABI is exhaustively validated.  DAS native DOBJ1 relocation writer,
exact 96K memory profile and full self-hosting remain open.

KCORE size-only optimization remains the final KCORE development
stage, after functional and integration acceptance.
