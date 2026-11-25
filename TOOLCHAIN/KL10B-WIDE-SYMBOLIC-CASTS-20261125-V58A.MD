# GCC wide symbolic pointer casts and native displacement limits (v58a)

GCC f2a2597, testkit 81b6c63; based on v57zb. DAS, dlink,
emulators, DAIMOS and production installation remain unchanged.

## Repair

The wide CONST(UNSPEC_GLOBAL_RELOC) introduced in v57x stores a symbol,
selector and signed displacement independently. The conversion routines
previously decoded only the compact symbolic representation. As a
result, wide static casts could retain the wrong selector. The global
pointer and void conversion helpers now explicitly decode wide constants,
apply the existing selector mapping and reconstruct the relocation with
its original displacement. The ordinary native pointer conversion and
internal conversion entry points also recognize wide constants, using
the existing native conversion policy. No dynamic conversion table,
pointer storage or new target runtime code is introduced. Local-address
behavior is unchanged.

## Acceptance

The new tools/test-gcc-global-wide-cast-v58a.py runner independently
models physical bit positions and the established selector rules for
native 6/7/8/9/16/18/32/36 and packed exact16/32 source pointers.
Displacements on both sides of the former signed addend range are
exercised, including values near the architectural limits. Conversions
through void, direct exact-width conversions and aligned native casts
are tested. The source uses explicit representation probes rather than
dereferencing misaligned or synthetic high addresses. Ordinary native
subbyte direct conversions retain their legacy rules; arbitrary
misaligned native-to-native casts are not accepted by this stage.

At each of O0/O1/O2/Os, 3106 positive and 3106 negative static
relocations and their linked words pass (24848 each). The v57zb O0
baseline emits 2244 incorrect static words per sign (4488 total).
Each of eight profiles also verifies 91 distinct required wide code
relocations (728 total). Positive function-return comparisons execute
successfully on SIMH and KLH10 at all four optimization profiles;
negative high DATA placements are verified only at object/link level.
The runtime probe compares the raw pointer words rather than relying
on void-pointer equality. No high synthetic address is dereferenced.

Native extreme displacement controls verify both signs of 2^30-1
for all eight scalar widths at four optimization profiles (64 exact
relocation objects). The corresponding +/-2^30 displacements are
explicitly rejected (64 controls). These are displacement magnitude
checks, not claims that every symbol placement will link.

Replays: previous void/mixed runtime (16), wide native runtime (8),
wide packed runtime (8), wide native object/link (9888 each), wide
packed object/link (14472 each), packed extreme/rejection (16 each),
and earlier static pointer objects (4 profiles) pass. All 56 extended
and 112 local assembly guards are byte-identical to v57zb. Inventory:
2400 unchanged matrix sources, 450 special entries, zero unowned
sources.

Evidence: pdp10-testkit/manifests/PDP10-GCC-GLOBAL-WIDE-CAST-20261125-v58a.tsv
(138 observations: 8 cast profiles, 2 baseline controls, 64 native
extreme objects and 64 native rejection controls). Staged GCC:
TMPDIR/pdp10-gcc-wide-cast-v58a/gcc. cc1 SHA256:
e563717bd09fd36e8f95759b158aec9082f5e594109f75c538198d37b72418b9.
Use staged DAS v56y and linker v56m. No installation performed.

## Remaining scope

Direct aggregate construction, indirect address consumers, arbitrary
misaligned native-to-native conversion semantics and other combined
expressions remain open. This stage does not resolve the emulator
ADJBP discrepancy documented in v57zb. Aggregate strides, code size
optimization, extended runtime, multiple TEXT placement, KCC and
mixed-compiler ABI remain open. Preserve the historical compiler
matrix until extended work is complete.
