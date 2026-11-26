# KCC static absolute word-pointer constants (v59g)

A static initializer containing a non-null absolute word pointer with a
qualification conversion could cause KCC to emit a runtime `.LINK`
constructor instead of the constant word. DAS rejects that constructor.
This was exposed by the native PDP-6 bitmap overlap fixture in DAIMOS.

The fix in `ccgen.c:gizconst` recognizes a pointer-to-pointer cast of an
`N_PCONST` whose source and destination have the same ordinary word-pointer
representation. It preserves the constant word instead of constructing a
runtime initializer. Byte, packed, polymorphic, bit, and void pointer
conversions are excluded from this rule. This is a compiler correction,
not an assembler compatibility workaround.

The dedicated `daimos-testkit/tests/validation/kcc-absolute-pointer-v59g`
regression rejects the previous compiler and passes native PDP-6 execution
with the corrected compiler at `-O` and `-O0`. The corrected compiler also
passes the source-linked bitmap overlap fixture and its negative controls.

- KCC commit: `89c036e` (v59g).
- Testkit commit: `944b19b`.
- The regression covers ordinary absolute word-pointer qualification; it
  does not establish correctness for all other pointer representations.
