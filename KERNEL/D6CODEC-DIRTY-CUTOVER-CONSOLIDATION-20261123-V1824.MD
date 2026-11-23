# D6CODEC retired cutover and independent host codec verification

Date: 2026-11-23. Scope: source recovery and test consolidation, not
a new decoder implementation or successful full-guest device boot.

## Historical kernel cutover v1457 versus production v1458

The remaining dirty worktree on the v1452 baseline switched kernel
and five Stage-1 build paths to the experimental v1457 fixed core.
Its 56-word core (55 instructions and an inline LDB pointer) is
superseded by the current production v1458 decoder: 53 TEXT words,
zero DATA/BSS/relocations, fixed address octal 000060. The current
58-word VFS/direct-buffer adapter calls that one fixed core rather
than linking a second decoder. Compressed wire format is unchanged.
The untracked historical v1458 include was otherwise identical to
the current checked-in include, except for its stale 56-word comment.

The current `daimos-testkit/tests/validation/d6codec-fixed-v1458/
run-v1458.sh` component suite was rerun against current DAIMOS:

- Six successful native low-core decode cases and five malformed
  stream rejections.
- Eight valid VFS refill/direct cases, nine malformed inputs,
  five full-DXR validations and stack-canary high-water 52 words.
- Five Stage-1 native assemblies, saving 8 to 9 words per device
  compared with v1452, including DSK/DRM/DTC/MTC/PTR.

All passed. The historic cutover branch's 11 modified tracked files
and seven untracked experimental files were individually SHA256
checked against an archive, along with a retained Git patch and
base commit. Only these exact files were restored or retired.
The original detached worktree remains attached but is now clean.
Backup location:
`~/backups/daimos-d6codec-v1457-cutover-retire-v1822-20261123/`.

## Historical v1452 host/reference identity recovered as v1824

The old detached testkit also contained a host test comparing the
production C `d6block_compress_v1452` output, **word for word**,
against the separately implemented v1421 Python reference. This
is stricter than merely compressing and decoding the same input.
The original test assumed exactly 29 boot DXRs, a count which no
longer matches the current build. Its new version accepts the actual
boot inventory (requiring KCORE, KINIT and DSK Stage-1), the full
current DSH image, and eight deterministic pathological inputs.

`daimos-testkit/tests/validation/d6codec-host-reference-v1824/`
compiles the current host C codec into a temporary library, then
compares every resulting packed 36-bit word to the independent
Python reference. The source-built strict C decoder must roundtrip
the original words and reject malformed and trailing streams.

Result: **PASS, 31 real boot DXRs plus DSH plus eight stress cases,
94,276 decoded source words; seven bad-stream cases rejected.**
Testkit main commit: `5f62723`. Nine old experimental test files
were SHA256 archived and retired from the detached testkit worktree:
`~/backups/daimos-d6codec-legacy-testkit-v1824-20261123/`.
The historical v1458 native fixed-core and speed scripts were
byte-identical to the already tracked current testkit variants.

## Acceptance boundaries

This work did not rebuild a complete source-matched boot image or
retest DTC/MTC cold boots; those remain subject to their independent
device and memory-profile gates. The original v1459 integration
report remains authoritative for the full-image and physical results.
