# GCC client linked against KCC-built DAIMOS libc.a (2026-11-23 V1828)

Status: PASS in two source-built PDP-6 DAIMOS guest runs, with the
final run covering virtual address bounds as well as overflow.

## Question and archive selection

The previous v1827 test established that GCC can recompile DAIMOS's
unchanged `userland/libc/stdlib.c` and execute its BRK-backed heap
allocator. The distinct question here is whether an application
compiled by GCC can call the **KCC-built** allocator from the existing
production `libc.a` without rebuilding that allocator using GCC.

`daimos-testkit/tests/validation/daimos-gcc-kcc-libc-heap-v1828.sh`
builds `DAIMOS/userland/libc/libc.a` from production sources using
KCC, compiles the C89 test client with GCC, and links a native DXR
using DAIMOS `crt0.s`, GCC's optional DAIMOS console archive and
independent v2 formatter, and the freshly KCC-built `libc.a`.
The compiler's standalone arena allocator is never linked.

The native linker map is required to contain `malloc`, `free`,
`calloc`, `realloc`, `dsys_brk`, `memcpy`, and `memset`. A negative
control omits the KCC libc archive and must fail the link; this
prevents accidental substitution with an implicit compiler libc.
The link and negative control passed.

## Actual guest verification

The fixture runs as `/CONFIG/GCCHEAP` inside a source-built PDP-6
DAIMOS image. The guest writes `GCC-HEAP-V1828-PASS` and the shell
reports `STATUS:0`. Both runs passed on 2026-11-23.

The client verifies the BRK position before and after allocation;
writable `malloc` storage; two live allocations; failure of oversized
`malloc` and overflowing `calloc`; failure of a `malloc` request
larger than the virtual user-word address space; failure of two
`realloc` growth requests without changing BRK or existing data;
successful `realloc` growth preserving original data and a neighboring
live allocation; `free`; zero-initialization by `calloc`; and
successful allocation after frees. The allocator and memory routines
are KCC-built and the syscall veneer is the genuine DAIMOS veneer.

## ABI conclusion and limits

**GCC and KCC interoperate for this tested DAIMOS allocator ABI.**
The allocator passes scalar sizes and pointers in the native calling
convention, and its heap metadata and word/character sizes work across
these compiler objects. GCC does not need a second production DAIMOS
heap allocator on the evidence of this test.

The result is selective. It does not establish that every KCC
`libc.a` entry point is ABI-compatible with GCC (particularly variadic
functions or callback-taking APIs), nor that GCC should automatically
link the KCC archive by default. The GCC console test deliberately
uses the GCC-native formatter, not KCC `printf`. Other CPU profiles,
asynchronous heap use, actual memory exhaustion, and all possible
invalid pointer cases are outside this regression.

No production compiler, kernel or libc source was modified.
