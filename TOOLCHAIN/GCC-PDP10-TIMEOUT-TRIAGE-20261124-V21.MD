# Compiler timeout classification and trampoline correction v21

Status: source-verified, completed bounded repair, 2026-11-24.

## Correct distinction between timeouts

The recorded restartable testkit sweep contained 135 `timeout`
checkpoints from seven C test sources. Ninety exhausted the simulator
step budget and 45 hit the simulator's wall-clock deadline. A timeout
does not, by itself, prove an infinite loop or an invalid test. A
false positive must be demonstrated by an independent defined-source
oracle or a successful test with an appropriate bounded increase.

## Invalid test source

The upstream postincrement test `920428-1.c` had an uninitialized
one-byte array used with `|=`. This reads indeterminate data before
writing it. Initializing the byte removes undefined behavior without
changing the pointer postincrement or pointer comparison being tested.
The formerly timed-out KS10 O1 cell now passes. All **66/66** GCC/KCC
profiles of the corrected test pass under strict exit policy.

## GCC nested-function trampoline stack bug

The GCC `20000822-1.c` test had 33 timeouts due to an actual
compiler stack-layout defect. Generic trampoline alignment rounded
a PDP-10 *word address* to a multiple of four word addresses by
mistakenly applying a 36-bit boundary as four 9-bit storage units.
At KI10 O2 the executable two-word trampoline was placed beyond the
allocated frame, where a following `PUSHJ` overwrote it with the
return PC. A scratch assembly relocation restored correct behavior.

The target now uses a one-address-unit runtime trampoline alignment,
relying on the stack's existing word alignment. The separate
`TRAMPOLINE_COPY_ALIGNMENT` in GCC's generic template copying logic
keeps the template at a true 36-bit word alignment, retaining the
compact two-word `BLT` rather than an eight-byte `LDB/DPB` loop.
This fixes the addressing model rather than extending time limits.

The original 44 GCC nested-function profiles now all pass. The
final installed-compiler cross-check selected the corrected timeout
sources together with the previous v19/v20 pointer regressions:
**218/218 PASS**, 160 GCC and 58 KCC.

## Remaining clusters

The v20 byte-pointer work had already resolved 16 step-limit
timeouts. Together with one invalid test and 33 repaired trampoline
cells, **50/135** original timeouts have been eliminated.
The other 85 are clustered in GCC wide-integer (36), GCC packed
aggregate by-value (27), and two KCC optimizer cases (11 each).
An extended 200-million-instruction KI10/O2 wide-integer run still
expired at PC 0. The full 44 GCC packed-by-value profiles were
rerun: 27 timeout and 17 runtime assertion failures remain. These
are not counted as fixed or as accepted slow executions.

Source and checkpoint detail:
`pdp10-testkit/history/PDP10-TIMEOUT-TRIAGE-20261124-v21.md`.
