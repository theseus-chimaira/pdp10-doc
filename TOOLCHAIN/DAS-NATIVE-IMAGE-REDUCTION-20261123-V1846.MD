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

## Follow-up: reuse parsed operands and canonicalizer copies (v1855-v1861)

The phase parser already supplies both a classified directive token and
an operand pointer. Native DAS had nevertheless parsed the directive's
mnemonic again in the ASCII/ASCIZ/SIXBIT word counters and text emitter,
then in BYTE and LONG counting/emission paths. V1855-v1856 now pass the
operand directly to those functions. The three text word counters use
one bounded delimiter/count implementation; fixed-width division is
preserved for the 5-character ASCII/ASCIZ and 6-character SIXBIT words.
The existing BYTE first-width and subsequent-width rules are unchanged.

In v1857, the literal SET snapshot canonicalizer shares one checked
output copy for replaced and unreplaced names. V1858 classifies an
IF/IFDEF/IFNDEF/ELSE/ENDIF once before processing phase-1 conditional
state. V1860 uses one call to the relocation-aware expression
addition/subtraction helper. V1861 eliminates a name buffer memmove
in the literal operator classifier while retaining the original
token-length distinction (including a leading dot).

A v1859 experiment that scanned bounded literal text without first
copying to a NUL-terminated temporary buffer was discarded: the KCC
phase images grew from 034226/031400 to 034302/031453 (octal), even
though the candidate could have used less temporary stack. Keeping
the strict resident-image ceiling was more important than accepting
that measured text growth.

| Version | DAS1 image | DAS2 image | DAS1 excess | DAS2 excess |
| --- | ---: | ---: | ---: | ---: |
| v1854 | 034470 | 031704 | 000470 | 000704 |
| v1855 | 034410 | 031521 | 000410 | 000521 |
| v1856 | 034317 | 031430 | 000317 | 000430 |
| v1857 | 034267 | 031400 | 000267 | 000400 |
| v1858 | 034226 | 031400 | 000226 | 000400 |
| v1860 | 034203 | 031355 | 000203 | 000355 |
| v1861 | 034173 | 031344 | 000173 | 000344 |

Measured savings from v1854 through v1861 are 000275 words in DAS1
and 000340 in DAS2. Relative to the original v1845 baseline they
are 001023 and 001364 words. Phase1/phase2 BSS remains 002103/001573.
The unchanged mandatory ceilings are 034000/031000: **both still fail**.
The source-built full DAIMOS guest has not accepted these images.

The versioned, independent daimos-testkit fixture
`tests/validation/daimos-das-directive-operands-v1856.py` checks
ASCII/ASCIZ/SIXBIT word packing across boundaries, BYTE field sizes,
LONG lists and file/pipe transport identity. The established fast and
full host suites cover conditional and macro semantics, expression
precedence, literal references and DXR/native selftests. No new DAS
feature or resident allocation was introduced.

## Follow-up: symbol return and phase-2 packing (v1862-v1864)

V1862 makes numeric-local and ordinary expression names share a single
symbol-resolution return tail, preserving their distinct lookup policy.
V1863 reuses the first `,,` delimiter found in the phase-2 data parser
instead of repeating the same string search. V1864 specializes the
text word-packing width to the two supported inputs, 6 and 7 bits,
instead of requiring native division by a parameter. All three changes
reduce native TEXT without modifying BSS or supported syntax.

| Version | DAS1 image | DAS2 image | DAS1 excess | DAS2 excess |
| --- | ---: | ---: | ---: | ---: |
| v1861 | 034173 | 031344 | 000173 | 000344 |
| v1862 | 034135 | 031307 | 000135 | 000307 |
| v1863 | 034135 | 031301 | 000135 | 000301 |
| v1864 | 034135 | 031253 | 000135 | 000253 |

All values are octal. The strict native phase limits still fail.
The host fast/full, KCC self-assembly, native comment-scanner,
independent directive word/transport, and exhaustive OWGBP tests pass.
The `das_read_line` function is 105 instructions against its 136 hard
limit and optional 104 target. Physical guest assembly acceptance has
not been established for this checkpoint.

