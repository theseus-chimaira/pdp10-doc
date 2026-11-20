# GCC PDP-10 freestanding buffered stdio, v1

## Scope

The GCC 3.2 pdp10-dec-none target has a real freestanding
stdio.h and a separate libp10stdio-v1.a archive. It does not
borrow KCC's sprintf variadic-argument assembly entry.
GCC's own built-in va_list ABI is used throughout.

Installed paths:

    $PDP10_PREFIX/pdp10-dec-none/include/stdio.h
    $PDP10_PREFIX/lib/gcc-lib/pdp10-dec-none/3.2/freestanding/libp10stdio-v1.a

The GCC source is
gcc-pdp10/gcc/config/pdp10/freestanding/p10stdio-format-v1.c.
The normal GCC build stages the header, archive, and assembled
formatter as p10stdio-format-v1.s. Compilation is C89 with
PDP-6-safe base instructions and -Os. No hidden heap or
writable static data is required.

## Available buffer interfaces

- sprintf and vsprintf: unbounded, caller-owned output storage.
- snprintf and vsnprintf: size-bounded, return would-be output
  length excluding the terminating NUL.
- Integer bases 10/8/16, signed/unsigned values, characters,
  strings, pointers, width, precision, flag characters and
  ordinary short/long integer qualifiers.
- Nine-bit target char and 36-bit target int/long.

This is a memory-only subset of stdio. Unsupported floating
formatting returns -1 instead of producing invented digits.
There is no FILE, stdin, stdout, stderr, fprintf, printf,
putchar, or file-stream API yet. Those must depend on actual
DAIMOS or other operating-system I/O facilities, not fake
test stubs. No stream interface is claimed as implemented.

## ABI and regression results

Real GCC target simulations pass on PDP-6, KA10, KI10 and
KS10, at both -O0 and -O2, for the comprehensive buffered
formatting regression. The unchanged historical GCC P1
920726-1.c original also PASSes with the actual formatter
and target strlen/strcmp across those CPU/opt profiles.
The production DARC archive is extracted with the genuine
DOBJ linker.

The normal pdp10-testkit compiler matrix selects the
installed GCC formatter for these four memory-formatting
entry points without altering KCC's library selection.

The original 920726-1.c ignores uninitialized returned
objects, so separate initialized scalar/struct-return
assertions remain necessary. The associated GCC variadic
scalar-return defect is not repaired by adding stdio.

The local source of truth for the formatter and its
limitations is:

    gcc-pdp10/docs/P10-STDIO-FREESTANDING-20261120-v1.md

Actual P1 source hashes and simulator report evidence are
verified in the testkit. This is GCC toolchain functionality,
not a DAIMOS kernel change.
