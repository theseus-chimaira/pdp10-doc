# KCC native variadic builtins and packed tag layout (v61j)

KCC 2f7e016, testkit 3e88afa3. Native KCC mode recognizes
`__builtin_va_list`, `__builtin_va_start`, `__builtin_va_arg`,
`__builtin_va_copy` and `__builtin_va_end` without an include-time
macro substitution. The cursor type is the existing native char pointer.
The parser lowers these operations to existing typed expression nodes.
No new runtime archive, opcode, persistent type field or calling
convention is introduced. GCC's own stdarg implementation is unchanged.

## Native stack extent

A stack word is 36 bits, and a native C byte is 9 bits. Each unnamed
argument occupies `sizetype(type)` complete physical words. Its C
object size need not be a multiple of four bytes: a packed three-byte
aggregate occupies one stack word, and a packed five-byte aggregate
occupies two. Both the builtin lowering and the native header round
the cursor advance to the physical stack extent. The builtin reads a
packed aggregate's raw word image from the aligned stack slot rather
than using a byte load. The value retains the exact aggregate type.
`va_start` checks the final named parameter of a variadic function;
`va_copy` and `va_end` use the existing cursor representation and
preserve evaluation side effects. Invalid cursor/type usages are
diagnosed. These are native KCC operations, not a promise of portable
compiler-private va_list representation or new mixed GCC/KCC coverage.

## Aggregate tag identity

Struct and union interning must not depend on the mutable size of an
incomplete tag. Hash by tag identity and aggregate kind, compare type flags, and propagate the
completed word count and exact packed byte extent to qualified variants
in the existing bucket. Prefix `__attribute__((packed))` is consumed
by the defining tag before its member declarations, rather than being
mistaken for a member attribute. Suffix packed attributes retain their
existing behavior. All qualified variants share canonical tag layout.

This corrects a coverage defect in the v61i wide-field test: its prefix
packed spelling had silently produced a nonpacked array. The revised
oracle uses the supported trailing spelling and explicitly checks the
six-byte element size and twelve-byte array extent. Before the tag
repair, the genuine volatile packed array produced initializer ICEs in
all 22 configurations. After repair, all 22 pass, including the element
starting mid-word. The independent tag-layout oracle also covers
prefix and suffix packing, qualified forward declarations and ordinary
forward declarations. See KCC-WIDE-FIELDS-20261128-V61I.MD for the
historical wide-field work; its original packed-array coverage claim
is superseded by this verified correction.

## Verification

Evidence directory under TMPDIR:
`kcc-va-builtins-v61j.CuN3XV`. Final candidate-v6.

- Historical 1042-cell replay: 804 pass, 172 compile_fail, 66 run_fail.
- Compared by exact failure keys with v61i: 22 recovered cells of
  `20001123-1.c`, zero new failures, unchanged runtime failure keys.
- New builtin, packed header and tag-layout oracles: 66/66 pass on
  the final candidate. The genuine packed-header baseline failed
  all 22 configurations before the word-rounded advance repair.
- Final related guard suite: 1730/1774 pass. The 44 runtime failure
  keys exactly match the v61i guard suite (22 each of `bf64-1.c`
  and `991118-1.c`); zero new failure keys.
- Eight new negative sources cover invalid variadic cursor/type,
  address-taking, and invalid start/copy usages.
- The 66 historical runtime failures retain their original status;
  the 32/64-bit upstream expectations are not silently reclassified.

Authoritative remaining manifests:
`PDP10-KCC-COMPILE-REMAINING-20261128-v61j.tsv` and
`PDP10-KCC-COMPILE-RUNTIME-TRANSITIONS-20261128-v61j.tsv`.

## Native compiler footprint

Baseline v61i cross-compiles both revisions for PDP-6 with
-Pgnu99 -O -m=gas -DHOST_DAIMOS=1 -DHOST_UNIX=0; DAS DOBJ sizes:

| Unit | Before text | After text | Delta |
| --- | ---: | ---: | ---: |
| ccdecl | 14000 | 14021 | 21 |
| cclex | 3584 | 3616 | 32 |
| ccstmt | 8932 | 9360 | 428 |
| ccsym | 3117 | 3216 | 99 |
| Total | 29633 | 30213 | 580 |

Data and BSS unchanged in every unit. No measurements of linked native
phase size, peak dynamic RAM, native execution time or native self-host
execution. The 580-word increase remains a future size-optimization
opportunity. No GCC or DAIMOS source changes.

## Installation

`install-v61j/local.tar.gz` installed into PDP10_PREFIX. Both files
compared byte-identical to the verified candidate and source header.
The previous compiler and header are retained in the same versioned
installation directory.

- bin/kcc SHA256: 912b66bb1fa03d6f584835518baadc7124d96ed6395d00eb83e09ce6c8c02139
- lib/kcc/include/stdarg.h SHA256: 89c515c47682c6dd487376a16a4d40bbed18d274a2b7a80aef2027a04713ce7d
- local.tar.gz SHA256: 35ab7156d5b4b9c30029354a136f1133a71968eeb0a1c799b488998484b0e8b1
