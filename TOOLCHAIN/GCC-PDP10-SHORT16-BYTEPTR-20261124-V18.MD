# GCC PDP-10 short16 byte-pointer repair, v18

Status: verified bounded GCC compiler/backend correction, 2026-11-24.

## Data representation

A `short16` occupies two nine-bit C addressable storage units, but
one physical PDP-10 16-bit byte-pointer element. Generic C pointer
arithmetic may present a dynamic index multiplied by two to target
lowering, whereas packed record-field lowering provides an already
unscaled element index. The two paths cannot be treated alike.

The GCC `pdp10_expand_binop` hook now detects a proven generic
multiply-by-two in the original pointer-expression tree and converts
the already-evaluated RTL back to physical element units. This does
not reevaluate side-effectful index expressions. The early symbolic
byte-pointer constant fold uses the matching short16 normalization;
without it, `&a[3]` could encode the wrong P/S and address bits.

## Exact-width loads and result register

GCC's optimized lowering of a signed short16 pointer dereference
could produce `HLRZ` (18-bit halfword load), then sign-extend only
16 bits. This read -32767 as 4 in a focused source reproducer.
For a constant encoded native `POINT 16` address, the backend now
uses `LDB`, which retrieves the exact 16-bit field.

On KS10, the dynamic short16 adjustment can select an S=1 logical
pointer or a native S=16 pointer. The native path previously returned
a new register to avoid modifying the original byte pointer, but
the dispatcher's caller ignored that result. Choosing a
non-overlapping destination before the branch gives both paths the
same output register and fixes optimized reverse pointer adjustment.

## Test semantics

GCC defines its built-in `char6` type as unsigned SIXBIT, while KCC
defines `char6` as signed. The shared testkit signed six-bit oracle
now uses explicit signed `int6`, and separately verifies GCC's
unsigned `char6` values. The signed expectations have not been
removed or relaxed.

## Acceptance and outstanding failures

The new registered GCC C89 short16 regression passes all **36/36**
target profiles in an isolated checkpoint directory, including all
four machines and their relevant optimization modes. The older broad
byte-pointer test passes all **9/9** GCC O0 profiles but still has
**11 optimized run failures and 16 optimized timeouts**. These
are open bugs and must be investigated separately; no complete
byte-pointer or full compiler-matrix acceptance is claimed.

The earlier unsigned division/remainder fix was regression-tested
through four existing matrix sources: **144/144** GCC cells passed
under strict exit policy in that same isolated checkpoint directory.

Implementation and source-linked tests:
`gcc-pdp10/gcc/config/pdp10/pdp10.c` and
`pdp10-testkit/history/PDP10-SHORT16-BYTEPTR-REPAIR-20261124-v18.md`.
