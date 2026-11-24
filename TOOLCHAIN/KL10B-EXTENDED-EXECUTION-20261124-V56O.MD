# KL10B initial C execution with global addresses (v56m/v56n/v56o)

Scope: host-only C toolchain validation. No DAIMOS integration and no
changes to the existing COMPILER_MATRIX.tsv. Dedicated testkit tests
run on SIMH and optionally KLH10. The historical local failure ledger
is unchanged. Production compiler and tool binaries are not replaced.

## Host linker and image contract

PDP10-TOOLS commit 93c3a58 adds an explicit host-only mode:

```
dlink --extended 1001000 2001000 3001000 -o program.p10x -M program.map objects...
```

The three arguments are octal virtual bases for TEXT, DATA and BSS.
Each nonempty range fits within a single nonzero hardware section;
ranges must be disjoint. The current profile places all input TEXT in
one section. Arbitrary multiple TEXT placements are not yet supported.
Ordinary archive resolution now recognizes ADDR30-only references.
DOBJ1 and DOBJ2 inputs are accepted, but relative RH18/LH18 relocation
must remain in the relocation word's hardware section. A global
ADDR30 relocation applies the signed addend to the full virtual
address, checks 30-bit overflow/underflow, and preserves the top six
bits. Legacy DXR, purity, MRES, fold and monitor-specific options are
not combined with this mode. No DXR format or loader is changed.

P10X1 is a fixed-address sparse test image. All words use the same
8-byte little-endian 36-bit container as DOBJ. The header is eight
words: SIXBIT `P10X1 `, entry address, then three pairs of (virtual
base, word count) in TEXT/DATA/BSS order. Initialized TEXT and DATA
follow in that order. BSS is zero-filled and has no stored words.
There is no relocation bitmap; all addresses are already resolved.
The linker relocates the existing object buffers in place rather than
constructing a dense virtual-address-space image.

The testkit loader validates these ranges and maps only the needed
physical pages, with guest CONO PAG and XJRSTF startup. The current
fixture supports sections 0..37 octal, 256K physical words, and a
one-page stack at 4,,1000. This is a test fixture, not an OS loader.
The startup checks the precise SIMH HALT instruction address, AC1
return, AC6 completion sentinel, AC17 restoration and observed guest
memory. KLH10 independently checks termination and the same words.

## Fixed defects

- DAS cc5f3b1: XMOVEI was encoded as opcode 0201 (MOVEI). The correct
  opcode is 0415. `XMOVEI 17,2(17)` now emits 415757000002. The
  old opcode discarded the section in compiled stack-frame prologues.
  The corrected table entry remains marked non-base, with no new
  resident table words.
- GCC aea4633: do not route extended native HImode accesses through
  the local S=1 packed-bit-stream helper. OWGBP uses a different
  selector encoding, and the local helper's IOWD stack adjustment
  corrupts a global stack. Native extended halfwords use LDB/DPB
  and the existing sign-extension path. Exact-width global packed
  pointers remain an open representation/ABI requirement.

## Acceptance

Testkit a9502da registers `kl10b-extended-link-v56m` as an optional,
manual validation. The test links an archive through global-only
references and executes it successfully on both emulators. It checks
positive and negative ADDR30 addends, a section carry, rejection of
cross-section RH18, 30-bit overflow and underflow, overlapping or
oversized ranges, undefined symbols and incompatible legacy options.
A representative local DXR output is byte-identical to the preceding
linker. The DOBJ2 suite passes all 26 selectors and seven mixed
relocations, malformed input guards and three local DOBJ1 comparisons.

`gcc-global-execution-v56m.c` passes O0/O1/O2/Os on SIMH and KLH10
(eight runtime observations). It uses native byte/halfword/word
pointers, static offsets, BSS, indirect function invocation, and a
global stack. It links the existing freestanding `__main` member;
this does not establish that the full libgcc archive is extended-safe.
The v56i compiler fails O0 with the corrected assembler and linker,
isolating the halfword defect. The preceding assembler separately
fails the exact XMOVEI encoding check. The v56i static pointer object
test still passes all four modes. Sixty-four local compiler assembly
comparisons are byte-identical (four sources, four CPU targets, four
optimization modes); these are codegen comparisons, not new runtime
results. Inventory audit passes.

Evidence: `manifests/PDP10-KL10B-EXTENDED-EXECUTION-20261124-v56o.tsv`
records ten independent runtime observations with image/log/binary
hashes. Staged binaries (under TMPDIR):

- `pdp10-das-xmovei-v56o/{das,das2}`
- `pdp10-extended-link-v56m/dlink`
- `pdp10-gcc-global-halfword-v56n/gcc` (cc1 override)

The previously staged extended record-array work (v56k) is preserved
on top of aea4633 but not accepted. Its compile/assemble recovery
requires runtime validation and remaining integer multiply/pointer
subtraction repairs. Arbitrary TEXT placement, full extended runtime,
packed types, and KCC extended mode remain open. No new compiler
matrix profiles or historical runtime recoveries are claimed.
