# KCC 27-bit target floating-point constant folding (2026-10-20)

## Implementation

Hosted KCC previously evaluated `TS_FLOAT` arithmetic, comparisons and
casts through IEEE binary32's 24 significant bits. PDP-10 36-bit single
floating point retains 27 bits, so this could silently change legal
constants and comparisons even after correct literal parsing.

The compiler now rounds operands and results explicitly to target
precision. A host `double` retains the rounded value and a sufficiently
wide `long double` retains arithmetic guard/sticky bits. Arithmetic
folding is deferred to target execution on hosts without sufficient
intermediate precision. Floating addition and multiplication are not
reassociated across variables, because target rounding can change the
result when evaluation order changes.

The host-only numeric helpers live in shared `ccoututil.c` so KPARSE,
KGEN, KOPT and integrated KCC use the same representation. No target
runtime instructions, stack or writable library data are added.

### KPCODE2 intermediate stream

The hosted compiler's `PCODE.Pfloatword` uses the existing PCODE union
storage to carry the exact 36-bit target float word. KGEN/KOPT serialize
and reconstruct that word in `KPCODE2` instead of truncating through
IEEE binary32. The stream's updated magic and version reject incompatible
`KPCODE1` files. The binary record test verifies exact round-trip of
`0234400000002`.

## Verified PDP-6 scope

The versioned testkit suites prove:

- 20 compile-time arithmetic, comparison and cast examples, and
  positive halfway runtime arithmetic comparison, pass on SIMH PDP-6
  with optimized and unoptimized KCC
- Both modes pass the same folding guest on the independent AAP PDP-6
  emulator. The immediate-HALT negative control is correctly rejected
- The real KCPP → KPARSE → KGEN → KPCODE2 → KOPT pipeline preserves the
  exact target float constant and executes the linked guest successfully
  on SIMH PDP-6 in both modes
- KPCODE2 malformed-format/version rejection and target-word
  serialization regression passes

These results do **not** extend automatically to KA10/KI10/KS10/KL10
hardware floating modes or to all floating overflow/underflow cases.

## Open hardware and compiler-phase discrepancies

The AAP PDP-6 model reports `0543400000000` for a negative halfway
native FADR/FSBR case, whereas the installed SIMH PDP-6 reports
`0544000000001`. The compiler's PDP-6 folding model follows the
independent AAP observation, but source hardware documentation is still
needed to reconcile the simulator discrepancy. The testkit's manual
`kcc-pdp6-negative-fp-runtime-v1` regression currently fails on SIMH
and is **not** counted as accepted behavior.

The older source-wide byte-identity comparison of integrated KCC versus
KPARSE/KGEN/KOPT also remains manual. Optimized KPARSE deliberately runs
`bindopt`, but an unoptimized output difference starting in `cccse.c`
remains unexplained. The focused real-phase floating-point test above
passes; that is narrower evidence than claiming all translation units
produce byte-identical assembly.

Authoritative test sources, exact expected values and recorded known
failure are in `pdp10-testkit/docs/KCC-PDP6-TARGET-FLOAT-FOLDING-20261020-v1.md`.
