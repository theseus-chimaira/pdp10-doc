# KCC bitfield constant width, integral declarations and aggregate array decay

Date: 2026-11-27. Extends KCC-FRONTEND-REPAIRS-20261127-V60K.MD.
Installed host compiler: KCC 443bb1c. Testkit: 985adab.

## v60l: constant conversion at target width

KCC 0550c4a; testkit 6ab70d8. The existing constant conversion
used host int shifts for bitfield masks. A new independent test
of ordinary signed and unsigned 32, 35 and 36 bit int fields
failed all 22 cells with v60k. Reusing the existing targetint
conversion function corrected all 22. Existing packed-bitfield,
negative width and wide multiplication guards passed 88/88.
Total acceptance: 110/110.

## v60m: integral one-word bitfield declarations

KCC fe56587; testkit 431be80. The declaration parser reuses the
existing signed/unsigned bitfield representation for integral
types of up to one word, preserving the declared width for
validation. Widths greater than the declared type, non-integral
types and long long fields are rejected. Enum, char, short
and long fields are verified, including static initialization,
volatile writeback, cross-word placement and target promotions.
The four original ordinary-field sources passed 68/68, and
bf-pack-1.c passed another 2/2. Total acceptance: 202/202.
The earlier experiment that compiled but failed readback was
not installed; v60l repaired its independently reproduced
constant conversion defect first.

## v60n: array member decay from non-lvalue aggregate values

KCC 443bb1c; testkit 985adab. In C99 array conversions, materialize
a non-lvalue struct or union operand in an ordinary automatic
temporary before computing its array member address. A sequence
expression retains side effects and evaluates the aggregate once.
No compiler node or type structure is enlarged. Generated
functions can need additional stack words for the temporary.
New semantic tests cover conditional, comma, assignment and
function-return expressions, nested members, discarded results
and sizeof. Original c99-array-lval-3.c: 4/4 PASS.
New semantic test: 22/22; existing guards: 42/42.
Total acceptance: 68/68 on a fresh replay after an interrupted
connection. Only the complete replay was accepted.

## Cumulative results

Original KCC compile_fail cohort: 2,196 cells. Previously excluded
as explicitly unsupported: 1,000. Recovered: 386 (312 through
v60k, 70 in v60m, four in v60n). Remaining: 810.
All 386 recovered cells replayed on the installed v60n compiler:
386 PASS, zero failures, no cached results.

Evidence directories under TMPDIR:
kcc-bitfield-v60l.NejYQu/mask-baseline-v1,
kcc-bitfield-v60l.NejYQu/mask-guards-v2,
kcc-bitfield-v60l.NejYQu/decl-guards-v3,
kcc-bitfield-v60l.NejYQu/packed-extra-v1,
kcc-array-rvalue-v60n.L8TCbn/guards-v2,
kcc-cumulative-v60n.05ce7l/replay-v1.
Exact recovered and remaining cell manifests are in the last
directory's parent.

Final compiler-only install archive:
kcc-array-rvalue-v60n.L8TCbn/install-v2/local.tar.gz
SHA256 5a8c66770c2e2e714c6330c41897de455061f167ed8864ac380097c33fd39959
Installed bin/kcc SHA256:
ed92922576bfcffe53931891dceded943fc8efc7fd50f59f67995f2a2ca1185e

No GCC or DAIMOS source changes. Native self-hosting and peak RAM
remain unmeasured; the cumulative replay is not a full matrix
sweep. Static address subtraction, terminal labels, wide bitfields
and over-aligned typedefs are outside the accepted series.
