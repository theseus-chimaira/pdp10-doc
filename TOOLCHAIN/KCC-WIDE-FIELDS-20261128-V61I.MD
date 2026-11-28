# KCC wide bitfields and raw aggregate pairs (v61i)

KCC 43e2c4d, testkit ce6903a0. Supersedes the width limitation in
KCC-NARROW-LONGLONG-FIELDS-20261128-V61G.MD and the next-stage
restriction in KCC-CROSSWORD-FRAGMENTS-20261128-V61H.MD.

GNU/KCC mode accepts signed and unsigned long long bitfields from
37 through 71 bits. The type retains the exact width and TF_BITF
but uses the existing two-word arithmetic value representation.
The ordinary 07300 field offset encoding is unchanged. Physical
storage is a contiguous field bit stream, not the arithmetic
high36:low35 pair with its redundant ABI sign bit. Native fragments
split at both physical word boundaries and the arithmetic limb
boundary; neighboring fields survive deposits. Packed fields with
unknown runtime phase use the existing logical bit-pointer stepping.
Static initialization emits the high payload followed by low35.
Wide arithmetic preserves GNU field precision until conversion to
another arithmetic type; narrowing normalizes the high limb. The
sizeof, address and alignment constraints on bitfields still apply.

The new tests exposed and corrected two existing ABI problems:
PDP-6/KA simulated DMOVE must fetch the second indexed word first
when the first destination overlaps the address; raw two-word
register aggregate parameters must retain both words. Postfix pair
updates also preserve the old result without freeing the live pair.
No runtime archive, GCC or DAIMOS source changes.

## Verification

Evidence under TMPDIR: kcc-wide-fields-v61i.d1bbZx. Final candidate-v4.
- Four target-specific preexisting wide-field oracles: 88/88 pass.
- kcc-wide-field-boundaries-v61i: 22/22 pass. Independent raw words,
  signed and unsigned boundaries, returned two/three-word aggregates,
  GNU narrow precision, side-effecting assignments and packed arrays.
- kcc-raw-pair-abi-v61i: baseline 0/22, candidate 22/22 pass.
- Five negative sources (including revised v60m width limit): 110/110.
- Existing v61h native fragment generation guard: 8/8 pass.
- Combined related regression suite: 810/854 pass. The 44 runtime
  failures are exactly the original bf64-1.c and 991118-1.c cells.

Historical 1042-cell replay: 782 pass, 194 compile_fail, 66 run_fail.
Exact failure-key comparison against v61g: 22 recovered cells of
920501-3.c; 44 compile-to-runtime transitions of bf64-1.c and
991118-1.c; zero new failure keys. The two original tests compare
negative 52/53-bit results to positive 64-bit hexadecimal constants
on a 71-bit long long target. The target-specific signed-value and
structure-operation companions pass. The original cases remain
reported as failures, not removed or converted into passes.
The preexisting 22 bf-sign-2.c runtime failures remain unchanged.
Authoritative manifests are PDP10-KCC-COMPILE-REMAINING-20261128-v61i.tsv
and PDP10-KCC-COMPILE-RUNTIME-TRANSITIONS-20261128-v61i.tsv.

## Native compiler footprint

Baseline v61h cross-compiles both source revisions for PDP-6 with
-Pgnu99 -O -m=gas -DHOST_DAIMOS=1 -DHOST_UNIX=0; DAS DOBJ sizes:

| Unit | Before text | After text | Delta |
| --- | ---: | ---: | ---: |
| ccdecl | 13922 | 14000 | 78 |
| cceval | 7178 | 7245 | 67 |
| ccgen | 5122 | 5230 | 108 |
| ccgen2 | 26292 | 26836 | 544 |
| ccout | 7500 | 7523 | 23 |
| ccstmt | 8895 | 8932 | 37 |
| cctype | 2133 | 2289 | 156 |

Sum of text increases: 1013 words. Data and BSS unchanged in each
unit. No claim about final linked phase size, native execution time,
peak dynamic RAM or full self-host execution is made. The additional
text is a feature cost, and should be considered in further size
optimization work. Arbitrary-phase packed fields are not yet fragment
optimized. No full compiler sweep beyond the reported suites.

## Installation

Compiler-only install-v1/local.tar.gz installed into PDP10_PREFIX;
installed compiler compared byte-identical to candidate-v4.
Previous binary retained at install-v1/previous-kcc-v1.

bin/kcc SHA256:
194aefee4ef425e5b8e407b2e1a27d554862d1f31ad30135a15551c2cc7fe67a

local.tar.gz SHA256:
cbf9f51aa10ec8a651e1c21f5924112fc4edbb44ce454b3fef248aea43cb5072
