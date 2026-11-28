# KCC GNU void indirection (v61f)

Date: 2026-11-28.

KCC 0afd8bb, testkit 83dfaeef. GNU/KCC mode accepts void-pointer
indirection as a void expression. The parser preserves the pointed-to
type and pointer representation, including for &*p. Discarding the
expression evaluates its pointer operand but does not read an object,
even when the pointed-to void type is volatile-qualified. A volatile
pointer object itself is still read. The generator enforces this rule
with optimization disabled as well as enabled. GNU sizeof(void) and
sizeof(*p) for a void pointer yield one C address unit, without
evaluating the operand. No void object storage or value conversion
is enabled by this change.

Strict C89/C99 still reject void indirection. The diagnosed expression
retains its typed lvalue for recovery instead of replacing it with an
integer dummy that subsequently triggered evalunop internal diagnostics
when its address was taken. The prior strict sizeof(void) diagnostic
policy is unchanged.

## Verification

The v61e baseline and candidate were tested on charybdis using
separate prefixes and fresh checkpoints. The 88 targeted cells
cover original 20000701-1.c, a new semantic regression and two
negative tests. Baseline: 44 pass, 33 compile_fail, 11 compiler_ice
(internal diagnostics during recovery from rejection of the new GNU
source). Final candidate: 88 pass. The new semantic source checks
pointer-expression side effects, branch selection, comma expressions,
void return, address identity and unevaluated sizeof. Negative tests
reject integer use and assignment of void lvalues.

The registered test kcc-void-indirection-v61f additionally compares
the emitted function bodies with equivalent discarded-pointer casts:
32 comparisons across PDP-6, KA10, KI10, KS10, opt/noopt. It requires
the volatile pointer and function-call symbol references to survive.
Four explicit strict C89/C99 opt/noopt rejections pass the ordinary
diagnostic/no-ICE checker. An intermediate candidate still produced
strict-mode internal diagnostics; final typed recovery eliminates them.

Fresh 1042-cell active compilation replay: 760 pass, 282 compile_fail.
Compared with v61e, exactly 22 cells of 20000701-1.c recovered and
no new failures. No ICE, link, run or timeout failures in this replay.
The remaining 18 sources are recorded in the testkit manifest
PDP10-KCC-COMPILE-REMAINING-20261128-v61f.tsv. Of the original 2196
compilation failures, 1154 remain documented unsupported, 760 are
recovered and 282 remain active. No new scope exclusions.

## Native footprint and installation

Same v61e host compiler, native PDP-6 -Pgnu99 -O -m=gas,
HOST_DAIMOS=1, HOST_UNIX=0; DAS -F -C. DOBJ sections in words:

| Source | Text before | Text after | Data before/after | BSS before/after |
| --- | ---: | ---: | ---: | ---: |
| ccstmt | 8885 | 8895 | 2 | 23 |
| cceval | 7138 | 7138 | 0 | 2 |
| ccgen2 | 26177 | 26199 | 3 | 6 |

Total +32 text words; no persistent data or BSS increase. Linked
native phase size, native phase execution and timing were not measured.

Evidence relative to TMPDIR: kcc-void-v61f.cB4oCQ (baseline-results,
final-guards, final-replay, native, strict-mode logs, install-v1).
Compiler-only local.tar.gz installed into PDP10_PREFIX; installed
bin/kcc compared byte-for-byte with the final tested candidate.
Previous binary retained in install-v1/previous-kcc-v1.

Installed bin/kcc SHA256:
cc6b3c1f1ba265ceb7396c9a583450353b56c4d40c9764fa124a8d96cd92fe4c

Compiler-only local.tar.gz SHA256:
0ceae38401a9c4e6d971c608917a4dc4ad3a956c094c2afbe4a04c6a326265ff
