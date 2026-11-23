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

## Follow-up: shared instruction encoder and text emission (v1849-v1850)

In v1849, the ordinary phase-2 instruction statement path reused the
instruction encoder already used for embedded instruction words in
data and literals. Strict base-profile checks, privileged I/O checks
and unknown pseudo-op diagnostics remain in the statement path. The
shared encoder accepts an omitted operand for ordinary instructions,
but a bare mnemonic inside a data expression still has to be
distinguished from a symbol name. Phase2 saved 000204 octal words.

In v1850, separate SIXBIT and 7-bit ASCII/ASCIZ data emitters became
one word-oriented text packer. SIXBIT produces six 6-bit characters per
word. ASCII and ASCIZ produce five 7-bit characters and a spare low
bit; the pass-1 word count retains the ASCIZ terminator. Phase2
saved another 000110 octal words. The DAS fast suite, full DXR/native
self-test suite and DOBJ/DARC/DLINK integration passed for both changes.

A second mnemonic helper experiment was rejected: it re-tokenized
less source but grew phase2 by 000016 octal words. The production
v1849 code avoids that additional helper.

## Follow-up: computed OWGBP positions (v1851)

The historical 21-element OWGBP selector lookup table contains five
contiguous byte-width ranges (6, 7, 8, 9 and 18 bits). Computing
`low = (selector - first_selector) * width` preserves the old table
for every defined selector and avoids retaining the selector table
in native phase2. This saves 000022 octal words.

The versioned daimos-testkit
`tests/validation/daimos-das-owgbp-compact-v1851.sh` checks all
21 valid selectors against an independent DXR-word oracle and
rejects eight gaps/invalid selectors. It passes with the revised
assembler and with the original installed DAS. The DAS fast,
full DXR/native, stream/bounded-store and DOBJ integration suites
all pass.

## Follow-up: SET names, BYTE widths and physical line reader (v1852-v1854)

In v1852, SET snapshot names are formatted directly into their
caller-provided buffers. All three callers provide at least 40
characters; the seven-character prefix and an unsigned 36-bit
decimal generation fit within 19 characters including NUL.
This removes a temporary buffer and a second copy, saving 000056
octal words from each native phase.

An alternative iterative decimal-digit formatter was considered:
its elimination of recursion could reduce stack depth, but the
tested implementation grew both phase images by 000103 octal
words. It was not retained. Simply substituting `char_distance`
for `strlen` also grew both images by 000001 octal words and was
reverted.

In v1853, parenthesized first-field and subsequent-field BYTE
widths share one bounded absolute-value parser that restores the
closing delimiter even on parse errors. This saves 000006 octal
words per phase.

In v1854, physical source-line EOF and newline finalization share
one path. The input limit is checked directly against `cap - 1`:
when the limit is reached, the input length remains at `cap - 1`
until the end of that physical line, preserving overlong-line
recovery. The native `das_read_line` function shrank from 135
to 105 instructions; the explicit 136-instruction acceptance limit
passes, while the separate 104-instruction optimization target is
still exceeded by one instruction. Native phase1 saved 000036
octal words. Input, fast, full DXR/native and DOBJ integration tests
pass.

## Current exact sizes after v1854

All numbers below are **octal native PDP-10 words**, measured by
rebuilding the KCC native sources and reading the produced DXRs.
The native image ceilings are unchanged and remain mandatory.

| Version | DAS1 image | DAS2 image | DAS1 excess | DAS2 excess |
| --- | ---: | ---: | ---: | ---: |
| Original | 035216 | 032730 | 001216 | 001730 |
| v1848 | 034612 | 032326 | 000612 | 001326 |
| v1849 | 034612 | 032122 | 000612 | 001122 |
| v1850 | 034612 | 032012 | 000612 | 001012 |
| v1851 | 034612 | 031770 | 000612 | 000770 |
| v1852 | 034534 | 031712 | 000534 | 000712 |
| v1853 | 034526 | 031704 | 000526 | 000704 |
| v1854 | 034470 | 031704 | 000470 | 000704 |

Total cumulative image savings versus the original baseline are
000526 in DAS1 and 001024 in DAS2. BSS remains 002103 and 001573.
The strict native phase-image size check still fails on DAS1
034470 > 034000 and also, independently, on DAS2
031704 > 031000. No completed native acceptance is claimed.
