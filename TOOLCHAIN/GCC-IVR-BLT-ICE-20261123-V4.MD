# GCC PDP-10: Virtual Registers, Raw Double Subwords, and BLT Addresses

Date: 2026-11-23.

## Recovered compiler defects

Two previously independent groups of GCC 3.2 ICEs shared an RTL
validation boundary in `instantiate_virtual_regs_1`.

The C front end can reinterpret a DImode pseudo-register as DFmode,
then access one 36-bit SImode word through a second SUBREG. An earlier
fix canonicalized this form as a SET source; when it appeared as a
destination, the enclosing instruction was still unrecognizable.
`gcc/function.c` now uses a single helper to canonicalize both sides
before address substitution. This transformation is an integer/floating
**bit reinterpretation**, not a floating-point conversion, and is
restricted to the DI->DF->SI shape. The separate complex-DImode
target instruction patterns are not changed.

Separately, a BLKmode address can be loaded through a pointer variable
stored on the stack. The BLT expander passed that memory reference
directly to `pdp10_blt_setup`, which expects effective-address
operands. A pointer increment then became `PLUS(MEM,1)` and failed
instruction recognition during virtual-register instantiation.
`gcc/config/pdp10/pdp10.c` now materializes memory-held pointers in
registers in `expand_blt` and `expand_blt_clear` before constructing
the BLT control word and end address. Direct register and constant
address paths are unchanged.

## Verification boundary

All 414 originally failing GCC ICE cells were rerun against the staged
compiler. The two changes moved 38 cells from compiler ICE to testkit
pass: 11 `20020617-3.c`, 7 `BLT.c`, 9 `20011217-2.c`, 9 `991228-1.c`,
and 2 `pdp10-gnu-dec-signbit-function-v1.c`. No other cell changed.
The current total is 25 ICE, 180 pass, 176 diagnosed compile failures,
and 33 runtime failures.

Raw status table:
`pdp10-testkit/workdir/ice-retest-20261123-v7/results-v1.tsv`.
The compiler was staged, not installed. A codegen pass does not by
itself demonstrate guest runtime correctness; the semantic tests
retain their separate execution acceptance criterion.
