# Daimos C Language

This document consolidates measured PDP-10 KCC/GCC interoperability findings originally maintained in pdp10-testkit. Each subsection preserves its source document and qualification; measurements are not generalized beyond their stated probes.

## OS-independent PDP-6 libc BLT acceleration (2026-10-14)

KCC and PDP-10 GCC share a separately linked freestanding `libp10c-v2.a`
archive, independent of DAIMOS kernel services and compiler arithmetic
helpers. Standard C memory/string functions operate on the compiler's
9-bit `char`; the separate `p10bytes-v2.h` interface accepts raw PDP-10
byte-pointer descriptors of variable widths.

On baseline PDP-6, aligned `memcpy`/`memset` and native 6/9/18/36-bit
`p10bcopy_v2`/`p10bfill_v2` use `BLT` for four or more complete 36-bit
words. Small transfers use ordinary word instructions, and partial or
unaligned fields use `LDB`/`DPB`/`IBP`. Forward seed propagation is safe
for uniform fill. `memmove` keeps its direction-sensitive backward path;
`BLT` must not be used for an unsafe overlapping copy.

The testkit verified four C semantic programs under both KCC and GCC
(eight passing execution cells) and separate assembly regressions for
6-, 7-, 8-, 9-, 18-, and 36-bit byte pointers, including long-copy and
fill tails. The installed KCC and GCC archives were binary-identical
in staging. These are correctness results; actual runtime speedups and
an optimal `BLT` crossover threshold remain to be measured.

---

## KCC header search and `#include_next` (2026-10-14)

The Unix-hosted KCC preprocessor supports GNU-style `#include_next` with
both `<header>` and `"header"` spelling. Search resumes after the explicit
or implicit include directory that supplied the current header; the origin
is preserved through nested includes. An excluded directive has no effect.

KCC's host installation includes its target headers under
`$(KCCLIBDIR)/include`, normally `$PREFIX/lib/kcc/include`. The testkit
uses `$PDP10_PREFIX/lib/kcc/include` or an explicit `KCC_INCLUDE_DIR` during
compiler tests; ordinary GCC headers are not interchangeable with KCC's
target ABI definitions.

The PDP-6 testkit regression checks three successive include directories,
both operand spellings, nested include return and an excluded directive.
This preprocessor support is **not** validation of the varargs calling ABI
or of libc linkage; those semantic tests remain separately pending.

---

## Source: DAIMOS-LANGUAGE-COMPATIBILITY.md

# DAIMOS KCC/GCC language compatibility v1

## Exact-width common subset

KCC and PDP-10 GCC share exact signed and unsigned 6-, 7-, 8-, 9-, and
18-bit integer semantics.  The v13 executable probes verify truncation,
sign interpretation, integer promotion, array storage independence, argument
use, and returned values under both compilers.

The canonical spellings are `daimos_intN_t` and `daimos_uintN_t` from
`support/common/daimos-language.h`.

## Widths not shared natively

GCC supports exact 16- and 32-bit integer types.  KCC does not.  KCC must not
silently call a 36-bit `int` an exact 16- or 32-bit type.

The shared header therefore reports `DAIMOS_HAVE_EXACT_INT16` and
`DAIMOS_HAVE_EXACT_INT32` as zero under KCC.  Explicitly widened adapters are
available as `daimos_word16_t`, `daimos_uword16_t`, `daimos_word32_t`, and
`daimos_uword32_t`; their names intentionally do not promise exact storage or
arithmetic width.

## Attributes

New compatibility code must use named DAIMOS attribute macros.  Diagnostic
attributes such as `unused`, `noinline`, and `noreturn` may degrade to empty
macros under KCC because this does not alter object representation or the
calling ABI.

Semantic and ABI attributes do not degrade silently.  `DAIMOS_PACKED`,
`DAIMOS_ALIGNED`, `DAIMOS_WEAK`, and `DAIMOS_ALIAS` deliberately produce a
KCC compilation failure until KCC implements the requested behavior or a
specific adapter is selected.

The catch-all `__attribute__(x)` macro in the old assembly-comparison shim is
legacy test infrastructure.  It is not part of the DAIMOS language
compatibility interface and must not be used by new shared source.

## PDP-6 native-compiler constraint

This phase adds no KCC compiler machinery.  It reuses KCC's existing native
byte types and makes unsupported widths visible at preprocessing or compile
time.  That keeps the native compiler small, avoids additional register
pressure, and requires no instructions unavailable on a PDP-6.

Future KCC language additions must be justified against native PDP-6 compiler
size, memory use, register pressure, bootstrap complexity, and generated
runtime cost.  Exact 16- or 32-bit types should not be added merely to mirror
GCC if explicit word adapters provide the required semantics more cheaply.

---

## Source: DAIMOS-LANGUAGE-CHAR-ENUM.md

# DAIMOS plain character and enumeration language rules v1

## Scope

This document records the common KCC/GCC behavior verified for PDP-6-compatible DAIMOS C source.

## Plain character types

- `char` occupies one addressable character unit.
- Plain `char` is an unsigned 9-bit type in the shared PDP-10 language subset.
- `signed char` is a signed 9-bit type.
- `unsigned char` is an unsigned 9-bit type.
- Arithmetic promotion follows the existing DAIMOS narrow-integer rule: all three promote to signed 36-bit `int` because every unsigned 9-bit value fits.

Portable source must use `signed char` when negative character values are required. It must not depend on host-C conventions where plain `char` may be signed.

## Enumerations

For enumerators representable as a signed 36-bit word:

- the enumeration object occupies one 36-bit word;
- values and comparisons behave as signed 36-bit `int` values;
- fixed arguments and returns use the ordinary one-word integer ABI.

Enumerations requiring values outside the signed 36-bit range are not yet part of the shared subset. Public DAIMOS interfaces should use an explicit `int71_t`, `uint71_t`, or raw representation instead of relying on compiler-selected extended enum types.

## PDP-6 implementation cost

These rules match KCC's existing native byte and word models. They require no new KCC machine mode, runtime helper, or additional native compiler pass.

---

## Source: DAIMOS-LANGUAGE-PROMOTIONS.md

# DAIMOS narrow integer promotions v1

The shared exact-width 6-, 7-, 8-, 9-, and 18-bit types use the existing PDP-10 byte representations, but expressions promote their values to the native 36-bit `int` domain before ordinary arithmetic and comparison.

This rule avoids adding a machine-mode lattice or new arithmetic operators to KCC. It is suitable for a native PDP-6 compiler because narrow values remain storage types while computation uses existing word arithmetic.

Unsigned narrow values also promote to 36-bit `int`: every supported unsigned narrow value fits in a signed PDP-10 word. Mixed narrow/word expressions therefore use signed 36-bit arithmetic unless an operand is explicitly converted to `unsigned int`.

The `DAIMOS_INT*_C` and `DAIMOS_UINT*_C` macros construct word-sized source constants. Assignment to an exact-width object performs the documented truncation. They do not create distinct narrow literal types.

Exact 16- and 32-bit arithmetic remains outside the KCC common language subset.
