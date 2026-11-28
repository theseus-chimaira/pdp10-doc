# KCC ordinary cross-word bitfield fragments (v61h)

Date: 2026-11-28. KCC 6ab0bd5, testkit 08005892.

Ordinary bitfields crossing a 36-bit word boundary already have an
exact compile-time bit offset (07300 offset marker). Generate one
native LDB/DPB per physical word fragment rather than one operation
per bit. Construct the initial S=1 byte pointer directly at the
known phase; do not call ADJBP for that constant phase. TLC changes
the known P/S fields while preserving the address. Advance one word
between the two fragments. Shift and combine the loaded fragments;
DPB consumes only its specified low source bits, so insertion needs
no separate fragment mask. Reuse the generated pointer register and
the first loaded fragment, avoiding redundant temporaries.

The saved lvalue address path copies the pointer before modifying it,
so compound assignment and increment/decrement retain their existing
single-evaluation semantics. Arbitrary logical S=1 pointers with
unknown runtime phase continue using one-bit stepping; the common
load/store path also avoids redundant accumulator initialization,
pointer copies and store masks. No changes to field layout, ABI,
compiler structures, or target runtime archives. The new fragment
helper is restricted to the existing one-word value widths. Fields
wider than 36 bits remain unsupported.

## Verification

The v61g baseline and candidate used isolated prefixes on charybdis.
Baseline and final candidate each passed all 22 configurations of
the new physical-word oracle, covering ten ordinary crossing layouts
and a side-effecting compound assignment/post-increment. Expected
raw words and signed values were computed independently of target
field operations. The registered code-generation guard requires
exactly two LDB and two DPB operations, with no IBP/ILDB/IDPB/ADJBP
or ADJBP helper, for the representative 12:32:28 layout. All eight
CPU/optimization combinations pass; the v61g baseline fails this
guard as expected. The final related guard run passed 678/678 cells,
including packed logical-pointer, aggregate and negative tests.

The complete 1042-cell historical replay retained exactly the v61g
failure keys and statuses: 760 PASS, 260 compile_fail and 22 run_fail
(historical bf-sign-2.c). No newly recovered historical cells or
regressions. Both v61g remaining manifests remain authoritative.

## Native and generated footprint

Same v61g compiler, PDP-6 native -Pgnu99 -O -m=gas,
HOST_DAIMOS=1, HOST_UNIX=0 and DAS -F -C. ccgen2 DOBJ text changes
26199 -> 26292 words (+93); data=3 and BSS=6 unchanged.

The standalone read/write probe is a pointer to a structure with
12-bit pad, 32-bit unsigned field and 28-bit tail. DOBJ text words,
including literals and any emitted helper:

| CPU | v61g opt | v61h opt | v61g noopt | v61h noopt |
| --- | ---: | ---: | ---: | ---: |
| PDP-6 | 328 | 25 | 397 | 31 |
| KA10 | 317 | 25 | 386 | 31 |
| KI10 | 317 | 25 | 386 | 31 |
| KS10 | 271 | 25 | 339 | 30 |

All probe data/BSS counts are zero. PDP-6 optimized assembly
instruction-line count changes 325 -> 27 (this textual count is
distinct from assembled text words). The reduction includes removal
of the software ADJBP helper. No general linked application size,
native phase execution, timing or peak RAM claim is made.

## Installation and next stage

Evidence relative to TMPDIR: kcc-field-fragments-v61h.L224IE
(oracle-baseline, oracle-candidate-v2, final-guards, final-replay,
probe, native, install-v1). Compiler-only local.tar.gz installed
into PDP10_PREFIX; bin/kcc verified byte-identical to candidate-v2.
Previous compiler retained in install-v1/previous-kcc-v1.

Installed bin/kcc SHA256:
9ecb1769717d6cf0ca1cef653182516550e99e59aca19f25be978be6685e8594

Compiler-only local.tar.gz SHA256:
989e273e6a4c1fe7fc52c2c27e0b21fe1e05f7407f2e3c19739f02b1fdc969e4

Multiword field support must extend the field type, static initializer
and value conversion paths together with extraction/insertion. Split
the physical stream at both 36-bit word and high36:low35 arithmetic
payload boundaries; preserve adjacent bits and normalize signedness.
Do not treat a raw physical pair as an arithmetic pair. Include
independent physical-word, signed boundary and aggregate-value tests
before accepting that change. The three historical wide-field
sources remain compilation failures. No GCC or DAIMOS source change.
