# Daimos Data Abi

This document consolidates measured PDP-10 KCC/GCC interoperability findings originally maintained in pdp10-testkit. Each subsection preserves its source document and qualification; measurements are not generalized beyond their stated probes.

---

## Source: DAIMOS-WIDE-DATA-LAYOUT.md

# DAIMOS wide integer data layout v1

## Measured common layout

KCC and GCC in normalized 71-bit mode agree on the following C object sizes.
The values are C addressable-character units and are written in PDP-10 octal:

| Object | `sizeof` |
|---|---:|
| `int` / one machine word | `04` |
| `int71_t` / `uint71_t` | `010` |
| `raw72_t` / `uraw72_t` | `010` |
| two adjacent wide values | `020` |
| `struct { int; int71_t; int; }` | `020` |

Thus both wide representations occupy two consecutive 36-bit words and the
measured structures introduce no extra padding between word-aligned members.
The normalized 71-bit and flat raw-72 representations remain semantically and
ABI-distinct even though their storage sizes agree.

## `sizeof` rule

Portable tests must express word counts relative to `sizeof(int)`.  They must
not compare a two-word object directly with integer constant 2, because PDP-10
C reports sizes in addressable character units rather than 36-bit words.

## Public interfaces

Public persistent formats should continue to use explicit high/low words.
Compiler bit fields remain outside this frozen layout until direction,
container selection, and cross-word allocation are measured separately.

---

## Source: DAIMOS-POINTER-AND-AGGREGATE-ABI.md

# DAIMOS pointer and aggregate ABI v1

## Scope

This document records measured KCC/GCC interoperability for ordinary pointers,
PDP-10 byte pointers, and small aggregate return values.  It does not define
function pointers, segmented pointers, packed persistent formats, or aggregates
larger than four words.

## Ordinary word pointers

Pointers to 36-bit objects are one-word word addresses in both compilers.

- The first pointer argument and pointer return value use AC1.
- Pointer addition by `n` words is ordinary address addition.
- Loads and stores use ordinary word memory instructions.

Direct mixed-compiler calls using pointers to 36-bit objects are compatible.

## Byte pointers

Pointers to `char` and `short` objects are native PDP-10 byte pointers in both
compilers.

- Loads use `LDB`.
- Stores use `DPB`.
- Pointer advancement preserves the byte size and position fields.
- GCC currently emits `ADJBP` directly.
- KCC currently implements equivalent advancement through `%ADJBPH`.

The different instruction selection does not imply a representation mismatch.
Byte-pointer values can cross fixed-argument KCC/GCC interfaces directly.
Persistent and externally serialized pointer values remain forbidden.

## Small aggregate returns

Measured aggregates occupying one through four words are returned in successive
accumulators beginning at AC1:

| Aggregate size | Return accumulators |
|---:|---|
| 1 word | AC1 |
| 2 words | AC1:AC2 |
| 3 words | AC1:AC3 |
| 4 words | AC1:AC4 |

This includes a structure containing one normalized 71-bit integer: its high
word is returned in AC1 and its low word in AC2.

KCC may construct an aggregate in temporary stack storage internally, but the
externally visible return registers agree with GCC.  Direct mixed-compiler
calls are therefore supported for measured aggregates of at most four words.

## Unresolved cases

The following still require probes before direct interoperability is claimed:

- aggregates larger than four words;
- unions and bit fields;
- function pointers and pointer conversions among address classes;
- floating-point-containing aggregates;
- hidden return-buffer conventions;
- packed or explicitly aligned structures.

---

## Source: DAIMOS-EXTENDED-AGGREGATE-ABI.md

# DAIMOS extended aggregate and function-pointer ABI v1

This document records measured KCC/GCC behavior.  It does not generalize
beyond the probes in `tests/compat/extended-aggregate-abi.c`.

## Structures larger than four words

A structure occupying five 36-bit words is returned through caller-provided
storage.  AC1 contains the destination address.  Explicit scalar arguments
begin in AC2 and continue in AC3 and AC4, with remaining arguments passed in
the ordinary stack argument area.  Both compilers copy the complete result to
the destination supplied in AC1.

This extends the directly compatible aggregate-return subset beyond the
one-to-four-word AC return convention documented earlier.

## Function pointers

The measured function pointer is one 36-bit code address.  Both compilers pass
it in an ordinary accumulator and perform an indirect `PUSHJ 17` through that
address.  Fixed-signature function-pointer calls are directly compatible.

## Union layout

A union containing a normalized 71-bit integer and a two-word structure places
the high word first and the low word second in both compilers.  This confirms
layout compatibility for this measured union only.

## Bit fields

The measured unsigned sequence of 6-, 9-, and 18-bit fields occupies one
36-bit word with identical PDP-10 byte positions in both compilers.  This does
not yet freeze signed fields, zero-width fields, fields crossing a word
boundary, mixed base types, packing attributes, or implementation-specific
alignment controls.

## Remaining work

The following remain unproven:

- aggregates requiring more complicated hidden-return and argument mixtures;
- unions with differently aligned members;
- signed and cross-word bit fields;
- function-pointer representation under overlays or non-default code models;
- floating-point and packed aggregate interfaces.

---

## Source: DAIMOS-LAYOUT-AND-FLOAT-ABI.md

# DAIMOS Signed Bit-Field and Floating ABI v1

## Scope

This document records behavior measured by
`tests/compat/test-layout-float-abi.sh` under KCC and GCC in normalized
71-bit mode.

## Signed bit fields contained in one word

The tested sequence of signed 6-, 9-, and 18-bit fields occupies one
36-bit word in both compilers. Fields are allocated from the high end of
the word toward the low end in declaration order. Loads sign-extend the
declared field width before arithmetic.

This agreement is limited to fields whose total width does not exceed one
word and whose declared base type is signed `int`.

## Cross-word bit fields are incompatible

For three consecutive unsigned 20-bit fields, GCC packs 60 bits into two
36-bit words. KCC allocates three words, one word for each field. The same
C declaration therefore has different size, field offsets, argument
layout, and persistent representation.

Cross-word bit fields must not appear in DAIMOS public structures or
mixed-compiler interfaces. Use explicit words plus masks and shifts.
Compiler-specific structures may remain private to one translation unit.

## Floating-point fixed-call ABI

A `float` argument and result use AC1. A `double` argument and result use
AC1:AC2 with the high word in AC1. Function pointers with fixed `float` or
`double` signatures use the ordinary one-word code address and `PUSHJ 17`
indirect-call convention.

KCC may implement double arithmetic through runtime helpers while GCC may
emit native double-float instructions. That is a code-generation choice,
not an external ABI difference.

## Explicit packing and alignment

GNU `packed` and `aligned` attributes are not part of the verified common
source subset. KCC compatibility headers must not erase them silently.
Public DAIMOS interfaces requiring nondefault layout must use explicit word
or byte representations until both compilers implement the same named
facility.

## Still unresolved

- zero-width and unnamed bit fields;
- signed fields crossing a word boundary;
- non-`int` bit-field base types;
- floating-point variadic calls;
- floating-point structure and union layout;
- nondefault packing or alignment.
