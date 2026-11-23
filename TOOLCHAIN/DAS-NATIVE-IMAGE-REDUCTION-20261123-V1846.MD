# Native DAS image reduction, v1846

The native assembler uses separate KCC-built DAS1 and DAS2 images.
The strict image ceilings are 034000 and 031000 PDP-10 words,
respectively. These are resident constraints and must not be relaxed.

## Reproducible baseline

With the installed KCC and the original DAS assembler, KCC flags
`-Pgnu99 -O -x=pdp6 -m=gas` produce:

| Phase | Baseline image | BSS | Ceiling | Excess |
| --- | ---: | ---: | ---: | ---: |
| DAS1 | 035216 | 002103 | 034000 | 001216 |
| DAS2 | 032730 | 001573 | 031000 | 001730 |

All figures are octal words. Rebuilding with `-Os` or `-O2`
instead of `-O` produces identical image and BSS sizes.

## Accepted source reduction

The expression parser previously duplicated symbol resolution in
`parse_expr_primary()` for ordinary names and numeric local labels.
Both branches performed lookup, mutable SET snapshot resolution,
relocation/value computation and undefined-symbol diagnostics.

The shared `expr_resolve_name()` now performs that work once, while
preserving the essential distinction: an undefined numeric local
label may refer only to an already-existing object symbol, whereas
an ordinary undefined name may add an external object symbol.
The existing SET generation and relocation behavior is preserved.
This is source deduplication, not a new feature or compatibility shim.

| Phase | Before | After | Words saved | Remaining excess |
| --- | ---: | ---: | ---: | ---: |
| DAS1 | 035216 | 035103 | 000113 | 001103 |
| DAS2 | 032730 | 032620 | 000110 | 001620 |

All figures are octal. Both BSS values remain unchanged.
This is a measured reduction in BOTH resident phase images, but
neither image ceiling has been reached yet.

The existing host regression checks for phase IR/transport,
reference assembly, C comments, conditional/repetition errors,
macro errors, numeric radix, relocation and symbol differences
passed. The strict native image-size gate correctly still fails
on DAS1's remaining excess. No native guest PASS is claimed.

The versioned diagnostic
`daimos-testkit/tests/validation/daimos-das-native-size-profile-v1845.py`
prints both image limits, BSS and the largest source assembly regions.
The regions are source-line counts, not exact per-function word
measurements. Further reductions must be measured against both
native images and validated without changing the ceilings.

## Follow-up: shared literal operator classification (v1847)

The literal image-size calculation previously duplicated a leading
opcode/operator classifier already present in the literal SET
canonicalizer. It now reuses the classifier, retaining a mode
distinction for exact special-operator tokens versus historical
prefix recognition during canonicalization. The literal size path
still treats doubled commas as single-word literal syntax and a
single ordinary comma as two-word data.

| Phase | After v1846 | After v1847 | Further saved | Remaining excess |
| --- | ---: | ---: | ---: | ---: |
| DAS1 | 035103 | 034653 | 000230 | 000653 |
| DAS2 | 032620 | 032367 | 000231 | 001367 |

All values are octal image words; BSS remains unchanged. Relative
to the original baseline, the combined v1846/v1847 changes save
000343 words in DAS1 and 000341 words in DAS2.

Ten relevant host DAS regression scripts pass, covering phase IR,
phase transport, reference assembly, macro and conditional errors,
numeric radix, C comments, relocations and symbol differences.
The strict native size limits still fail, so the reduction is not
yet a native acceptance.

## Follow-up: native expression operator decoding (v1848)

The native-only precedence-climbing expression parser decoded ten
binary operators through a sequence of conditional branches. A
single character switch now selects the operator and precedence,
retaining the two-character shift-token check and the existing
recursive expression evaluator. This is an implementation reduction,
not a change to precedence, relocation or arithmetic semantics.

| Phase | After v1847 | After v1848 | Further saved | Remaining excess |
| --- | ---: | ---: | ---: | ---: |
| DAS1 | 034653 | 034612 | 000041 | 000612 |
| DAS2 | 032367 | 032326 | 000041 | 001326 |

All values are octal image words. BSS remains 002103 (DAS1)
and 001573 (DAS2). Relative to the original baseline, the three
source reductions save 000404 words in DAS1 and 000402 in DAS2.
Ten host DAS regression scripts passed. Both native image ceilings
remain exceeded, so this is not native acceptance.
