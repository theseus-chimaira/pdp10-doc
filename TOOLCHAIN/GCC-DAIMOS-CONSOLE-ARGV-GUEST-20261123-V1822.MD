# GCC/DAIMOS console and argument guest validation (2026-11-23 V1822)

Status: PASS on a booted PDP-6 DAIMOS guest and in eight independent
CPU/optimization ABI simulations.

## Real DAIMOS console integration

`daimos-testkit/tests/validation/daimos-gcc-stdio-console-guest-v1.sh`
builds a GCC C89 program, links the installed GCC formatter and optional
DAIMOS console archives against the actual `userland/libc/syscall.s`
veneer and DAIMOS `crt0.s`, and boots a fresh PDP-6 image with the
program installed under `/CONFIG/GCCSTDIO`. The guest verifies
`sprintf`, `printf`, `vprintf`, `putchar`, `puts`, the actual WRITECHAR
monitor output and shell process status zero. PASS on 2026-11-23.

The existing guest script had omitted the installed freestanding header
include path. This caused `p10stdio-daimos-v1.h` lookup to fail before
the test could execute. The explicit header include path fixes the
test harness, not production GCC or DAIMOS code.

## Variadic ABI matrix

`daimos-testkit/tests/validation/daimos-gcc-stdio-console-abi-v1.sh`
tests the GCC `printf` and `vprintf` ABI through a deliberately
test-owned recording sink. The linked production archives must be
extracted, and output-error behavior must terminate immediately.
After correcting the same installed-header include path in both the
direct compiler invocation and P10RUN, all PDP-6, KA10, KI10 and KS10
`-O0`/`-O2` cells passed (8/8). This test does **not** exercise
the actual DAIMOS monitor; the separate guest does.

## Process entry and arguments

`daimos-testkit/tests/validation/daimos-gcc-startup-argv-v1822.sh`
builds and boots another GCC native program. Running
`/CONFIG/GCCARGV ARG1 ARG2` verified the native `main` register
entry: `argc == 3`, a valid `argv` vector, and two exactly matching
length-prefixed SIXBIT arguments. It verified real GCC `printf`/`puts`
output and process exit status zero. PASS on 2026-11-23.

DAIMOS command-line arguments are **not** hosted-C `char *` strings.
Its process ABI supplies pointers to length-prefixed 36-bit SIXBIT
word arrays. A GCC client that needs command-line text must interpret
that representation or call a separately defined conversion routine.
The guest probes intentionally decode SIXBIT without assuming a
hosted `char **argv` layout.

No GCC compiler, DAIMOS kernel, or runtime library changes were needed.
The tests do not establish complete GCC/KCC object ABI equivalence,
general environment-variable handling, or GCC `malloc` using DAIMOS
BRK. The standalone GCC arena allocator remains a separate opt-in
facility, not a replacement for DAIMOS process `malloc`.
