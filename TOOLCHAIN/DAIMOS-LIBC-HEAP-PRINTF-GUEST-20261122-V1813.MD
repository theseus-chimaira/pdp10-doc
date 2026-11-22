# DAIMOS libc heap and printf guest validation (2026-11-22 V1813)

Status: PASS, actual PDP-6 DAIMOS guest runtime.

The existing DAIMOS `userland/libc/stdlib.c` already provides
`malloc`, `free`, `calloc`, `realloc`, `brk`, and `sbrk` backed by
`dsys_brk`. The existing `userland/libc/printf.c` provides
`printf` and `vprintf` using the `dsys_writechar` monitor veneer.
These are KCC-built DAIMOS runtime objects, distinct from the
explicit-arena freestanding GCC `libp10heap-arena-v1.a`.

## Regression

`daimos-testkit/tests/validation/daimos-libc-heap-printf-v1813.sh`
compiles the actual DAIMOS libc `stdlib.c`, `string.c`, and
`printf.c`, links the real syscall veneers and the required
`dformat-f0-v33.s` support, and builds a bootable DAIMOS PDP-6
image containing `/CONFIG/LIBC.TEST`. The guest test checks:

* malloc allocation and write/read integrity;
* calloc zero-initialization;
* realloc growth and preservation of prior data;
* free of both allocations;
* printf integer/string formatting and the returned character count;
* actual monitor console output and process exit status zero.

The guest probe requires `LIBC-HEAP-PRINTF-PASS` and `STATUS:0`.
The complete source-built image and SIMH guest probe PASS.
No default libc or DAIMOS kernel source change was necessary.
No full compiler matrix was rerun.

## Boundaries

This test verifies KCC/DAIMOS runtime behavior, not the GCC
freestanding test environment. The latter has no DAIMOS monitor
to service BRK or WRITECHAR and requires a separately selected
allocator and I/O backend. Float formatting, allocator fragmentation
stress, and cross-compiler ABI interoperability are not established
by this regression.
