# KCC identical literal pointer differences (v61k)

KCC 41fdb0c, testkit d603ca9f. Extends the v61d native
bit-coordinate evaluator to GNU/KCC-mode subtraction of addresses
into identical string literals, including wide literals. The C
standard leaves literal object identity unspecified; when folding
such a difference KCC may choose a common object for these two
occurrences. This does not establish a general literal-pooling or
cross-expression pointer-equality contract.

The coordinate walk retains an existing Q_IDENT or N_SCONST base
node instead of just the symbol, preserving symbol identity checks
for ordinary static objects and the existing null-derived member
case. For literals, require the same interned lexer payload, exact
length, type and NF_STRWIDE/NF_STR9 encoding. Lexer suffix sharing
alone is not sufficient. The final coordinates must lie within the
literal extent including one-past, with exact native element stride.
No new node, symbol, table or persistent metadata is allocated.
No unrelated-symbol or loaded-pointer subtraction is legalized.

The original 20001116-1.c remains unmodified and passes all 22
KCC configurations. The new semantic oracle covers 9-bit and wide
characters, embedded zero, concatenation, negative differences,
one-past and qualifier conversion. Distinct content, suffix-only
sharing and out-of-range addresses remain ordinary diagnosed errors.

## Verification

Evidence directory under TMPDIR:
`kcc-literal-difference-v61k.ZK06rX`; final candidate-v2.

- Focused baseline: 132/176 pass, 44 compile_fail. Final: 176/176.
- Registered special test kcc-literal-difference-v61k: 16 GNU89/99
  accepts, 16 C89/C99 rejections (four CPUs, opt/noopt), no ICE.
- Broader literal and pointer guards: baseline 281/442 pass,
  156 compile_fail, 5 policy_fail. Final 325/442 pass,
  112 compile_fail, 5 policy_fail. Exact failure-key comparison:
  the original and new oracle (22 each) recovered, zero new keys.
  One unregistered float-literal source found by filename selection
  was not scheduled; no matrix exclusions were added.
- Fresh historical 1042-cell replay: 826 pass, 150 compile_fail,
  66 run_fail. Exactly 22 cells of 20001116-1.c recovered, zero new
  failure keys, unchanged historical runtime failure keys.

Remaining authoritative manifests:
`PDP10-KCC-COMPILE-REMAINING-20261128-v61k.tsv` and
`PDP10-KCC-COMPILE-RUNTIME-TRANSITIONS-20261128-v61k.tsv`.

## Native footprint and remaining work

Baseline v61j cross-compiles both cceval revisions for PDP-6 with
-Pgnu99 -O -m=gas -DHOST_DAIMOS=1 -DHOST_UNIX=0; DAS DOBJ headers:
text 7245 -> 7316 (+71 words), data 0 -> 0, BSS 2 -> 2.
Linked native phase size, native self-host execution, peak dynamic
RAM and execution speed have not been measured. No full compiler
matrix replay beyond the reported suites.

The remaining extern-void symbolic integer initializer cases
20010327-1.c and 20011114-1.c need both declaration semantics
and exact full-word relocation arithmetic. Existing RH18/LH18
relocations do not propagate carry/borrow between halves; emitting
an RH18 address relocation for a 36-bit integer addend would be
incorrect. Full-word relocation support requires separate design and
validation across the assembler/linker and native object consumers.
Do not confuse it with pointer coordinate folding. Different-label static differences were recovered in v61o; see
KCC-STATIC-LABEL-DIFFERENCES-20261128-V61O.MD. No assembler, GCC or DAIMOS
source changes were made.

## Installation

Compiler-only install-v61k/local.tar.gz installed into PDP10_PREFIX;
installed compiler compared byte-identical to candidate-v2. Previous
binary retained at install-v61k/previous-kcc-v61j.

- bin/kcc SHA256: 0d86324df274ff87c5f7844a4c04f89868f4e5a536da3bdcb6ac4a72a2d898e2
- local.tar.gz SHA256: 267db2b0750e83e96b74d4313545bc8c290dedfae07b07d47d9ee0ad3ec61b5a
