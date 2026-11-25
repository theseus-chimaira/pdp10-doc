# GCC wide native symbolic relocations (v57zb)

GCC 7988927, testkit 7aaa659; based on v57x. DAS, dlink,
emulators, DAIMOS and the production installation are unchanged.

## Representation and repair

The independent symbol/selector/signed-displacement RTL from v57x
now also serves native ADJBP constant simplification and symbolic
word-pointer adjustment. Native coordinate arithmetic uses one
host calculation for absolute and symbolic operands; only absolute
coordinates wrap. Word-valued constant expressions in EXPAND_SUM
and EXPAND_INITIALIZER use the same relocation constructor before
generic plus_constant can erase the displacement distinction.
The static native byte-pointer initializer no longer adds a second
selector to a CONST(UNSPEC_GLOBAL_RELOC) operand. Negative compact
native code literals now use %GADDR with a separate signed addend
rather than the legacy P/S decoder's invalid raw GIW sum.

Large constant native symbolic adjustments use a relocated literal
rather than unnecessary runtime ADJBP. The conservative threshold
is a magnitude of at least 01000000 in the element count or current
word displacement; ordinary short adjustments retain their existing
code generation. This is a compile-time optimization, not a change
to the dynamic ADJBP implementation. The architectural pointer
word and object format remain unchanged. No new runtime table.

## Acceptance

The new tools/test-gcc-global-wide-native-symbolic-v57y.py runner
uses independent native slot coordinates for widths 6/7/8/9/16/18/
32/36. It tests every start/end slot across four positive and four
negative displacements, addition, subtraction and chained adjustment.
All O0/O1/O2/Os profiles pass 1236 relocations and linked words per
sign, totaling 9888 of each. All 192 required distinct wide code
relocations are checked before and after linking. Positive placements
execute 96 function-return comparisons per profile on SIMH and KLH10
(eight passing executions). Negative placements are synthetic high
DATA link probes and are not executed. Neither group dereferences
synthetic high pointer values. v57x rejects both wide translation
units at O0. The tested wide displacements include values on either
side of the old signed limit and near both architectural endpoints;
exact extreme magnitudes and impossible native addends remain to test.

Earlier native symbolic checks (4944 relocations and linked words),
wide packed symbolic checks (14472 relocations and linked words,
64 distinct code checks, 16 extreme objects, 16 rejection controls,
8 runtime profiles), native absolute constants (73400 words),
static-pointer objects (4 profiles) and native boundary executions
(8 profiles) pass again. All 56 extended and 112 local assembly
guards are identical to v57x. Inventory: 2400 unchanged matrix
sources, 449 special entries and zero unowned sources.

## Emulator-dependent observation (not a resolved compiler defect)

The intermediate v57z build emitted ADJBP with a correct six-bit
pointer and large count instead of folding one in-range displacement.
For pointer 0460002001000 and count 027777777777 (octal), the host
coordinate oracle expects 0534002000777. SIMH returns 0530002000777;
KLH10 passes the same linked diagnostic image including the remaining
function-return checks. The observed difference is address bit 29.
The dynamic boundary suite still passes on both emulators; this new
case must not be counted as a passing dynamic test. The accepted
compiler folds known symbolic adjustments; it neither modifies nor
claims to repair the emulator-dependent dynamic result. No emulator
implementation work is in scope. Diagnostic evidence is preserved
in the manifest with the intermediate compiler hash and both logs.
The actual image is in TMPDIR/pdp10-wide-native-symbolic-v57y-9a4x7ve0/
positive-O0 (SIMH and klh10-diagnostic-v57zb logs).

Evidence: pdp10-testkit/manifests/PDP10-GCC-GLOBAL-WIDE-NATIVE-20261125-v57zb.tsv
(8 passing object/link profiles, 2 baseline rejection profiles,
1 emulator-dependent diagnostic). Staged GCC:
TMPDIR/pdp10-gcc-wide-native-v57zb/gcc. cc1 SHA256:
833ae32f6928cdeed64848969ca41e6aae2bf5b322390e4e3cbb587ea25e4138.
Use staged DAS v56y and linker v56m. No installation performed.

## Remaining limitations

Wide casts, direct aggregate address construction, indirect address
consumers, native extreme/impossible displacement controls and other
combined expressions still need validation. The new helper is not a
general proof of all symbolic address consumers. Additional aggregate
layouts, resident-size optimization, extended runtime, multi-TEXT
placement, KCC and mixed-compiler ABI remain open. Preserve the
historical compiler matrix.
