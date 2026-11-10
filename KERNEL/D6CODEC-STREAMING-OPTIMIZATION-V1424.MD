# PDP-6 streaming decoder optimization v1420-v1424

Date: 2026-11-10. Status: **completed isolated experiment**.
The production D6LZ36 decoder, KCORE build, Stage-1 and media remain
unchanged.

## Candidates and bitstream

1. Grouped v1367: the original resumable D6CODEC control.
2. Grouped v1420: identical compressed bits, BLT for complete literal
   groups, zero runs and word matches; token state cached in ACs.
3. Block LZ/RLE v1421: separate experimental format, six-word
   continuation, 32-word VFS refill window.
4. Block LZ/RLE v1424: exact same v1421 stream, but compact
   register-resident resumable ABI and a validated 32-word VFS adapter.

Block commands occupy one 36-bit word. Bits 35..34 select literal
block (0), zero run (1), backward whole-word match (2), invalid (3).
Bits 33..27 encode length minus one, giving 1..128 output words.
Bits 26..18 encode distance minus one (1..512) for matches and must
be zero for literals/zeros. Bits 17..0 are reserved zero. A literal
command is followed by exactly its raw words. Backreferences may
overlap the destination and must start inside existing output.
No halfword-match command is supported. The block format is
INCOMPATIBLE with grouped D6CODEC and D6LZ36.

## Assembled code and temporary RAM (36-bit words)

| Candidate | Core text | Adapter text | Total text | VFS frame | Canary |
|---|---:|---:|---:|---:|---:|
| Production D6LZ36 | 40 | 57 | 97 | 72 | n/a |
| Grouped v1367 | 225 | 92 | 317 | 67 | 71 |
| Grouped BLT v1420 | 272 | 92 | 364 | 67 | 71 |
| Block v1421 | 181 | 92 | 273 | 48 | 52 |
| **Register block v1424** | **75** | **89** | **164** | **40** | **43** |

All staged candidates have zero static DATA/BSS. v1424 uses AC11
for remaining literals, AC12 output cursor, AC13 output count and
AC14 output origin. Its frame is six saved ACs, vnode/offset and
32 refill words, with no continuation-state array. The canaries
include test CRT and leaf provider calls, NOT real nested filesystem
or EXEC call chains. All sizes are standalone assembly, not a
linked/folded KCORE delta. Compared with staged grouped v1367,
v1424 saves 153 instruction words and 27 frame words. Relative to
production D6LZ36, its standalone combined text grows 67 words.

## Real DXR compression corpus

The present built corpus contains 29 real DXRs, 48,717 input words.
Both host encoders independently roundtrip all full streams.

| Format | Encoded words | Encoded / input |
|---|---:|---:|
| Grouped v1367/v1420 | 35,704 | 73.29% |
| Block v1421/v1424 | 37,380 | 76.73% |

The block format uses 1,676 extra compressed words, a 4.69%
increase over the grouped output (3.44 percentage points of input).
Its host encoder searches up to 128 previous words, chooses matches
of length >=3 and zero runs of length >=2, and is not globally
optimal. The block format permits distances through 512 words;
better producer match finding remains a separate possible improvement.

## Measured native simulator instructions

SIMH step-limit binary searches successfully bracketed complete
direct-buffer decoder + adapter execution, including a minimal CRT.
Normal data comprise 2,477 source words concatenated from all 29
distinct DXR prefixes. Worst is a deliberately inefficient, valid sequence
of 1,499 one-word matches after one literal, not actual encoder output.

| Decoder | Zeros (3810) | Mixed (2477) | Literals (1920) | Repeated (2033) | Worst (1500) |
|---|---:|---:|---:|---:|---:|
| Grouped v1367 | 13,204 | 94,115 | 78,004 | 11,240 | 92,006 |
| Grouped v1420 | 1,955 | 57,169 | 13,685 | 1,125 | 89,054 |
| Block v1421 | 1,499 | 3,999 | 1,109 | 953 | 72,137 |
| **Register block v1424** | **1,063** | **2,490** | **628** | **670** | **52,575** |

BLT counts as one simulator step regardless of the number of
transferred words. Consequently, these are NOT machine cycles and
the step ratios must not be presented as real-hardware speedups.
The versioned testkit also measures BLT calls and BLT-transferred
words. An illustrative timing model uses 2.5/4.0 microseconds per
ordinary instruction and the DEC PDP-6 brochure F-61 Table 2
fast/slow BLT formulas (1.5+0.8*n / 2.4+1.2*n microseconds).
In its slower-memory scenario, the estimated **CPU-only** rates are:

| Decoder | Mixed words/s | Zero-run words/s | Pathological words/s |
|---|---:|---:|---:|
| Grouped v1367 | ~6,600 | ~72,000 | ~4,100 |
| Grouped v1420 | ~10,800 | ~309,000 | ~4,200 |
| Block v1421 | ~131,000 | ~363,000 | ~5,200 |
| **Register block v1424** | **~193,000** | **~435,000** | **~7,200** |

These rates are illustrative, not PDP-6 measurements or a
cycle-accurate prediction. DEC warns that actual instruction
times depend on memory and individual operands. The model omits
provider and device I/O, contention, and filesystem overhead.
Source: https://bitsavers.trailing-edge.com/pdf/dec/pdp6/brochures/F-61_PDP-6_Brochure_196402.pdf

## Tests

- Native v1422: 24 success-path cases across grouped control,
  grouped fast-path and block, plus 22 malformed-stream rejections.
- Native v1424: eight direct/refill cases, ten error cases,
  saved-register check and 43-word stack canary.
- Five **complete** current DXRs (KCORE, D6FS, MonitorFS, SWAP,
  tape) pass native word-by-word validation for each of three
  experimental decoders, 15 full-image runs in total.
- The 29-full-DXR host corpus roundtrips under both independent
  format producers.
- All tests reside under versioned daimos-testkit/validation
  d6codec-fastpath-v1420, d6codec-block-v1421 and
  d6codec-compare-v1422. Assembly sources are unlinked, versioned
  files in DAIMOS/system/kernel/core.

The standalone v1424 adapter checks exact sizes, physical
source/output bounds, stack-frame and source/output non-overlap,
bad commands, reserved bits, truncated input and trailing input.
Partial output on error must remain unpublished by any future caller.

## Decision and unresolved gates

**v1424 is the leading experimental replacement**: the simple
block format, accumulator-resident decoder and BLT are much faster
for real executables than the grouped versions, with smaller
resident code and stack use at a modest compressed-data cost.
However, it remains 67 standalone TEXT words larger than current
production D6LZ36. No production cutover is authorized here.

Before acceptance, measure real linked/folded KCORE and MRES sizes,
normal cold VFS/provider/EXEC stack depths and RAM high-water; test
full Stage-1/KINIT boot, all media, error injection, PURE restore,
TSFS and 32K/48K configurations. Decide one canonical wire format,
implement bounded PURE-prefix reconstruction or separate framing,
and migrate host producers, boot and runtime consumers atomically.
Do not introduce a permanent legacy decoder or dual-format fallback.
