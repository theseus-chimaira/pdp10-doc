# Wide floating conversion ABI and unsigned casts (v59s)

KCC 36f3a4d, 49fc2b1; GCC 1693dde9, 7e49ff3e;
testkit 1cdfc7a3. Follow-up to v59r.

The portable __kcc_floatdidf and __kcc_fixdfdi routines used
AC10 for the sign flag without preserving it. Both compiler
ABIs make AC7 volatile and AC10 preserved. Substitute AC7;
no added instruction, stack word, or writable static storage.
Apply the identical correction to the KCC source and mirrored
GCC freestanding source. Both independently rebuilt core C
archives compare byte-identical (14232 host bytes, unchanged
size, new hash). Historical SHA-frozen evidence remains intact.
The old archive fails the direct oracle at call 3 (first
nonzero input); the corrected archive passes 89 calls per CPU
on PDP-6, KA10, KI10, KS10, preserving AC10..AC16 and SP.

The signed SF and DF libgcc magnitude conversions now also
export __floatundisf and __floatundidf. AC7 distinguishes the
unsigned entry and later records a negative signed result.
The unsigned high word is not interpreted as a negative
integer. Both entry points share normalization and rounding,
including guard/sticky bits across the 71-bit input. The
signed and unsigned results use the same floating ABI.
Remove the old AC10 push/pop: both affected routines have
unchanged static instruction counts (56 SF, 77 DF) and one
fewer saved stack word. Host libgcc.a size 41448 -> 41720
bytes; this is archive metadata/serialization size, not a
measured increase in linked guest memory. No extra member
or persistent storage is needed.

KCC routes all wide FP casts through pair-aware helpers.
Integer-to-SF uses the direct shared libgcc conversion for
both signs, including PDP-6; the former signed PDP-6 D-format
intermediate could double-round. Unsigned-to-DF uses the new
entry; unsigned SF/DF-to-integer uses the existing fixuns
entries. Signed PDP-6 DF conversions retain the repaired
KCC-specific pair routines. The ABI-call prepass recognizes
all wide casts and preserves live incoming arguments. Results
from integer-fix helpers have their redundant low sign bit
removed for KCC's internal representation. No GCC front-end
or optab lowering changes; the new unsigned entry points are
used by KCC, not a claim about all GCC unsigned C casts.
Standalone KCC links using unsigned-wide or direct signed SF
casts now require these genuine libgcc entries, including on
PDP-6. No OS libc or test stubs required.

Exact integer oracle test-wide-float-abi-v59s.py computes
independent target bit patterns for signed and unsigned 71-bit
integer-to-SF/DF round-nearest/ties-even and representable
inverse conversions. It checks all preserved ACs and SP on
every call, including limits, guard/sticky ties and seeded
random values. Compiler-only archive linkage. 437 calls per
CPU, 1748/1748 final PASS (includes the original 356-call
preservation check). The new C99 unsigned-cast regression
uses explicit volatile limbs and fixed independent SF/DF
patterns, unsigned high-bit boundaries and fractional
truncation; reverse cases that round outside 71-bit range
are intentionally not executed. The old KCC rejects it with
'vrwiden: reg already wide' on KA10 noopt; all 22 new cells
pass. Signed v59r oracle also 22/22. Both existing PDP-6
software D-format/native SF special suites pass opt/noopt.

Final combined matrix guard replay: 356 cells, 328 PASS and
28 pre-existing run_fail, no new failures. GCC 184/200,
KCC 144/156 including the 22 new unsigned-cast cells. Every
previous pass preserved (184 GCC + 122 KCC). Historical
GCC stages 32/32, raw shared ABI 28/28, full-word/narrow
integer conversions 132/132, scalar FP guards 64/64,
signed/unsigned KCC cast oracles 44/44. Original conversion
and line companion retain 28/56 PASS. Installed freestanding
parity, selector and conversion-scope audits all PASS.

The original 404-link cohort remains at 128 confirmed PASS,
24 invalid registrations corrected, 252 unresolved/unverified;
this repair addresses additional compiler defects rather than
reclassifying historical link cells. No full-matrix, extended
section, raw72, native self-host, target timing or exhaustive
floating-point representation claim. Out-of-range float-to-
integer conversions are not part of this acceptance.

Evidence: `$HOME/tmp/pdp10-wide-float-v59s.EYG0ilpX`,
abi-before.log, abi-v1.log, abi-v2.log, before-cast/compile.log,
native-v2, guards-v2, build-core.log, build-libgcc.log,
build-kcc-v2.log. Installed incremental archives SHA256:

* local.tar.gz (mirrored core source/archive):
  `1a0d808903f2f8a12212053bc2c2fe7f35677cf603787e56fcaee14eea1d5d27`.
* libgcc-v2/local.tar.gz:
  `cb737271d9feaa1c52227d06c1ea95d058772c24a98eb16288fd3622a49aee34`.
* kcc-v2/local.tar.gz:
  `a0dbbb77a9410cfde62f4bc401773e2d13556f9a99649f4153301cc24f59ee03`.

Installed core archive SHA256
`a76e9515d70d365b6c1c8f96a5642b58713128400b7cbf5b3667d511c22bb26c`;
installed libgcc.a SHA256
`146472ccdf554efbce5df07aad61cc5f9c6d76c4c51eeaf8197b3020afcf3a77`.
