# GCC PDP-10 Static Byte-Pointer Relocation Correction

Status: repaired and verified using a staged GCC build on 2026-11-23.

## Architecture and failure

An unextended PDP-10 byte pointer uses a 36-bit word containing a
byte-position/size (P/S) selector in its left half and an 18-bit
word address in its right half. A static symbolic pointer contains
a relocatable word address plus an encoded selector.

GCC's generic static `MINUS_EXPR` expansion combined the address and
an integer displacement using `plus_constant`. For subtraction this
can borrow into the P/S field. That transforms a valid 9-bit byte
pointer into a different byte-pointer encoding even when the object
file assembles successfully. The proper operation advances the byte
position across 36-bit word boundaries while preserving the byte size.

Tree reassociation was a second independent problem. The generic
folding pass can strip a Pmode pointer cast, treating an `int *` word
pointer and a `char *` 9-bit byte pointer as the same representation.
Their target representations are not the same. Preventing a bad raw
subtraction alone is insufficient if the cast is already discarded.

## Compiler implementation

Four areas cooperate to preserve static pointer semantics:

- `gcc/fold-const.c`: preserve pointer-typed subtrees rather than
  reassociating an offset across a representation-changing cast.
- `gcc/varasm.c`: retain pointer-typed expressions in static scalar
  and aggregate initializer expansion instead of discarding the cast.
- `gcc/config/pdp10/pdp10.c`: evaluate constant word-pointer offsets
  symbolically, and constant local byte-pointer offsets with the
  existing ADJBP simplifier, without generating temporary registers.
- `gcc/expr.c`: in a constant static byte-pointer `MINUS_EXPR`, use
  the target ADJBP simplifier instead of `plus_constant` on the 36-bit
  encoding. Unfoldable cases remain visible to the initializer
  validation path instead of being silently emitted as raw arithmetic.

The assembler's local symbolic pointer output keeps selector and
relocatable right-half address independent. The fix does not introduce
any new runtime helper or DAIMOS resident code.

## Verification and limits

The `pdp10-testkit` source
`tests/semantic/regression/pdp10-static-pointer-cast-offset-v1.c`
exercises static pointer variables and a static aggregate. Volatile
accesses force reads of the initialized pointer words. It checks
four-, five- and eight-byte negative offsets, casts to and from
word pointers, plus-negative syntax, and referenced integer values.

All **44 applicable local GCC profiles passed** in the target emulator.
The exact original 414-cell GCC ICE baseline remains at zero ICEs,
with its prior 205 passes, 176 diagnosed compile failures, and 33
runtime failures unchanged. Nineteen adjacent pointer tests had
identical statuses with staged and installed compilers; the eight
runtime failures observed there are not resolved by this change.

Evidence and the A/B experiments are recorded in
`pdp10-testkit/history/GCC-STATIC-POINTER-RELOCATION-20261123-v1.md`.
The testkit records live in its versioned `workdir` result folders.
The installed compiler in `$PDP10_PREFIX` was left unchanged.

An additional independent test,
`tests/semantic/p10-static-pointer-rh18-v1.c`, was incorporated
from the previously untracked testkit source during repository
consolidation. It compares dynamically computed pointers with
standalone and aggregate static initializers after negative byte
offsets and casts back to word pointers. All 44 registered GCC
profiles passed in the emulator; see
`pdp10-testkit/workdir/static-pointer-orphan-20261123-v1/results-v1.tsv`.

## Verified additional upstream GCC linkage repair (v8)

The unmodified GCC upstream test
`tests/semantic/upstream/generic/20021101-1.c` contains global
initializers casting a word pointer to `char *`, subtracting eight
9-bit bytes, and casting back to `int *`. All its 44 previously
registered `link_fail` cells were due to corrupt negative RH18
relocation addends: `dlink` reported `RH18 relocation overflow`.

After the compiler repair, all 44 original cell identities pass
compilation, linking, and execution using the staged compiler.
The exact original failure keys were compared to the 44 new
checkpoints, and every new status is `pass`.

Result table:
`pdp10-testkit/workdir/static-pointer-upstream-link-20261123-v32/results-v1.tsv`.
This establishes the repair of those 44 link failures only; other
previous linker failures remain separate and require their own
verification.

This verification applies to local RH18 pointers; extended addressing
and unrelated compiler/runtime behavior require their own acceptance.