## Follow-up: phase-2 operand decoding and native I/O (v1865-v1868)

V1865 strips the optional leading dot as the instruction mnemonic is
copied, preserving the original 31-character input truncation, rather
than copying it and shifting the name again. V1866 removes the second
effective-address trim: the index-suffix parser always performs that
trim, including when no index suffix exists. V1867 shares the native
scratch-file short read/write transfer loop. V1868 also makes phase-2
output writes use that same transfer path. Phase 1 retains its direct
output writer because full sharing increased its image; nevertheless,
the stream-parameter specialization adds 000004 words back to phase 1
relative to v1867 while removing 000034 from phase 2. The trade-off
reduces total resident code, does not increase BSS, and keeps the same
short-transfer and error semantics.

| Version | DAS1 image | DAS2 image | DAS1 excess | DAS2 excess |
| --- | ---: | ---: | ---: | ---: |
| v1864 | 034135 | 031253 | 000135 | 000253 |
| v1865 | 034135 | 031244 | 000135 | 000244 |
| v1866 | 034135 | 031242 | 000135 | 000242 |
| v1867 | 034110 | 031215 | 000110 | 000215 |
| v1868 | 034114 | 031161 | 000114 | 000161 |

All values are octal native PDP-10 words. Compared with the first
v1845 measurement, v1868 saves 001102 in DAS1 and 001547 in DAS2.
BSS is unchanged (002103 and 001573). Both mandatory image ceilings
continue to fail and no current native DAIMOS guest PASS is claimed.
The v1869 experiment sharing a single quotient/remainder sign fixup
was discarded because each native image grew by 000002 words; its
original two-branch form was restored. This does not invalidate the
arithmetic identity, but KCC produces smaller code from the original.

## Follow-up: compact directive index and full process limit (v1870-v1871)

V1870 changes phase-2 text emission to advance an input pointer and
remaining-character counter instead of calculating `word_index *
characters_per_word + slot` at every character. Its fixed-width text
padding and ASCIZ NUL rules remain unchanged. This saves 000023 octal
words in DAS2 and does not change DAS1.

V1871 replaces the 43 pairs of 36-bit keyword keys and word-sized token
identifiers with 43 single-word entries. Six alphabetic letters fit in
30 bits at five bits each, leaving six bits for a token identifier
(the maximum identifier is 30). The sorted binary search and six-letter
prefix matching remain, including P2ALIGN's distinct seven-character
recognition and historically accepted overlong EXTERN_ matches. An
independent C89 oracle in daimos-testkit verifies all 43 spellings,
lowercase variants, P2ALIGN, and invalid/misspelled keys. It passed,
along with the host fast/full assembly suites, KCC self-assembly,
directive/transport oracle and exhaustive OWGBP checks.

| Version | DAS1 image | DAS2 image | DAS1 image excess | DAS2 image excess |
| --- | ---: | ---: | ---: | ---: |
| v1868 | 034114 | 031161 | 000114 | 000161 |
| v1870 | 034114 | 031136 | 000114 | 000136 |
| v1871 | 034054 | 031076 | 000054 | 000076 |

V1871 saves **000040 octal words in each phase** relative to v1870.
An additional v1872 probe-key local was neutral under KCC and was
reverted. Total savings relative to v1845 are now 001142 for DAS1 and
001632 for DAS2. BSS remains 002103 and 001573 respectively.

The diagnostic profiler now reports the mandatory **040000-word total
process ceiling**, including the 02000-word reserved stack, in addition
to each phase's separate image limit:

| Phase | Image | BSS | Reserved stack | Process total | Process excess |
| --- | ---: | ---: | ---: | ---: | ---: |
| DAS1 | 034054 | 002103 | 002000 | 040157 | 000157 |
| DAS2 | 031076 | 001573 | 002000 | 034671 | 000000 |

