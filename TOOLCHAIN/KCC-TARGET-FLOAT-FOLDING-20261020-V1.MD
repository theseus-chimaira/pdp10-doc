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

The original SIMH PDP-6 reported `0544000000001` for the negative
halfway `-134217728.0F + -1.0F`, while AAP reported `0543400000000`.
The historical Stanford SAIL `10TO6.TED` explicitly specifies that
negative halfway floating arithmetic on a PDP-6 rounds toward zero,
unlike the later PDP-10. In SIMH's sign-extended two's-complement
fraction, the discarded bits were exactly `FPRBIT2`, but the PDP-6
path added a full `FPRBIT1`, crossing a normalization boundary.
SIMH now cancels the half-bit only for this negative exact-tie case;
other rounding operations, and all later PDP-10 CPU paths, are unchanged.
The original regression and eight cross-verified AAP/SIMH negative-halfway
cases pass under both KCC modes, as do 24 mathematically calculated
boundary cases on corrected SIMH.

**A different AAP result remains unresolved, and AAP is not changed.**
For `-134217728.0F + 1.0F` (positive operand, not negative halfway),
AAP yields `0544000000003`. A mathematical exact-result oracle and
corrected SIMH yield `0544000000001`. The AAP result was independently
confirmed by guest memory examination (case index `011`, operand
words `0543400000000` and `0201400000000`). This may be an original
PDP-6 arithmetic peculiarity rather than an emulator defect: point 6
of `10TO6.TED` describes loss of significant low-word bits during
PDP-6 floating normalization. Whether that condition applies here
requires studying the original DEC 166 CPU drawings, especially the
`FAT*` and `NRT*` pulse sequences corresponding to AAP `emu/apr.c`.
No AAP correction has been designed or applied.

The complete AAP 24-case run is preserved as a separately registered
**manual, failing** test; the normal independent AAP acceptance includes
the eight confirmed negative-halfway cases. Treat AAP hardware fidelity
as **unproven**, not disproven, until primary DEC circuitry resolves the
exact-result discrepancy.

Primary evidence: [SAIL 10TO6.TED (1973), Floating Point Instructions,
items 5 and 6](https://www.saildart.org/10TO6.TED%5BH%2CDOC%5D).

## DAIMOS peephole-ring phase identity (2026-10-20)

The previously unexplained unoptimized `cccse.c` output difference
was caused by inconsistent code-generator peephole ring capacities.
KGEN used 64 `PCODE` entries while DAIMOS KCC1 and integrated KCC
used 256. The register allocator consults the pending pseudo-code
history; KGEN's earlier flush changed its recent-register-use data
and therefore its allocation choices, despite equivalent input trees.

All DAIMOS KCC backends now use the **same 64-entry ring**, saving
192 `PCODE` slots in those previously using 256. Non-DAIMOS
builds retain the historical 256-entry configuration. An isolated
`cccse.c` experiment confirmed byte identity after using a common
ring size. The full regression
`pdp10-testkit/misc/kcc-regression/kcc-phase-byte-identity-20261006.sh`
then passed on all **27 compiler translation units**, with strict
integrated/KCC1 identity in both KCC modes and strict KIR1/KGEN
identity at `-n`. It has been re-enabled in the testkit.

Optimized KPARSE deliberately runs its additional `bindopt` pass;
its optimized KIR1 assembly need not be byte-identical to integrated
KCC. Instead, the KPCODE2 floating-point guest executes successfully
through the entire optimized and unoptimized phase pipeline.

A detached, sequential 13-check acceptance run also passed all
tests with zero failures, including the full phase test, KPCODE2
format/guest, SIMH PDP-6 arithmetic/conversions, AAP PDP-6 execution,
ABI and test inventory. Subsequently the SIMH PDP-6 negative-halfway
FADR/FSBR discrepancy was corrected and independently verified
against the AAP PDP-6 model on eight cases. The 24-case corrected
SIMH boundary suite passes; the separate AAP exact-normalization
discrepancy at `-134217728.0F + 1.0F` remains unresolved. The manual
AAP 24-case disagreement regression must not be counted as passing.

Authoritative test sources, exact expected values and recorded known
failure are in `pdp10-testkit/docs/KCC-PDP6-TARGET-FLOAT-FOLDING-20261020-v1.md`.
