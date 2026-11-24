# KL10B global relocation and GCC static pointer repairs (v56h/v56i)

Host C toolchain only. Local DOBJ1 and DXR formats remain unchanged;
extended image linking is not yet implemented. No new resident
assembler object-mode state and no DAIMOS changes.

## DOBJ2 contract

DAS b71ffa6 and PDP10-TOOLS 876750b introduce DOBJ2. The header,
sections, symbols and three-word relocation record layout retain
DOBJ1 encoding, with distinct SIXBIT magic `DOBJ2 `. Relocation type 5
is local section-base ADDR30; type 6 is symbol ADDR30. Low 30 bits of
the stored word are zero pending relocation. The high six control or
selector bits survive. The signed 36-bit addend has magnitude at most
07777777777. The future extended linker must check the final sum fits
30 bits, without carrying into the selector. RH18/LH18 retain their
existing encoding. Legacy DXR linking and the local optimizers reject
DOBJ2, including selected archive members; darc preserves the version
and objdump identifies it. DOBJ2 reader validation rejects malformed,
out-of-range, overlapping relocations and nonzero relocated fields.

GIW and OWGBP accept octal expressions. OWGBP selectors 045 through
076 are preserved verbatim; EXIND retains the existing expression
radix. Symbolic global words require host DOBJ2. Base mode rejects
global forms. Local byte pointers use POINT, not OWGBP.

## GCC v56i

GCC 3248cbc selects the global representation in constant native
word/byte conversions, emits explicit GIW/OWGBP for symbolic data,
corrects the signed word carry in global ADJBP folding, and preserves
EXPAND_INITIALIZER while evaluating nested static pointer casts.

The v55c baseline fails all four optimization profiles of the new
static-pointer fixture. The staged v56i compiler passes O0/O1/O2/Os:
13 static pointers, null pointer in BSS and one code literal per
profile. This is object-generation validation, not extended C runtime.
The extended record-array/switch ICE at expr.c:9362 remains open.

## Evidence

Testkit 7b9d3e5 registers dobj2-global-address-v56h and
gcc-global-static-pointer-v56i. All 26 selectors, seven mixed
relocations, archive preservation, rejection by local consumers,
malformed input and three unchanged DOBJ1 fixtures pass. The corrected
local-format golden fixtures use local pointer expressions. The DAS
fast suite passes. SIMH and KLH10 independently pass 92 global ISA
assertions and 20 physical memory observations.

The combined v56i staged toolchain passes 520/520 local runtime guards.
All 520 generated assembly hashes match v55c. For 220 cells with
archived object-size evidence the total words also match individually.
No measured speed or resident RAM claim is made. The two manifests
PDP10-GCC-GLOBAL-INIT-20261124-v56i.tsv and
PDP10-GCC-GLOBAL-INIT-LOCAL-GUARDS-20261124-v56i.tsv record provenance.
The staged cc1 SHA256 is
38421740e19673d5d80a507699b99c8f23a75525fd5dc145a215f1bf8b448bb0.
Production tools remain unchanged; no new extended runtime recoveries
are counted, and KCC extended mode remains unvalidated.
