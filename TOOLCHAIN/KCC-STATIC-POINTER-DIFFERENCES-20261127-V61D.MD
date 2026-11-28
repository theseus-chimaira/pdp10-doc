# KCC constant pointer casts and static pointer differences (v61c-v61d)

Date: 2026-11-27.

## v61c: raw pointer constants

KCC 0e1b99a, testkit 37b4ae7. The constant folder now matches
the runtime unsigned raw-word conversion between pointers and
normalized 71-bit integers (high36:low35). The high pointer bit
is preserved, widening/narrowing does not duplicate payload bits,
and top-level pointer qualification alone does not change the
raw constant. Nonconstant symbolic pointers are not folded by
this path.

The original regression failed 22/22 on v61b. The final candidate
passed 110/110 (five semantic sources). An intermediate candidate
failed 22/110; this was corrected before installation. Fresh
historical replay on installed v61c: 606 PASS, 436 compile_fail,
zero cached cells and no new regressions. Native PDP-6 cceval
cross-compiled and assembled; complete native phase execution
was not attempted.

## v61d: same-object static pointer differences

KCC 6f61d0a, testkit 0f7ce82. In GNU/KCC mode, fold pointer
subtraction only when the static symbolic bases match and the
native physical offsets can be represented exactly. Coordinates
are measured in bits from the high end of a word. The difference
is divided by the pointed-to object stride; no encoded byte
pointer words are subtracted and no relocations are discarded.

The walker uses existing member P/S offsets, array types and
sizeptobj. It accepts addressable static objects, decayed static
arrays, native member selections, exact pointer casts and bounded
constant index arithmetic. A null base is accepted only for a
member-derived offsetof-style difference. A nonzero raw pointer,
loaded pointer value, unrelated object, bitfield, packed/ambiguous
pointer representation, VLA or non-evenly-dividing native byte
size is not folded by this path. GNU label differences and
separate string literal identities were open in v61d. Identical literal
payloads and exact coordinates are supported as of v61k (see
KCC-LITERAL-DIFFERENCES-20261128-V61K.MD). No new permanent
compiler table, type field or node field.

A new semantic regression compares independent literal offsets
with runtime pointer subtraction, including nested fields,
halfwords, array members, multidimensional arrays and negative
differences. Two negative sources verify unrelated static
objects and loaded pointer values remain rejected. The first
candidate recovered four original sources but the additional
multidimensional guard failed compilation. The corrected walker
handles the parser's array-typed subscript addition; final
guards also cover the original array-member layout.

Original baseline of new guards: 44 negative PASS, 22 semantic
compile_fail. Final candidate: 176/176 guard PASS (110 previous
v61c and 66 new). Full exact active historical replay: 1042
executed, 694 PASS, 348 compile_fail; all 606 prior passes retained,
88 newly recovered, no cache, ICE, link, run or timeout failures.

Recovered originals (each 22): 920928-1.c, 920928-4.c,
920928-6.c, 930326-1.c. Remaining constant initializer cases:
20001116-1.c (literal difference), labels-3.c (label difference).
Neither was marked unsupported. v61k recovered 20001116-1.c;
labels-3.c was recovered in v61o using assembler-resolved same-function
label differences; see KCC-STATIC-LABEL-DIFFERENCES-20261128-V61O.MD.

## Native footprint and remaining work

Same installed v61c compiler cross-compiled before/after cceval
for PDP-6 -Pgnu99 -O -m=gas, HOST_DAIMOS=1, HOST_UNIX=0.
DAS -F -C assembled each. DOBJ headers:
text 6431 -> 7138 (+707 words), data 0 -> 0, BSS 2 -> 2.
The code increase is material. Peak recursive stack use,
execution speed, linked KPARSE size and native self-host
execution remain unmeasured.

Original 2196 KCC compilation failures: 1154 documented
unsupported, 694 recovered and 348 active remaining (21 sources).
No new scope exclusions. The exact remaining replay selection is
pdp10-testkit/manifests/PDP10-KCC-COMPILE-REMAINING-20261127-v61d.tsv.
Three stale native PDP-6 error registrations (920829-1.c,
bf-layout-1.c, bf-pack-1.c) returned to semantic; both optimization
cells of each passed the fresh replay. Matrix remains authoritative
for registration; results remain in per-cell checkpoints.

## Evidence and installation

v61c: TMPDIR/kcc-constant-pointer-cast-v61c.jZTEgX
(guards-v2, baseline-v1, install-v1/local.tar.gz).
Final native object: TMPDIR/kcc-v61c-native-final.VTeARm.
Installed v61c binary SHA256:
4664c3299f2043e28cc163dd7a9ce0356796d4bc5251fc99ae1bd93d0069fc26

v61d: TMPDIR/kcc-symbolic-difference-v61d.UcBrAo
(baseline-v1, guards-v4, full-v4, native-before, native-after,
install-v1/local.tar.gz). The archive was installed to
PDP10_PREFIX; the installed binary matches the tested staged
binary byte for byte. Compiler-only archive SHA256:
e3801881ace5e279d0674e97b65e123e14a7f4c479ef7a4d6d949c68c3a90c46
Installed bin/kcc SHA256:
b0a5e2c5e7641aeacc73b1ccac1cb0c6c09b61d432226e6335ee516ec5d23f0a
