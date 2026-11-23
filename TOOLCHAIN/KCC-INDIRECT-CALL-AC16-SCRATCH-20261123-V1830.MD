# KCC indirect-call AC16 liveness repair (2026-11-23 V1830)

## Failure and instruction-level evidence

The GCC/KCC allocator interoperability validated in v1828 did not
establish callback compatibility. The independent DAIMOS testkit
`daimos-gcc-kcc-qsort-v1829` built its client with GCC and
`userland/libc/libc.a` with KCC, then executed the production `qsort`
using a GCC comparator over two-word records.

With the old installed KCC, the source-built PDP-6 DAIMOS guest
reported `UUO 777740000007 350000000071` and shell `STATUS:126`.
The binary linked normally. A minimal bare-metal PDP-6 instruction
trace localized the failure: at PC 002455, `PUSHJ 17,0(16)` was about
to call through AC16=0, even though AC13 still held the correct
function address (octal 001007). The next instruction fetch went
to address zero. This was not a DAIMOS monitor syscall failure.

KCC's `ccgen2.c:gcall()` had moved the indirect function target into
reserved AC16 before computing outgoing arguments. Its separate
`ccout.c:simadjbp()` implementation uses AC16 as a scratch accumulator
for variable 9-bit byte-pointer displacement on PDP-6. The qsort
comparator target was therefore overwritten before the indirect call.

## Compiler correction

The function designator is still evaluated exactly once and before
the argument expressions. KCC now pushes that computed target directly
to one temporary stack word, below any outgoing stack arguments.
After all argument computations and ABI register loading, it reloads
AC16 using the outgoing-stack-word offset and immediately emits the
indirect `PUSHJ`. It removes outgoing arguments and then the target
word after the call. The compiler's stackoffset tracking remains
balanced, including calls with a fifth stack-passed argument.

No separate trampoline, alternative sorting implementation, permanent
RAM allocation, or system call was introduced. The fix is contained
in `kcc/ccgen2.c`; `DAIMOS/userland/libc/stdlib.c` is unchanged.

## Independent acceptance

`daimos-testkit/tests/validation/daimos-kcc-indirect-arg-scratch-v1830.c`
exercises two- and five-argument indirect calls, with dynamic 9-bit
byte-pointer arithmetic during argument evaluation. The final fixture
also checks a nested call evaluated while the outer indirect target
is pending and a zero-argument indirect call. Its shell runner
compiles with KCC both without optimization and with `-O`, then runs
the real assembled code on PDP-6, KA10, KI10 and KS10.

All eight repaired-compiler cells PASS. Before the fix, the installed
old KCC failed the PDP-6 no-optimization cell by running into the
step timeout. With the corrected KCC, the original GCC-to-KCC qsort
guest passes in source-built PDP-6 DAIMOS: comparator calls execute,
records are sorted and preserved, the process prints its pass marker,
and the shell reports `STATUS:0`.

The `v1829` runner accepts `KCC_GUEST_COMPILER` to build the KCC libc
archive from an uninstalled compiler candidate without changing the
rest of the guest toolchain. The final minimized code-generation fix
also passed its eight-cell matrix. The unchanged production DAIMOS
libc archive builds successfully with the repaired installed KCC.

## Installed compiler and remaining scope

The corrected host KCC binary was installed to
`$PDP10_PREFIX/bin/kcc` after preserving the old binary in the
versioned `~/backups/kcc-before-indirect-target-v1830-20261123/`
directory. Installed compiler SHA256:

    bc2781c96189c04e893d714873f64e7ffa108a800a30d240dbcb1045eb6ba273

The installed KCC also passes all eight focused regression cells.
This establishes this specific nonvariadic indirect callback ABI;
it does not imply complete GCC/KCC ABI equivalence for variadic
functions, structure returns, or all libc interfaces.
