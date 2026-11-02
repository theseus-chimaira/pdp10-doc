# Daimos Integers And Conversions

This document consolidates measured PDP-10 KCC/GCC interoperability findings originally maintained in pdp10-testkit. Each subsection preserves its source document and qualification; measurements are not generalized beyond their stated probes.

---

## Source: DAIMOS-INT71-ABI.md

# DAIMOS signed 71-bit integer ABI v1

## Scope

This document freezes the source-level and object representation used by the
PDP-10 GCC `long long` mode and selected for DAIMOS compatibility.  It is the
reference for KCC/GCC comparative tests.  Raw 72-bit arithmetic is a separate
extension and is not described here.

## C types

The canonical signed type is `int71_t`; the canonical unsigned type is
`uint71_t`.  In both GCC and KCC test headers these are aliases for `long long`
and `unsigned long long`.  Ordinary `int` and `long` remain one 36-bit word.

The historical testkit names `Dint` and `uDint` denote the same 71-bit model.
The old `int72` and `uint72` names are retained temporarily as compatibility
aliases, but they do not denote raw 72-bit arithmetic and must not be used in
new tests.

## Representation

An object occupies two consecutive 36-bit words in high-word, low-word order.
The high word contains the sign and the upper 35 value bits.  The low word
contains 35 further value bits.  Its sign-position bit is a normalization bit,
not an independent 72nd value bit.

For a normalized pair `(hi, lo)`, bit 0 of the low PDP-10 word is a copy of the
sign bit of `hi`.  The mathematical value therefore has 71 significant bits,
with signed range `-2^70` through `2^70 - 1`.

The in-memory pair and an accumulator pair use the same word order.  A value
returned in accumulators starts with the high word in the lower-numbered
accumulator.

## Arithmetic contract

Signed addition, subtraction, negation, multiplication, division, remainder,
and shifts operate on the normalized 71-bit value.  Results are normalized
before becoming observable through memory, calls, or comparisons.

Unsigned operations use the corresponding normalized two-word representation.
The exact unsigned range and overflow wording remain provisional until the
GCC and KCC boundary probes are complete; tests must not assume a flat modulo
`2^72` integer.

Shift counts requiring explicit compatibility coverage are 0, 1, 35, 36, 70,
and 71.  Raw 72-bit shifts and arithmetic must use separately named types or
intrinsics.

## ABI requirements

1. Objects occupy two words, high word first.
2. Register pairs use the lower-numbered accumulator for the high word.
3. Arguments, returns, helper calls, and memory operations preserve the same
   normalized representation.
4. Public interfaces must use `int71_t` or `uint71_t`, not misleading `int72`
   names.
5. A compiler configuration using 36-bit `long` for `Dint` is semantically
   incompatible and must be rejected by comparative-test metadata.

## Open items

The following are deliberately not frozen by version 1:

- the final unsigned overflow specification;
- complete varargs placement;
- all helper symbol names and clobber sets;
- structure-return interactions;
- raw 72-bit extension spelling and ABI.

---

## Source: DAIMOS-RAW72-ABI.md

# DAIMOS raw 72-bit extension ABI v1

`raw72_t` and `uraw72_t` denote a flat pair of 36-bit words.  All 72 bits are
value bits for unsigned operations; signed operations use flat 72-bit two's
complement.  The high word is stored first.

This representation is distinct from normalized `int71_t`/`uint71_t` and uses
separately named helpers (`raw72_*`).  Public mixed-compiler interfaces must
not pass a raw 72-bit value as `long long`.

GCC programs compiled wholly with `-mlong-long-72bit` may use `int72_t` and
`uint72_t` as native `long long` aliases.  In 71-bit GCC mode and in KCC,
those names resolve to the two-word aggregate adapter.  DAIMOS remains built
with `-mlong-long-71bit`.

## Complete adapter operation set

The portable aggregate ABI provides modulo-2^72 add, subtract, negate,
bitwise operations, logical shifts, multiplication, and unsigned comparison.
`raw72_udivmod` and `raw72_sdivmod` return zero on success and `-1` for a zero
divisor.  Signed quotient truncates toward zero and signed remainder has the
sign of the dividend.

`raw72_from_int71_words` sign-extends a normalized 71-bit word pair into flat
72-bit two's complement.  `raw72_to_int71_words` returns `-1` when the flat
value is outside the signed 71-bit range; otherwise it restores the duplicated
low-word sign bit.  These functions are the required boundary adapters for
mixed 71-bit and raw-72 interfaces.

---

## Source: DAIMOS-LANGUAGE-DIRECT-WIDENING.md

# DAIMOS direct 36-to-71-bit widening v1

KCC and PDP-10 GCC directly convert signed and unsigned 36-bit words to the
normalized 71-bit representation.

For signed input, the high word is the arithmetic sign extension and the low
word preserves the complete 36-bit source word, including the duplicated sign
position.

For unsigned input, source bit 35 becomes bit 0 of the high word and the low
word retains source bits 0 through 34 with its sign position clear.

Direct full-range 71-to-36-bit narrowing remains outside the common compiler
subset in GCC. Use `daimos_u71_to_u36()` or `daimos_i71_to_i36()` from
`daimos-wide-convert.h`; these helpers are the canonical DAIMOS boundary
until GCC stops folding the conversion to an ordinary low-word subobject.

---

## Source: DAIMOS-LANGUAGE-FULL-RANGE-CONVERSIONS.md

# DAIMOS full-range integer conversions v1

## Selected semantics

Signed 36-to-71-bit widening sign-extends the high word and preserves the
complete low source word, including its duplicated sign position. Unsigned
36-to-71-bit widening transfers the source high bit into the upper word and
keeps the remaining 35 bits in the low word. Narrow 6-, 7-, 8-, 9-, and
18-bit values first undergo the ordinary 36-bit integer promotion.

KCC previously cleared the low-word sign position during signed widening;
the correction preserves the low word and normalizes constant materialization.

DAIMOS conversions retain the low destination-width bits.  Unsigned results
use modulo arithmetic.  Signed results interpret the retained bits as a
PDP-10 two's-complement word.

For a normalized 71-bit value, a 36-bit result is not merely the stored low
word.  The low word contains 35 independent value bits; result bit 35 comes
from bit 0 of the high word.

## Portable interface

`support/common/daimos-wide-convert.h` provides:

- `daimos_u36_to_u71`;
- `daimos_u71_to_u36`;
- `daimos_i71_to_i36`.

These helpers use ordinary shifts, masks, and additions.  They require no new
PDP-6 instruction, arithmetic mode, or runtime library entry point.

## Compiler status

KCC now reconstructs the full 36-bit result for direct casts from its
normalized 71-bit type.

PDP-10 GCC still folds some direct full-range casts as raw low-word moves and
also folds some unsigned 36-to-71 constants incorrectly.  Until its generic
conversion lowering is corrected, DAIMOS public code must use the helper
interface for values that may have bit 35 set.

Direct casts remain valid for the already verified representable subset.
