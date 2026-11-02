# Daimos Call Abi

This document consolidates measured PDP-10 KCC/GCC interoperability findings originally maintained in pdp10-testkit. Each subsection preserves its source document and qualification; measurements are not generalized beyond their stated probes.

---

## Source: DAIMOS-WIDE-CALL-ABI.md

# DAIMOS 71-bit fixed-argument calling ABI v1

## Scope

This document records the measured fixed-argument calling convention for the
normalized PDP-10 71-bit `long long` model.  It applies to direct KCC/GCC calls
using ordinary non-variadic C functions.  It does not claim variadic,
structure-return, floating-point, or raw 72-bit native ABI compatibility.

## Measured convention

A normalized 71-bit value occupies one adjacent accumulator pair.  The high
word is in the lower-numbered accumulator and the low word is in the next
accumulator.

For the simple fixed-argument functions measured by
`tests/compat/wide-call-abi.c`:

- the first 71-bit argument is passed in AC1:AC2;
- the second 71-bit argument is passed in AC3:AC4;
- a 71-bit return value is returned in AC1:AC2;
- ordinary calls use the PDP-10 `PUSHJ 17,symbol` convention;
- a compiler may replace a final call with a tail `JRST symbol` after restoring
  its caller frame.

Both KCC and GCC generate this convention.  Direct mixed-compiler calls for
these fixed signatures therefore require no adapter.

## Representation requirement

The accumulator pair uses the same normalized 71-bit representation as memory:
the high word contains the sign and upper value bits, while the low-word sign
position duplicates the sign.  Passing a flat raw 72-bit pair through this ABI
is invalid even though both representations occupy two words.

## Consumed-pair register lifetime

A mixed signature such as

```c
int71_t f(int a, int71_t b, int c)
{
    return b + a + c;
}
```

places `a` in AC1, `b` in AC2:AC3, and `c` in AC4.  A read-once 71-bit
parameter may be consumed directly from its incoming pair, but those ACs cease
to be protected ABI inputs at that point and must become available to the
normal temporary allocator.  The KCC regression suite checks this lifetime
rule explicitly so later DImode operations do not exhaust `rrdfind`.

## Compatibility status

Direct native ABI compatibility is established only for the measured
fixed-argument 71-bit cases.  The following remain unresolved and must not be
inferred from this result:

- 71-bit values after accumulator arguments are exhausted;
- mixtures with pointers, narrow byte values, structures, or floating point;
- variadic functions and `va_list` traversal;
- structure return and hidden result pointers;
- native raw 72-bit calls under GCC `-mlong-long-72bit`;
- caller- and callee-saved accumulator rules beyond the observed functions.

---

## Source: DAIMOS-WIDE-VARARGS-ABI.md

# DAIMOS wide variadic ABI status v3

## Decision

KCC and PDP-10 GCC may interoperate directly across variadic calls only for
argument types whose external representation has been demonstrated to agree.
The compiler-private `va_list` representation is not part of this contract.

Direct mixed variadic calls are currently supported for:

- default-promoted narrow integer arguments;
- ordinary 36-bit integer arguments;
- normalized 71-bit `int71_t` arguments;
- ordinary word pointers;
- packed byte pointers whose byte size/position representation is shared;
- aggregates whose layout is already classified as ABI-compatible;
- floating arguments after the normal variadic promotion to `double`.

The v30 mixed matrix executes these classes in both directions:

- GCC caller -> KCC variadic callee;
- KCC caller -> GCC variadic callee.

## Unsupported direct cases

A native type is not automatically variadic-compatible merely because a
scalar value of that type can be converted to the same mathematical value.
Direct mixed variadic use remains unsupported for representations already
known to differ, including pointers to native exact 16-bit or 18-bit objects
whose C storage layout differs between KCC and GCC.

For such interfaces, use an explicit wrapper and a canonical ABI value.  The
existing exact-width adapters convert 16-bit and 18-bit scalar values to/from
ordinary word-register representations; raw 72-bit interfaces use `raw72_t`.
The wrapper must perform the conversion before entering the variadic boundary
or after leaving it.  Do not expose compiler-private `va_list` objects across
the boundary.

## Evidence

`tests/compat/test-kcc-gcc-varargs-matrix.sh` compiles the same caller and
callee interfaces with both compilers and executes both mixed directions.  It
covers promoted narrow integers, a normalized 71-bit value, an ordinary word
pointer, a packed byte pointer, a two-word aggregate, a promoted floating
argument, and a trailing word argument.

`tests/compat/test-wide-varargs-abi.sh` independently covers normalized 71-bit
stack traversal with a KCC variadic callee and both KCC and GCC callers.

These runtime tests establish external call compatibility only for the
enumerated representations.  They do not make KCC's compatibility
`__builtin_va_*` macros and GCC's native varargs lowering equivalent for code
size or performance comparisons; those remain result-only under the
optimization-comparison policy.

---

## Source: DAIMOS-MIXED-FIXED-EXEC.md

# DAIMOS mixed KCC/GCC fixed-call execution

Executable KS10 probes confirm direct calls in both directions for normalized
71-bit fixed arguments and returns:

- GCC caller to KCC callee;
- KCC caller to GCC callee.

Both tests pass two 71-bit arguments in AC1:AC2 and AC3:AC4, return the result
in AC1:AC2, link the generated assembly together, and execute it under SIMH.
Variadic interoperability is tested separately by `test-wide-varargs-abi.sh`.

---

## Source: DAIMOS-SYMBOL-RUNTIME-ABI.md

# DAIMOS external symbol and wide-runtime ABI v1

## Measured common external-symbol subset

KCC and GCC emit the same assembler-visible spelling for external C identifiers
that are no longer than 31 characters. The measured subset includes lowercase
names, mixed-case names, and names beginning with an underscore.

DAIMOS public C and assembler interfaces shall therefore use externally visible
identifiers of at most 31 characters. This applies to functions, objects, and
runtime entry points intended to cross compiler boundaries.

## Longer identifiers

GCC preserves identifiers longer than 31 characters. KCC diagnoses truncation
and emits only the first 31 characters. Two distinct source identifiers sharing
those first 31 characters can therefore collide in a KCC object.

Long external names are not part of the common ABI. Source-private identifiers
may be longer only when they cannot become assembler or linker symbols.

## Case and leading underscores

The compiler outputs preserve the measured source spelling, including case and
a leading underscore. This document does not yet define whether every supported
assembler, librarian, and linker preserves case distinctions. Public interfaces
should not rely on two symbols differing only by case until the complete object
tool chain has been measured.

## 71-bit runtime helpers

GCC emits calls to `__divdi3` and `__moddi3` for the measured signed 71-bit
division and remainder operations. KCC emits compiler-private division code and
does not reference those GCC helper names.

Consequences:

- Runtime-helper names are not presently a shared KCC/GCC compiler ABI.
- A mixed image containing GCC objects must provide the required GCC helpers.
- DAIMOS public assembly interfaces must not call compiler-private helpers.
- Shared helpers, if introduced, need explicit names no longer than 31
  characters, documented arguments, results, clobbers, and edge behavior.
- Legacy compiler helper aliases may be added after their complete ABIs are
  measured; name similarity alone is insufficient.

## Unresolved

The following remain to be measured:

- case significance in DAS and the final object/linker tools;
- maximum symbol significance in every object format and linker;
- weak symbols, aliases, common symbols, and tentative definitions;
- GCC helper argument registers, clobbers, divide-by-zero behavior, and signed
  overflow behavior;
- helper requirements for unsigned division, multiplication, and conversions.
