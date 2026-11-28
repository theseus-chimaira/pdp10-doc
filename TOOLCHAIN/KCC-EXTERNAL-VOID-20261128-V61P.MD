# GNU incomplete external void linker symbols (v61p)

KCC 0bf0733, testkit 41a8e71e. GNU/KCC mode accepts explicitly
extern-declared void symbols without an initializer, preserving their
incomplete type and allocating no object storage. Strict C89/C99
retain the prior rejection. Void definitions, arrays, initializers,
incompatible redeclarations, and use as a value remain rejected.
Address-of accepts the GNU void identifier lvalue. Both static and
runtime generation use a nine-bit byte pointer to the first byte
of the linker symbol's word, matching the GNU label-address convention.
This avoids division by zero in the old single-byte address path.
Discarded void identifiers, including volatile-qualified ones, have
no object value to load. No new runtime helper, relocation kind,
persistent metadata or data/BSS storage is introduced.

## Verification

Evidence: TMPDIR/kcc-external-void-v61p.YM9Gxf.
- Registered test kcc-external-void-v61p: 16 linked executions pass
  (four CPUs, GNU89/GNU99, optimized/unoptimized). Assembly-defined
  symbols anchor/end occupy adjacent words with independent contents.
  Checks static and block-extern runtime pointers, +4 C-byte offset,
  pointer difference, casts and actual word reads, sizeof/alignof,
  no .LINK, and discarded-void versus empty function code equivalence.
  All 88 negative declaration/initializer/mode checks pass. Baseline
  probe rejects the formerly unsupported declaration.
- Prior special guards: void indirection 36 checks, static label
  differences 72, scalar sizes 224; all pass.
- Fresh historical 1042-cell replay: 892 pass, 84 compile_fail,
  66 run_fail. All 150 exact failure keys unchanged. All 44 cells of
  20010327-1.c and 20011114-1.c now report only the remaining integer
  constant initializer error; the void declaration error is gone.
  v61o historical manifests remain authoritative.
- Fresh broad 3422-cell replay: 3183 pass, 180 compile_only,
  44 run_fail, 2 link_fail, 13 policy_fail, zero compile_fail.
  All 59 exact failure keys unchanged from v61o. No changed exclusions
  or classification policies.

## Full-word relocation boundary (not implemented)

Both historical sources need conversion of a symbolic pointer into
an integer and subtraction of 0x10000000L and 1. The local C pointer
representation occupies all 36 bits. Integer arithmetic after the
cast is not 18-bit effective-address arithmetic. DOBJ1 preserves a
fixed halfword and relocates the other half independently; it cannot
express a general carry or borrow between halves. The existing
DAS .word syntax for a surviving relocation is an RH18 contract,
not a full-width arithmetic relocation.

Evidence in relocation-audit-v1: assemble a data word containing
`anchor+0777777` with anchor defined in another object. Link at base
020; anchor is at 022. The mathematically valid full word is
000001000021, but both installed dlink and the tree dlink reject the
RH18 overflow. The tree diagnostic identifies type 2, target 022,
addend 262143. The installed and tree linker binaries have different
hashes; neither was modified. A non-carry high-half adjustment and
one borrow probe happen to produce the expected word at this placement.
These are not proof of general support. The first audit misindexed
TX2 payload using a two-word instead of three-word header; corrected
comparison confirms the matching non-carry case. Retain the source
rejection instead of generating an unsound RH18 initializer.

A complete solution needs explicit full-word relocation semantics,
including symbolic/local identity, signed/modular addends, carry and
borrow, object validation, archive/relink/optimization consumers and
native linker support. For a relocatable load image, loader relocation
metadata must also distinguish full-word addition from RH18 addition;
a fixed-base link alone does not establish that runtime contract.
DAS has host DOBJ2 address30 kinds 5/6, not full-word arithmetic;
the current daimos-tools source exposes DOBJ1 kinds 1..4. Reconcile
format support before choosing a new encoding. This feature was not
implemented in v61p; no fabricated compilation passes or workaround.

## Native footprint and installation

Same baseline cross-compiler builds both revisions for PDP-6,
-Pgnu99 -O -m=gas -DHOST_DAIMOS=1 -DHOST_UNIX=0. DAS DOBJ headers:

| Unit | Before text | After text | Delta |
| --- | ---: | ---: | ---: |
| ccdecl | 14206 | 14225 | 19 |
| ccgen | 5453 | 5471 | 18 |
| ccgen2 | 26836 | 26839 | 3 |
| ccstmt | 9396 | 9400 | 4 |
| Total | 55891 | 55935 | 44 |

Data and BSS unchanged. No linked native phase size, native
self-host execution, peak dynamic RAM or execution timing measured.
No GCC, DAS, linker or DAIMOS source changes.
Compiler-only install-v61p/local.tar.gz installed into PDP10_PREFIX
and compared identical with the tested compiler. Previous binary
retained in install-v61p/previous-kcc-v61o.

- bin/kcc SHA256: ec5b22b5148450a629748147ce38bf01f82b1992295b3c5599754827c4a72db6
- local.tar.gz SHA256: 21b5e32b4c6c53c12dd2196aeae0426c0a7472e934a5d64880bb75cb84b311da