All numbers above are octal PDP-10 words. The DAS1 process ceiling is
more restrictive than its nominal image ceiling with current BSS:
DAS1 must reduce its image to at most **033675** or reduce BSS by a
corresponding amount. The strict image acceptance also remains open
for DAS2. No new native guest PASS is claimed. The next optimization
should account for both image and process limits; do not relax either.

## Follow-up: shared scratch names and buffer lifetimes (v1874-v1876)

The v1874 change removes a redundant full path array from each native
scratch-wordfile. Each descriptor now keeps only an immutable suffix;
the native assembler context keeps a pointer to the caller-owned output
filename. Opens and cleanup reconstruct exactly the original basename
plus suffix into the already resident `das_native_tmp` buffer, preserving
the 102-character path limit and all six scratch-file suffixes.
This buffer is unused during scratch initialization and again after the
source pass has returned, so it cannot collide with line decoding.

V1875 overlays native include-directory scratch space with the parser's
`das_native_tmp` buffer, which is dead after `parse_include_line()` has
extracted the include name. The full include depth and per-depth source
filenames remain unchanged; no nested include capability was removed.
The fixed-size native directory allocation is therefore unnecessary.

V1876 fixes two independent native error branches (phase 1 and phase 2)
that passed a format-string pointer as a numeric diagnostic code when
rejecting unsupported `.DIRECTIVE` syntax. Using `DAS_DIAG` restores the
existing host message and proper numeric native diagnostics, and also
shrinks both phase images.

| Version | DAS1 image | DAS1 BSS | DAS1 process | DAS2 image | DAS2 BSS |
| --- | ---: | ---: | ---: | ---: | ---: |
| v1871 | 034054 | 002103 | 040157 | 031076 | 001573 |
| v1874 | 034117 | 001707 | 040026 | 031134 | 001346 |
| v1875 | 034116 | 001655 | 037773 | 031134 | 001346 |
| v1876 | 034101 | 001655 | 037756 | 031117 | 001346 |

All figures are octal PDP-10 words. DAS1 and DAS2 now both satisfy the
040000-word complete process budget with their reserved 02000-word
stacks. However the strict resident **image** ceilings 034000/031000
still fail by 000101/000117 respectively. Do not interpret the
full-process PASS as native guest acceptance.

New independent `daimos-testkit` test
`daimos-das-native-scratch-paths-v1875.sh` compiles both actual native
phase configurations against mocked DAIMOS UUOs. It verifies all phase-1
and phase-2 suffixes, open/unlink/close ownership, failure cleanup and
the 102-character path boundary. Both phase variants pass. The normal
host DAS fast/full, KCC self-assembly, directive/transport, OWGBP and
comment-scanner validations remain required independently. The native
guest integration gate remains OPEN.

## Follow-up: merge phase-2 opcode keys and metadata (v1877)

The existing sorted 390-entry opcode index retained a full 36-bit
mnemonic key plus separate packed ten-bit opcode/nonbase metadata. In
phase 2, the 364 entries with three through five alphabetic characters
now use one word per entry: a five-bit-per-letter mnemonic key and its
ten-bit opcode/nonbase metadata fit into 35 bits. The 26 six-letter
entries retain their exact 30-bit key and a separate compact nine-word
metadata array. Phase 1 still uses its original table.

The old phase-2 390+130=520-word constant tables became 364+26+9=399
words, a reduction of 121 table words. Additional runtime decoding
accounts for some of that reduction, so **measured** linked images must
be used rather than treating table savings as total image savings.

| Version | DAS1 image | DAS1 BSS | DAS2 image | DAS2 BSS |
| --- | ---: | ---: | ---: | ---: |
| v1876 | 034101 | 001655 | 031117 | 001346 |
| v1877 | 034101 | 001655 | 031037 | 001346 |

These are octal native words; v1877 saves 000060 in DAS2 with no BSS
change. Both full-process budgets remain within 040000, but image
limits are still exceeded by 000101 (DAS1) and 000037 (DAS2).

