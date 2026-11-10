# v1464: single-format block LZ/RLE production cutover

Date: 2026-11-10.

The explicit production decision supersedes the earlier v1395/v1462
deferral: **D6LZ36 has been removed**, not retained behind a runtime
compatibility branch. Production compressed boots, user executables,
PURE backing and TSFS restart extents now use the existing canonical
v1421 block format. Disk and drum swap records remain raw and
uncompressed. The historical development branches, experiments and
design documents remain available in Git; they are not linked fallbacks.

## Decoder and execution ABI

- One 53-word v1458 fixed low-core decoder, located at octal 000060,
  starts each KCORE TEXT image and is embedded byte-for-byte in all five
  Stage-1 loaders (DSK, DRM, DTC, MTC, PTR). The inline `LDB` byte
  pointer is included in the exact 53-word identity check.
- One 58-word resident VFS/direct-buffer frontend,
  `d6codec_decode_vfs_v1458` / `d6codec_decode_buffer_v1458`, handles
  bounded compressed extents and window refill. The core uses zero
  additional static DATA/BSS; the frontend's temporary call frame is
  39 words, including the 32-word refill window.
- `exec_load.s`, the optional bounded FULL parser, SWAP/PURE reload and
  the TSFS restart reader use the block decode entry points. The later
  checked BACKSTORE and MEMFS ownership corrections remain intact.
- The source-matched `d6block-v1464` host build uses the v1452 encoder
  and is the default boot compressor. The current `mktsfs-v1461` and
  `tsfscheck-v1461` build from the same block-codec sources and validate
  actual regular-file extents; stale installed tools are not trusted.

The obsolete `system/kernel/core/d6lz_pdp6.s`, `d6lz.h`, Stage-1
`decompressor.inc`, host `d6lz.c` and `d6lz-codec.[ch]`, and the
userland D6LZ command/manual were removed. The host tools installation
also uninstalls an older `d6lz` executable and installs the block
encoder and TSFS producer/checker in `$PDP10_PREFIX`.

## Verified acceptance

1. Native PDP-6 v1458 fixed-core valid streams, corrupted inputs, VFS
   refill and direct-buffer paths, and stack-canary checks pass.
2. All five Stage-1 images assemble and include the same exact 53-word
   KCORE decoder. Relative to the earlier v1452 block implementation,
   the read-in sizes drop by 8-9 words per device.
3. A fresh complete 256K KCC DSK/D6FS image builds with block-only
   KINIT and compressed executables. Isolated real SIMH cold boot
   reaches ROOT LOGIN, DSH and an external compressed ECHO command:
   **PASS**.
4. Source-matched TSFS media validates **53 files and 401 extents**.
   Independent block decoding reproduces LOGIN (3221 words, 13 extents),
   DSH (15760 words, 62 extents) and ECHO (1035 words, 5 extents) exactly
   against their linked DXRs. The native PDP-6 decodes the actual final
   LOGIN restart of 149 words from a 64-word compressed stream: **PASS**.
5. The existing source-linked v1405 checked swap-in refusal, MM rollback,
   retry and ignored-result negative-control tests pass with the new
   decoder selected.

6. An independent, isolated **256K TSFS-root cold boot also PASS**:
   ROOT LOGIN, DSH and a real external compressed ECHO returned to
   the shell prompt with the expected output. A first run timed out
   after a 240-second command allowance while the sampled PC was octal
   030775, in `dtc_search_wait` inside the DECtape block-search driver,
   not inside the block decoder. A second, otherwise identical run
   completed with a 480-second command allowance. This bound tolerates
   the observed physical DECtape seek latency, but the test still
   requires the completed external command and returned DSH prompt;
   no decoder or kernel source was modified between the two runs.

The testkit's `inspect-exec-v1465.py` provides an isolated-media SIMH
register and history probe for the slow tape-read path. The first
snapshot demonstrates where the processor was executing at that instant,
not proof that the entire delay was exclusively caused by tape motion.

## Measured RAM cost and outstanding gates

The matched GCC lowmem disk-root KCORE changes from **9687 to 9701
resident 36-bit words**, +14. The movable SWAP MRES remains **1762
words**, and full linked KCORE+MRES inventory changes from **16367
to 16381**. The strict 32K single-root permanent-address test still
fails by 45 words; there is no new static codec BSS. Optimization must
preserve bounded decoding and the source-linked ownership invariants.

Real DTC/MTC *Stage-1* cold boots, an accepted 48K runtime, strict 32K
placement, and a fresh 256K F07 physical swap/OOM postflight with block-only
executables are separate remaining acceptance requirements. The older
v1406 full-guest result predates this incompatible format cutover and
must not be reported as a new v1464 test.
