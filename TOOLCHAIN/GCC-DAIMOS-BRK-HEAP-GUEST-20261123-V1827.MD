# GCC-compiled DAIMOS BRK allocator guest validation (2026-11-23 V1827)

Status: PASS in a source-built PDP-6 DAIMOS guest.

## Implementation selected

DAIMOS already provides the production allocator in
`userland/libc/stdlib.c`. It implements `malloc`, `free`, `calloc`,
`realloc`, `brk`, and `sbrk` using the real `dsys_brk` monitor veneer.
The free list is address ordered, permits splitting and coalescing,
and can extend an allocation at the current process break.

The GCC C89 freestanding compiler accepts the unchanged production
`stdlib.c`. The guest test compiles that source with GCC rather than
introducing another heap implementation. The existing DAIMOS
`crt0.s`, `syscall.s`, `memcpy.s` and `memset-v1.s` are linked along
with the GCC console formatter archives. The runtime is not stubbed.

## Test and result

`daimos-testkit/tests/validation/daimos-gcc-brk-heap-v1827.sh`
builds a GCC client and a GCC copy of the production DAIMOS allocator,
links a native DXR, installs it as `/CONFIG/GCCHEAP`, builds a PDP-6
DAIMOS boot image, and checks real guest output and exit status.

`daimos-gcc-brk-heap-v1827.c` checks the process break before and
after `malloc`, writes and preserves word arrays across a growing
`realloc` while a second allocation remains live, frees both blocks,
checks zero initialization from `calloc`, and allocates again after
freeing. The guest emits `GCC-HEAP-V1827-PASS` and exits zero.
All checks PASS on 2026-11-23.

## Scope and limitations

This proves the GCC-built production DAIMOS allocator can use the
real DAIMOS BRK syscall in a native process, including successful
`realloc` preservation and `calloc` zeroing. It does not claim that
the KCC-built `libc.a` is entirely ABI-interchangeable with GCC,
nor does it establish concurrent heap use, all out-of-memory paths,
or a default GCC libc installation. The independent GCC arena
allocator is still an optional freestanding facility, not the
DAIMOS process allocator.