The new independent `daimos-testkit` frozen oracle
`daimos-das-opcode-map-v1877.tsv` records every pre-change opcode and
nonbase flag; its ISO C89 checker also verifies lowercase aliases and
invalid opcodes against the new implementation. It validates all 390
records without generating expected output from the implementation
being tested. Native guest acceptance remains OPEN.

## v1878 correction: preserve the six-bit internal opcode interface

The first compact prototype passed the public 390-opcode name oracle
but failed the host JUMP/JRST optimizer regression. The optimizer and
macro parser also call `lookup_op_mn()` directly with existing six-bit
mnemonic keys; the prototype changed that internal argument encoding.
V1878 restores the six-bit internal host interface by converting its
keys to five-bit only at the decoder boundary. The native phase-2
public lookup uses the compact decoder directly, so its resident image
size remains 031037. The frozen oracle now checks BOTH lookup entry
points and all 390 pre-change encodings and nonbase flags, to prevent
this specific class of regression. Host fast/full and native scratch
tests pass. Native phase-image ceilings still fail as above.

## v1881: eliminate concurrent launcher and first pass

The public native `das` launcher formerly used RUN to spawn DAS1 and
waited for its exit, retaining the launcher's image and 02000-word
reserved stack alongside DAS1. The measured v1878 combined peak was
**042743 octal = 17,891 decimal words**, exceeding the fixed
040000-word (= 16,384 decimal) budget.

The corrected phase chain uses two process-image replacements:

1. `das` constructs a counted-SIXBIT EXEC V1 argument block, marks
   argv[0] with the private `DAS1CHAIN` identifier, and replaces its
   own image with DAS1. It no longer launches a waiting child.
2. DAS1 retains the original parser and scratch-backed DASIR2 writer.
   Direct DAS1 invocations still execute only phase 1. With the private
   marker, a successful, closed intermediate file triggers EXEC of DAS2
   with the original output record. An error does not launch phase 2.
3. DAS2 executes in the same process, so its exit status is reported to
   the original caller without a resident driver.

To accommodate the small transition routine, the native symbol hash
index was resized from 128 to 64 bucket heads and the cache from 13
to seven entries. They accelerate an otherwise unchanged spill-backed
symbol store; symbol capacity and syntax remain unchanged. This saves
000232 octal BSS words in each phase relative to preliminary v1879.

| Process | Image | BSS | Reserved stack | Resident process |
| --- | ---: | ---: | ---: | ---: |
| Launcher | 000571 | 000000 | 002000 | 002571 |
| DAS1 | 034331 | 001423 | 002000 | **037754** |
| DAS2 | 031037 | 001114 | 002000 | 034153 |

All figures are octal PDP-10 words. The maximum **steady-state** user
allocation is **037754 octal = 16,364 decimal words**, 20 decimal words
below the 040000-octal (16K) budget. The simultaneous 17,891-word
driver+DAS1 residency no longer occurs.

The independent daimos-testkit tests `daimos-das-exec-chain-v1880.sh`
and `daimos-das-exec-peak-v1880.py` verify the native EXEC argument
blocks, error behavior, and size budget for each DXR. Host assembler
fast/full suites and the independent 390-opcode oracle pass.

**Outstanding:** DAIMOS `exec_replace_current` can stage a replacement
VM before freeing the old image. Its low-memory fallback frees the old
VM before loading the next one, but this does not establish a hard
16K-word transient physical-RAM bound without guest measurements. The
separate strict image ceilings 034000 for DAS1 and 031000 for DAS2
still **fail** by 000331 and 000037 octal words. Native guest acceptance
and transient-memory acceptance remain OPEN.

## v1882: pair three-letter native phase-1 opcode keys

The 59 three-character base-opcode names use exactly 18 bits in DAS's
historical six-bit key encoding. Two now occupy one 36-bit constant
word, with one unused half-word for the odd final entry. Longer base
names retain their original full keys. The lookup still performs a
sorted binary search and reconstructs the original full key before
comparison, so the recognized opcode set is unchanged.

