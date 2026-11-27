# GCC wide bitfield arithmetic limbs (v60b)

GCC `4651ff3d` corrects extraction and insertion of 37..71-bit
integer fields in the default normalized 71-bit arithmetic mode.
A contiguous physical bitfield has no duplicated sign bit. The
arithmetic pair has high36:low35 payload and duplicates the high
sign in the spare low-word position. Previously the generic
36-bit split followed by a special signed shift attempted to
normalize extraction, while insertion retained the raw split.

The expmed.c paths now split at 35 payload bits, preserving the
actual physical field offset and adjacent bits. Extraction extends
the high piece with the field's signedness, extracts the low
payload unsigned, and adds the normalization bit. Insertion
copies the arithmetic input before any potentially overlapping
write and stores only the two payload pieces. The inline
pdp10_bitfield_sign pattern uses TLNE/TLO, no additional scratch
AC or literal. No target runtime function, helper, data table or
ABI change. Full physical pair copies and raw72 mode retain their
existing paths. The old compiler's raw72 test already passes;
changing the default mode would conceal the bug.

An initial field-only experiment had an invalid non-sign-extended
SImode RTL mask and failed recognition; this was corrected before
acceptance. A separate speculative output-side literal normalization
was rejected after it doubled the high limb: expr.c's INTEGER_CST
expansion already encodes source-level long long constants. Its
patch and failed test are preserved but never installed. This
repair does not establish a general solution to all wide scalar
constant folding, add/sub or arbitrary aggregate representations.

Testkit `a3e478d6` adds a generated independent physical-word oracle:
20 layouts (37,52,53,70,71 bits; signed/unsigned; first/last), 160
vectors, 320 extraction/insertion checks per guest. Exact expected
raw and normalized words are computed by Python, not target wide
arithmetic or the same bitfield lowering. Volatile raw limbs feed
the arithmetic inputs and capture outputs. Generator --check
verifies the checked-in source. Testkit `33689bf1` adds an independent
six-case static aggregate initializer check.

Final installed default-mode acceptance: original
pdp10-signed-bitfields-operations-v1.c 16/16, physical-word oracle
44/44, static initializer 44/44. Previous compiler fails the original
PDP6 O0 and the independent oracle at PDP6 O0/O2 (fail_id 052 octal).
Separate raw72 suite executes both original and new oracle at
PDP6/KA10/KI10/KS10 O0/O2: 16/16 PASS. The new oracle's PDP6 raw72
O0/O2 assembly is byte-identical to the previous compiler.
In the original PDP6 O0 output the four XOR function instruction
line counts change 48->42,48->41,46->42,48->39. No general linked
size, peak stack or timing claim.

The 612-cell GCC guard replay matches the latest applicable
checkpoints exactly: 508 PASS, 104 existing run_fail. The latter
are 44 each of historical 991118-1.c/bf64-1.c, 12 shifted
accumulator and 4 large structure return. The historical tests
compare signed narrow fields with positive 64-bit sign-pattern
literals in the wider PDP10 type. Keep source and registrations
unchanged; the native signed-value companion passes. The static
initializer oracle separately verifies contiguous field layout.
Combined 716 distinct default matrix cells: 612 PASS, 104 existing
run_fail; 16 additional raw72 guest passes. No new regression.
The legacy GCC host build emits existing warning classes.

Original 404 link-failure cohort remains 259 verified runtime
passes, 48 corrected registrations and 97 unresolved. No full
matrix rerun, native self-host, general volatile wide field access
count guarantee or extended runtime validation. Remaining priority:
packed-pointer array addressing (16 original companion failures),
then other compiler defects and console support. New target
runtime functions must be written in aggressively optimized
PDP10 assembly, prioritizing RAM and speed.

Evidence: `$HOME/tmp/gcc-wide-fields-v60b.YxmbAQDb`,
baseline-71/72, fields-v2-run, fields-v3-run, combined-experiment-v1.patch,
combined-v1-run, baseline-71-O0/O2 and candidate-fields-v3-71-O0/O2,
raw72-v3.log, targets-v3, static-v3, guards-v3,
guard-comparison-v60b.tsv, build-fields-v3.log.
Installed compiler-only install-v3/local.tar.gz SHA256:
`25e94f87cb0f7c31ffbaf281bcdf2c8a9b83a2e096dd41381a54b4cef5640abd`.
No runtime archive or DAIMOS kernel changes.
