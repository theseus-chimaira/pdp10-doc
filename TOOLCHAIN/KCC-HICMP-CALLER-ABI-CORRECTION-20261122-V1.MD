# KCC HIcmp LARGE-OFFSET CALLER ABI CORRECTION (V1)

Date: 2026-11-22.
Status: implemented, simulator-tested, committed and pushed.

## DEFECT

The earlier reduction of HIcmp.c from 1.2 GB to approximately 26 KB
of assembly fixed pathological instruction expansion but did not
establish that the resulting code was correct.

A separate defect remained in the immediate-displacement branch of
KCC's `simadjbp` in `ccout.c`.  For PDP-6, the compiler emitted
`SETM 1,3` and `SETM 3,1` to transfer a byte pointer between
accumulators.  Those effective-address references cannot access PDP-6
accumulators as ordinary low-core memory.  The same branch also used
`PUSH 17,1` and `POP 17,1` for register preservation, which are
not valid PDP-6 accumulator-to-stack transfers.

The repaired immediate branch saves AC1 with MOVEM into a real stack
slot, marshals the source byte pointer through a separate stack slot,
calls the existing ADJBP helper with AC1=pointer and AC16=count,
marshals the result back to the destination accumulator through
stack memory, and restores AC1.  The runtime helper's signed
quotient/remainder algorithm is unchanged.

## TESTS

* Original HIcmp.c regenerated for PDP-6, KA10 and KS10; none of
  the three outputs contains the incorrect SETM transfer.
* New registered assembly regression:
  `misc/validation/test-kcc-hicmp-codegen-v1.sh`.
* Existing signed ADJBP simulator fixture extended to reproduce the
  HIcmp-style immediate displacement 11,111,111 on a pointer in AC3,
  including address wrap and AC1 preservation.  PDP-6, KA10 and KS10
  guest executions all PASS.
* HIcmp.c and existing large-offset pointer regression: 40/40
  selected KCC matrix cells PASS.
* Inventory, script ownership, raw p10run ownership, and fixture
  integrity audits: zero issues.

HIcmp.c contains deliberately invalid absolute memory references.
Its strict runtime execution is not a meaningful acceptance oracle;
the generated ABI sequence and pointer arithmetic are checked
independently.  The original full compiler sweep was not rewritten.

KCC commit: `1af1a2e`.
Testkit commit: `5a65e3e`.

This corrects an actual output-code defect that was not caught by the
previous code-size and compilation-only acceptance.  It does not
claim that all other code emitted for HIcmp.c is semantically proven.
