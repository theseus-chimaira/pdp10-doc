# GCC calls into KCC-built DAIMOS variadic libc (V1838)

Date: 2026-11-23. Status: PASS at GCC `-O0` and `-O2` in a real
source-built PDP-6 DAIMOS guest. No additional ABI adapter is needed
for the tested external calling convention.

## Existing mixed-compiler variadic evidence

The existing `pdp10-testkit/misc/compat/test-kcc-gcc-varargs-matrix.sh`
and `test-wide-varargs-abi.sh` were rerun against the installed
toolchain. Both PASS. The first tests both GCC-caller/KCC-callee and
KCC-caller/GCC-callee directions for promoted narrow integers,
normalized 71-bit values, native word pointers, packed byte pointers,
a two-word aggregate, a promoted floating argument, and a trailing
word argument. The second independently validates wide variadic
stack traversal with a KCC callee and both caller compilers.

These tests establish agreement for those external caller/callee
representations. KCC's private `va_list` and GCC's native builtin
`va_list` are **not** interchangeable objects and must never be
passed across a compiler boundary without a separate explicit ABI.

## Real DAIMOS libc test

The new
`daimos-testkit/tests/validation/daimos-gcc-kcc-varargs-libc-v1838.sh`
builds the unchanged DAIMOS production `userland/libc/libc.a` with
KCC, compiles the same GCC C89 client at `-O0` and `-O2`, links both
against the KCC archive (not GCC's stdio archives), embeds their DXRs
as `/CONFIG/GCCVAR0` and `/CONFIG/GCCVAR2`, and boots PDP-6 DAIMOS.

Both guest probes PASS and exit with `STATUS:0`. The GCC client
calls the KCC library's assembly `sprintf()` entry and checks the
exact buffer `MIX:-17:XY:123` and its 14-character return length.
It then calls the KCC `printf()` entry with integer, string and octal
unnamed arguments. Real console output contains exactly the tested
marker `KCC-VARARGS-V1838: 23 OK 172`.

The final linker maps contain KCC `printf`, `vprintf`, `sprintf`,
`vsprintf`, and `__daimos_fround0_v33`. DLINK is allowed to relax
the genuine `dsys_writechar` veneer to a direct monitor UUO, so the
veneer symbol is not required to remain in the final map. A negative
link without KCC `libc.a` fails, excluding accidental formatter
substitution. Both separate guest programs passed on 2026-11-23.

## Boundaries

The guest explicitly tests integer, octal, string arguments and
formatted string output at two GCC optimization levels on PDP-6.
It does not establish all KCC floating-format variants, passing
compiler-private `va_list` values between GCC and KCC, all GCC/KCC
formatting extensions, or real DAIMOS execution on later CPU models.
No DAIMOS libc or kernel production source was changed.

The independent GCC frame-relative BLT repair and 32-cell mixed
structure-return suite are documented in
`GCC-KCC-STRUCT-RETURN-BLT-20261123-V1836.MD`.
