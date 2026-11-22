# GCC PDP-10 freestanding arena allocator (2026-11-22 V1)

Status: implemented, installed, and validated on PDP-6/KA10/KI10/KS10.

## Architecture

`gcc/config/pdp10/freestanding/p10heap-arena-v1.c` implements C89
`malloc`, `free`, `calloc`, and `realloc` using a contiguous region
provided explicitly through `p10heap_init_v1(void *arena, size_t bytes)`.
The caller owns the region and must align it suitably for long double
and pointer objects. No system call, default static heap, or OS
dependency is embedded in the allocator. Reinitialization invalidates
outstanding allocations. The allocator is single-threaded.

Each block stores a size in alignment units and an available flag.
Allocation scans for a fitting free block and splits when useful;
free merges adjacent free blocks. Realloc can resize in place, grow
into an adjacent free block, or copy and free on relocation. Allocation
failure leaves the old realloc allocation intact. Calloc checks for
multiplication overflow and clears all requested bytes.

## Distribution

The build installs these independent versioned artifacts:

* `freestanding/libp10heap-arena-v1.a`
* `freestanding/p10heap-arena-v1.s`
* `pdp10-dec-none/include/p10heap-arena-v1.h`

The archive is deliberately **not** inserted into the default
`libp10c-v2.a`: a freestanding program must explicitly select its
heap backend and initialize the arena before allocating. This avoids
an accidental, unusable default heap and preserves the historical
compiler-stdlib evidence archive and its expected linker failures.
An automatic DAIMOS heap backend remains separate future work.

## Targeted verification

The registered `gcc-p10heap-arena-v1` test checks eight real guest
configurations: PDP-6, KA10, KI10 and KS10 at GCC O0 and O2. It
exercises allocation, block reuse, adjacent-block merging, data
preservation during realloc, calloc zeroing, overflow rejection,
zero-size realloc and large allocation after freeing. All eight
guest executions PASS. An independent real dlink archive extraction
check resolves all five exported symbols. All four testkit audits
PASS. No full compiler matrix was run.