This saves 29 table words but adds some lookup instructions. **Measured
linked DAS1 image savings are 000015 octal words**, with no BSS change.
The new independent native-phase1-only opcode oracle checks all 390
frozen pre-change names plus unknowns, complementing the full phase-2
opcode+nonbase oracle and host assembler regressions.

| Phase | Image | BSS | Full process including stack |
| --- | ---: | ---: | ---: |
| DAS launcher | 000571 | 000000 | 002571 |
| DAS1 | 034316 | 001423 | 037741 |
| DAS2 | 031037 | 001114 | 034153 |

All values are octal. The steady-state process maximum is now 037741
octal = 16,353 decimal words, or **31 words below 16K**. Strict
individual native image limits still FAIL by 000316/000037 (octal),
and the guest/transient-EXEC validation remains OPEN.

## v1883: compact the phase-2 extended-opcode table

The 24 non-base opcode names previously each used separate word-sized
mnemonic and opcode fields. The 22 names of three through five letters
now fit in one 36-bit word with the nine-bit opcode; the two six-letter
names remain exact direct comparisons. The public lookup reuses its
existing five-bit mnemonic key without encoding it twice. A new frozen
24-opcode oracle independently checks original opcode values,
non-base flags, lowercase aliases and invalid names.

The measured linked DAS2 image shrinks **031037 -> 031022 octal words**
(000015 saved), with unchanged BSS 001114. DAS1 remains 034316 image,
001423 BSS, and a full process size of 037741 including the stack.
Both native process budgets and the single-process steady-state 16K
RAM ceiling remain satisfied. The separate image ceilings are still
OPEN: DAS1 exceeds 034000 by 000316 and DAS2 exceeds 031000 by 000022.

## v1884: shorten the private phase-file basename; retain hard RAM gate

When the output is `PROGRAM.DXR`, DAS1 now writes the DASIR2 transport
to `PROGRAM.D2R`, and native DAS2 derives and removes that same path.
The host file-backed two-phase path follows the same naming rule.
For non-`.DXR` outputs the existing appended-suffix convention is
preserved, importantly `PROGRAM.D2R` output still uses the distinct
`PROGRAM.D2R.D2R` temporary filename. Source and destination paths
are not conflated, and both phases derive their path from `-O`.

The change is **not a memory optimization**. The pathname buffers have
fixed bounds; changing the characters does not shrink them. The chosen
`strrchr`/`strcmp` test reuses existing native helpers, and `strchr` finds
the append point without generating expensive indexed byte-pointer
arithmetic. This is smaller than the tested byte-indexed and
shared-function variants, and adds only 000005 octal image words to
each native phase. Measurements from
fresh v1884 native binaries (all counts octal; stack reserves 002000):

| Program | Image | BSS | Reserved process | Hard image limit |
| --- | ---: | ---: | ---: | ---: |
| DAS launcher | 000571 | 000000 | 002571 | n/a |
| DAS1 | 034323 | 001423 | **037746** | 034000 (FAIL +000323) |
| DAS2 | 031027 | 001114 | 034143 | 031000 (FAIL +000027) |

`037746` octal = **16,358** decimal 36-bit words, leaving only **26**
below `040000` octal/16,384 decimal. The v1880 static executable
image+BSS+stack gate PASSes. Fast/full host DAS tests and the v1875,
v1880, v1882, v1883 regressions PASS. The new versioned v1884 host
marker-path test and mocked native DAS1/DAS2 monitor-path tests also
PASS, including output-name collision avoidance.

The requirement of **at most 16K total RAM at every instant** is
STRONGER than the steady-state process gate. The normal kernel EXEC
transaction constructs a staged VM before releasing the previous one,
potentially exceeding 16K during driver -> DAS1 and DAS1 -> DAS2.
The kernel's low-memory destructive fallback is triggered on a staging
allocation failure, not unconditionally. No physical guest peak was
measured in this checkpoint, and neither image-only gate passes.
Absolute 16K physical RAM and complete guest acceptance therefore
remain OPEN; do not reinterpret the steady-state PASS as that proof.
