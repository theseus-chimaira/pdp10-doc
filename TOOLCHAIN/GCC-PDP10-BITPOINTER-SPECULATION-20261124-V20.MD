# GCC logical bit-pointer speculation repair v20

Status: verified bounded runtime repair, 2026-11-24.

## Failure

PDP-10 short16 pointers can denote native S=16 byte positions or
exact packed fields represented by logical S=1 bit pointers. GCC
selects these representations at runtime. On pre-KL10 CPUs, the
logical branch previously contained an opaque untyped ADJBP
operation lowered at assembler output into a reverse IBP search.
GCC optimization could hoist this ostensibly pure instruction ahead
of the selector branch. A native S=16 pointer then entered a search
intended for S=1, which never found its target. A representative
PDP-6 O1 guest looped at octal PC 005127..005132 and expired the
100-million-step simulator limit.

## Correction

The logical S=1 branch now expands through typed one-bit pointer
arithmetic expressed as real RTL. Constant multiples of 36 bits
become right-half word-address adjustments without loops; otherwise
the established real RTL pointer adjustment remains available.
Large counts are not truncated through the host-int helper. Zero
offsets preserve the original pointer. The optimizer sees the real
control flow of any necessary bit steps, and cannot speculate an
opaque nonterminating assembly loop into the native pointer path.
KS10 native ADJBP continues to use the hardware instruction.

Implementation is in `gcc-pdp10/gcc/config/pdp10/pdp10.c`.

## Acceptance

The official independent PDP-10 testkit run of
`runtime-byte-pointer-diff.c` advanced from 9/36 PASS and 16
timeouts to **33/36 PASS and zero timeouts**. The remaining three
failures involve two separate KS10 optimized load defects (one O1
signed 9-bit extraction and O2/Os halfword selection); they have not
been accepted or discarded.

The new registered C89 short16 reverse-pointer test passes 36/36
GCC profile cells. All 216 cells of six older v17-v19 GCC test
sources also pass unchanged under strict exit policy. This is a
bounded correctness result, not a complete full matrix run.

Detailed source-linked results are in
`pdp10-testkit/history/PDP10-SHORT16-REVERSE-POINTER-20261124-v20.md`.
