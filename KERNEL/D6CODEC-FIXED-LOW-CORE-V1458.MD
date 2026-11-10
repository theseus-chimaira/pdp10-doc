# Block decoder v1458: 53-word low-core PDP-6 component acceptance

Date: 2026-11-10. **Component validated and committed as an isolated
candidate, not production boot-integrated.** The D6LZ36-free block
cutover baseline is v1452. This experiment preserves the v1421 block
wire format and retains the decoder at executive address 000060.

## Exact installed size

| Component | Native 36-bit TEXT words | Static DATA/BSS |
|---|---:|---:|
| v1452 fixed-low-core decoder | 61 | 0 |
| v1458 fixed-low-core decoder | **53** | **0** |
| v1458 VFS/direct-buffer adapter | **58** | **0** |

The fixed decoder has **52 executable instructions and one inline
LDB byte-pointer word**. DAS/DOBJ confirms 53 total TEXT words,
no static DATA/BSS and zero relocations. It includes the byte
pointer in the copied image. A separate fixed copy is not kept
in KCORE: Stage-1 copies its embedded decoder to 000060 and the
runtime adapter calls the same fixed address. The source is
`system/stand/pdp6/common/block_decompressor_v1458.inc`;
its standalone VFS adapter is
`system/kernel/core/d6block_vfs_fixed_v1458.s`.

The adapter reserves a 39-word temporary VFS frame (32 compressed
input words, five saved ACs and two state words), restored on exit.
The potential standalone fixed-code total is therefore **111 words**
(53 + 58), excluding MM/VFS/EXEC call chains and linked/folded size.

## Instruction changes compared with v1452

The format remains: 36-bit command headers with a two-bit opcode,
seven-bit count-minus-one, nine-bit distance-minus-one and 18
reserved zero bits. Opcodes are literal, zero-run and match;
opcode three is invalid. Zero-run and literal must have zero
distance. Overlapping back-references are copied with BLT.

The v1458 fixed decoder reduces the earlier 61-instruction
v1452 implementation using these substitutions:

- `SOJL` combines available-input decrement and the need-input
  branch; the adapter must refill after +1 rather than assume AC4
  remains zero.
- `LDB` extracts the seven length bits using one inline byte
  pointer, and that pointer is relocated with the fixed image.
- `CAIN` plus `AOJA` handles a single-word zero run without an
  extra unconditional branch.
- `MOVNI` plus `ADD` computes a negative backward distance in
  two instructions instead of three.
- The literal, match and zero-copy paths share BLT pointer
  completion and fall through to a common status return.

No malformed-stream checks are intentionally removed: reserved
bits, opcode, distance, history origin, output length and bounded
stream consumption remain checked by the core/adapter pair.

## Native PDP-6 acceptance, completed

Tests run the actual PDP-6 SIMH simulator through `p10run`.
The fixed-image harness copies v1458 with BLT to 000060 before
calling it, with **source and length labels in one assembly
unit** to avoid the separately relocated symbol-difference
assembler issue identified during the initial v1452 test.

| Test class | Passed |
|---|---:|
| Relocated low-core valid streams including real DSH and mixed DXRs | 6 |
| Direct low-core malformed stream and NEED_INPUT controls | 5 |
| VFS/direct-buffer/refill valid streams | 8 |
| VFS malformed-stream rejection | 9 |
| Whole KCORE, D6FS, MonitorFS, SWAP and Tape DXRs | 5 |
| VFS stack-canary | 1 |
| **Total native executions** | **34** |

All five Stage-1 sources also assemble with v1458. Measured
read-in sizes in 36-bit words, with the v1452 baseline for
comparison:

| Stage-1 | v1452 | v1458 | Savings |
|---|---:|---:|---:|
| DSK | 384 | **376** | 8 |
| DRM | 376 | **367** | 9 |
| DTC | 168 | **160** | 8 |
| MTC | 133 | **125** | 8 |
| PTR | 167 | **159** | 8 |

Assembly/layout checks are **not real cold boots**. The five
native Stage-1 source assemblies were performed on temporary
copies, preserving the actual relative `.include` structure.
The selected build is deliberately unchanged in this commit.

## Native speed caution

Prior native STEP measurements comparing the earlier v1457
decoder with v1458, on identical decoded words:

| Sample | v1457 instructions | v1458 instructions |
|---|---:|---:|
| One literal word | 58 | 59 |
| 415-word overlap | 2,224 | 2,225 |
| 4,096 zero words | 21,243 | 21,243 |
| 3,072 DSH words | 21,205 | 21,318 |

The smaller version is not always the fastest. These are SIMH
retired instruction counts, **not cycle-accurate PDP-6 wall time**;
one BLT may transfer many words. The memory-first priority still
favors saving eight resident words without losing validation.

## Remaining work

This stage establishes the validity of the optimized decoder
as a component. It does **not** assert a complete block-only boot,
KINIT/TSFS/EXEC/PURE linkage, cold DECtape restore, runtime
module installation, or 32K/48K integrated MM validation.
Those gates must pass before the selected KCORE and Stage-1
build rules are switched to v1458 and the cutover accepted.

Tests, native fixture generation and Stage-1 size comparison
are in `daimos-testkit/tests/validation/d6codec-fixed-v1458/`.
