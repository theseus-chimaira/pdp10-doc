# GCC indexed bitfield word strides (v59z)

## Repair

GCC `9f3e31a5` extends the existing native record reference recognizer
in expr.c, pdp10_native_record_decl_ref, to nonvolatile bitfields of
1..36 bits in word-stride record arrays. Previously the recognizer
rejected DECL_BIT_FIELD and required the selected member's bit position
to be divisible by the member width. Such references entered the
generic subword byte-pointer path, which could scale the containing
record subscript by the member width and emit IBP against an unencoded
word address. The v59y indexed-field-store probe reproduces this bug.

Obtain the leaf bitfield width from DECL_SIZE rather than the promoted
expression type. A bitfield need not begin at a multiple of its own
width. Retain the actual constant bit position and accumulate each
variable array stride in physical words, including padding. Allow
dynamic word-stride arrays embedded in an ordinary record VAR_DECL.
The existing physical scalar slice helpers emit extraction, sign
extension and insertion. No runtime helper, persistent storage, new
target register convention, or generic optimization change is needed.

Packed records and nonword strides retain their previous paths.
The existing memory-backed root check is retained. The change is
limited to native nonextended mode, nonvolatile direct record
declarations, with the original native array restrictions for
non-bitfield members. No general indirect/volatile/packed or wide
bitfield repair is claimed.

## Acceptance

Testkit `4b9bf6e1` adds two 44-profile tests:
- gcc-indexed-bitfield-copy-v59z.c retains the original v59y probe
  (indexed stores followed by aggregate copies and fills).
- gcc-indexed-bitfield-words-v59z.c verifies the physical stored words
  through a separate character-wise copy into a union oracle, signed
  fields, local and static arrays, nested multidimensional strides,
  compound assignments, incrementing subscripts evaluated once, and
  neighboring field/record guards.

Both new tests fail at assertion 1 with the saved pre-repair GCC
on PDP6 O2, and pass all 44 installed GCC profiles each. The unchanged
pdp10-bitfield-array-writeback-v1.c fails assertion 2 in the old
compiler's PDP6 O2 replay, and now passes 16/16. Its PDP6 O0 emitted
stack frame decreases from 31 to 7 words and instruction lines
from 316 to 165; all 12 erroneous IBP instructions are removed.
These are measurements of one generated program, not global stack,
linked-image size or runtime timing claims.

Initial and independent oracle runs: 104 PASS, 16 unchanged run_fail
(signed 52/53-bit operations). Separate 560-cell guard replay: 544
PASS, 16 unchanged run_fail (packed-pointer array). The latter
preserves 324 original passes, 44 previously repaired original
link_fail cells, and 176 newer passing cells; zero new regressions.
Combined matrix acceptance: 648 PASS, 32 unchanged run_fail, 680
distinct selected cells. Two additional PDP6 O0/O2 small register
array probes pass. Compiler builds successfully; the legacy GCC
host build continues to emit existing warning classes.

## Remaining scope

A direct volatile indexed bitfield probe fails both before and after
this patch on PDP6 O2. Its emitted IBP path shows the remaining
volatile exclusion from the physical word-coordinate lowering.
Do not declare volatile indexing repaired. An equivalent nonvolatile
indirect pointer-based probe passes before and after on PDP6 O2;
this does not certify every indirect access. Retain the volatile
probe for the next compiler repair, with actual volatile memory
effects and correct assignment-result semantics.

The original 404 link-failure cohort remains 259 genuine runtime
passes, 48 corrected registrations and 97 unresolved. This patch
repairs separate runtime defects, not additional original link
failures. No full post-repair matrix sweep, native self-host or
extended-section runtime claim. DAIMOS kernel code is unchanged.

## Evidence

`$HOME/tmp/gcc-indexed-fields-v59z.YmhfOTgy`:
initial-v1, words-v1, guards-v1, guard-comparison-v59z.tsv,
baseline (old compiler failures), short.s, original-probe.c,
scope-probes (including volatile/indirect before/after), build-v1.log.
Installed compiler-only install-v1/local.tar.gz SHA-256:
`459570583c5a611140aec6d46aceb19bc25820aeef075cb02645b380a9a384f4`.
Runtime archives were not modified.
