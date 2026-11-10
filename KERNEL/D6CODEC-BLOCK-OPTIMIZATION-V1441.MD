# Block file decoder optimization v1441

Date: 2026-11-10. Completed **native PDP-6 component optimization**
and validation. Not yet an installed KCORE/Stage-1 decoder or
boot-media format migration.

## Decision and wire format

The compact v1421 block LZ/RLE wire format remains unchanged
through v1441. A bounded stream contains literal blocks, zero
runs and backward matches. The core rejects unsupported opcodes,
nonzero reserved fields, invalid distances, and decoded-length
overruns. The VFS adapter rejects truncated or trailing input
and enforces bounded compressed counts from external metadata.
Source/destination physical geometry, disjoint extents and MM
ownership are trusted KCORE caller invariants, not duplicated
inside the decoder.

Block v1441 is intended to **replace D6LZ36 completely** as
the single immutable TSFS/executable decoder after an atomic
Stage-1, KINIT, EXEC, PURE-prefix and compressed-file cutover.
Do not retain both executable codecs permanently. D6LZ36
remains in the present production boot path until acceptance.

## Measured code size

| Version | Core TEXT | VFS adapter TEXT | Combined TEXT | Frame |
|---|---:|---:|---:|---:|
| v1424 baseline | 75 | 89 | 164 | 40 |
| v1435 | 66 | 89 | 155 | 40 |
| v1436 | 64 | 89 | 153 | 40 |
| v1437 | 63 | 88 | 151 | 40 |
| v1438 | 60 | 88 | 148 | 40 |
| v1439 | 60 | 88 | 148 | 39 |
| v1440 | 60 | 84 | 144 | 39 |
| **v1441** | **60** | **58** | **118** | **39** |

No static DATA/BSS. Relative to v1424 the final v1441 code
saves **46 words (28.05%)**. The 39-word VFS frame contains
32 compressed-input words, five saved ACs and two persistent
words. The complete nested VFS stack may be larger.

Optimizations include PDP-6 halfword pointer construction,
shared zero-run/match BLT handling, using the updated cursor
as the inclusive BLT end, combined increment/branch operations,
reducing saved ACs and eliminating repeated checks of trusted
kernel-owned addresses and buffers. Checks for corrupt file
data and untrusted stream lengths remain.

## Native PDP-6 acceptance

All **23 applicable native regression tests pass**: eight
valid direct/refill fixtures, nine malformed compressed-stream
cases, a stack-canary check, and five complete real DXR
reconstructions (KCORE, D6FS, MonitorFS, SWAP and Tape MRES).
The source/destination-alias fixture is a test of an invalid
KCORE caller, not malformed encoded input, and is excluded
under the explicit caller-owned-memory contract.

| Compiled source sample | Original v1424 SIMH steps | v1441 steps |
|---|---:|---:|
| 29-DXR mixed prefix, 2,477 words | 2,490 | **2,003** |
| DSH first 2,048 words | 5,039 | **4,054** |
| KCORE first 2,048 words | 2,961 | **2,382** |

These are simulator-retired **instruction counts**, not
cycle-accurate hardware speed or TSFS I/O timings: a BLT
instruction can transfer many 36-bit words.

## Tradeoff versus present D6LZ36-BLT

| Measure | D6LZ36-BLT | Block v1441 |
|---|---:|---:|
| Core TEXT | 39 | 60 |
| Core plus VFS adapter | 96 | 118 |
| VFS stack frame | 72 | 39 |

The persistent-code premium is only **22 standalone words**
instead of the original 68. On 29 compiled DXRs, compressed
whole-image payloads total 37,310 D6LZ36 vs 37,190 block
for 48,487 original words. These exclude TSFS media framing.

## Open integration gates

The current Stage-1 fixed decoder occupies 39 words and
v1441 occupies 60. A naive swap adds 21 core words before
changed initialization/layout. Prove each hardware loader
placement and read-in budget without blindly relaxing limits.
The historical DTC 106-word test cap already fails before
this block conversion and needs a separate investigation.

Complete one block-only path for all supported boot devices,
TSFS root files, KINIT, executable DXRs, MRES, EXEC and bounded
PURE prefixes. Measure linked/folded KCORE, 32K/48K RAM,
complete nested VFS stack, cold TSFS reads, malformed-stream
rejection and interrupted-load rollback. Delete D6LZ36 only
after the full cutover passes.

## Separate optional bitmap MRES

The independently selectable swapless TSFS-only bitmap
encoder/decoder is **not** part of the file decoder. Its
current trusted-RAM pair is 30 TEXT words, zero DATA/BSS
and no saved-register frame, documented in
D6CODEC-BITMAP-TRUSTED-OPTIMIZATION-V1451.MD. Bidirectional
p10fold found no eligible shared cross-domain run of two
or more words. Do not create a hard dependency between the
optional bitmap MRES and file-decoder internal entrypoints
for a theoretical single-word POPJ saving.

Sources and native tests:
DAIMOS/system/kernel/core/d6codec_block_*_v1441.s and
daimos-testkit/tests/validation/d6codec-block-opt-v1441/.
