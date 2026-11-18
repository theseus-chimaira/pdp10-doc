# GCC PDP-10 Packed 9-bit Record Pointer Correction

Status: implemented, committed, and verified on 2026-11-18.

Compiler repository: `pdp10-gcc`, commit `c099c40`.
Test ownership: `pdp10-testkit`, original
`misc/compat/packed-char.c` and new
`misc/compat/gcc-packed-record-pointer-v1.c`.

## Problem

For an explicitly packed structure containing three 9-bit C
characters, `sizeof(struct packed3)` is 3 addressable characters
(27 bits). An array of such structures must use a 27-bit stride.
The GCC PDP-10 constant array-offset path previously inserted unused
padding to the next 36-bit word. Consequently, its emitted stores
used word-separated records while `struct packed3 *` pointer
arithmetic advanced by three 9-bit bytes.

Independent errors compounded the mismatch. Ordinary field expansion
discarded the native pointer's byte position and reconstructed
`POINT 9,(register),position` from the record's field offset.
That is not equivalent to loading or storing through a native PDP-10
byte pointer whose P/S selector already identifies a byte in a word.
At higher optimization levels, GCC also merged distinct fields'
loads before final output, so an assembler-output-only substitution
could not repair the lost accesses.

At `-O0`, the record pointer could be held in an automatic
stack slot. The address constructor then discarded the P/S bits
of the loaded pointer and reinstalled a default selector, even
though the pointer had already been advanced across a word boundary.

## Compiler correction

The PDP-10 GCC 3.2 backend now makes three coordinated changes:

1. `gcc/config/pdp10/pdp10.c`:
   `pdp10_md_array_offset` skips whole-word tail padding for
   explicitly packed record/union constant array subscripts,
   preserving the actual type-size bit stride. Ordinary scalar
   byte-array padding is unchanged.
2. `gcc/expr.c`: the packed 9-bit `COMPONENT_REF` read and
   assignment paths form exact native QImode byte-pointer MEMs
   before the normal RTL optimization passes. This preserves
   distinct byte addresses for loads and stores through packed
   record pointers; the constant-index field-read case is also
   lowered to the exact byte address.
3. `pdp10_dynamic_addr_expr_byte_pointer` accepts the encoded
   indirect pointer whether stored in a register or stack memory,
   and applies the field-byte displacement without clearing its
   P/S selector.

The special early paths are restricted to nonvolatile explicitly
packed 9-bit scalar record fields. No general volatile-field,
other-byte-size, or cross-compiler packed aggregate ABI claim follows
from this check. No runtime helper or DAIMOS resident object was
introduced.

## Verification

Two source-level regressions execute against the real target CPU:

- Original packed-char oracle: record/array size, contiguous fields
  and pointer increment to the next three-character record
- Expanded packed-pointer oracle: field writes and subsequent reads
  through a pointer, increment across 36-bit word boundaries, and
  constant-index verification of all six affected fields

Each passes for PDP-6, KA10, KI10 and KS10 with `-O0`,
`-O1`, `-O2` and `-Os` (32 successful target-executed cells).
The same 32 cells passed with the repaired GCC installed as
`$PDP10_PREFIX/lib/gcc-lib/pdp10-dec-none/3.2/cc1`.
Its SHA256 is
`d43034a6151e57787514ff6293b6d8da846de34a6192766d701505dab1eb6cc8`.
The prior compiler is saved as
`cc1.pre-packed-record-20261118-v1`.

Adjacent PDP-6 packed-bit-pointer, bit-pointer arithmetic and
char-word-boundary tests passed in six additional staged
compiler/runtime cases. The complete source/verification record,
including the failing pre-fix output-stage-only experiments, is in
`pdp10-testkit/docs/PDP10-GCC-PACKED-RECORD-BYTEPOINTER-20261118-v5.md`.

## Unrelated open compiler and runtime gates

The earlier GCC dynamic-sized `va_arg` compiler ICE was resolved
separately in `pdp10-gcc` `66f5a15`. Its original variadic
aggregate execution still fails; the compiler-generated copy must
be investigated against the real 9-bit byte-pointer `memcpy` ABI.
Historic `varargs.h`, hosted allocator/I/O dependencies, and full
compiler/DAIMOS matrix verification remain separate.
