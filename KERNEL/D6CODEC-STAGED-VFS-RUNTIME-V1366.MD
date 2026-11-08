# v1366: staged grouped PDP-6 runtime VFS decoder

Date: 2026-11-08. Status: native PDP-6 **fake-provider** validation
only; no resident KCORE link or production D6LZ36 format switch.

## Actual caller ABI requirements

The existing D6LZ36 resident runtime frontend has a streaming VFS
entry and a complete-buffer entry. Runtime EXEC expects exact
compressed input consumption. PURE-backed reloads can pass AC3=0,
meaning validated immutable backing with an unknown compressed
length; decoding may finish at the exact output word boundary.
TSFS restart extents use a complete in-memory compressed buffer.
These distinct modes must survive any grouped codec cutover.

Two new, versioned DAIMOS assembler objects implement this contract:

- `d6codec_stream_v1366.s`: grouped resumable decoder, with
  `DS_REMAIN=-1` denoting trusted immutable VFS source. Nonnegative
  lengths still demand exact compressed source consumption.
- `d6codec_vfs_v1366.s`: AC1..AC4 ABI-compatible VFS and direct
  buffer entries, preserving AC10..AC16; zero static data/BSS.

The frontend uses a 32-word VFS refill window and 26-word grouped
continuation state, including up to 15 packed descriptor words.
Its 68-word temporary stack frame also holds seven saved ACs and
three provider parameters. The existing D6LZ36 VFS frame occupies
72 words. The grouped decoder checks the entire reserved adapter
stack frame for illegal output/input overlap, not merely the 26
state words. The source is rejected on malformed geometry, counts,
truncation, bad tokens or provider error without claiming success.

## Native acceptance and memory tradeoff

The testkit `d6codec-vfs-v1366` uses the independently host-encoded
current KCORE DXR 128-word prefix. Native PDP-6 tests PASS exact VFS
refills, short seven-word provider reads, trusted backing with excess
input, direct exact buffer decoding, rejected early EOF, malformed
control, trailing input, truncated/overlapping direct buffers,
negative/19-bit source counts, overlapping adapter stack regions and
callee-saved AC10..AC16 preservation. All 128 reconstructed output
words are compared. A stack-canary test reports **0110 octal words**
(72 decimal) including the test CRT and decode calls. This provider
is a leaf fake; real nested provider/EXEC/PURE stack usage is OPEN.

Object-size comparison in 36-bit instruction words:

| Component | Production D6LZ36 | Grouped staged v1366 |
|---|---:|---:|
| Decoder | 40 | 237 |
| VFS adapter | 57 | 96 |
| Combined text | 97 | 333 |
| Static data/BSS | 0 | 0 |
| Temporary VFS frame | 72 | 68 |

Although the grouped adapter saves four stack words, it would add
**236 permanently resident instructions** before linking/folding.
This is a serious RAM-first optimization concern. The numbers are
standalone assembler measurements, not an updated KCORE size.
No claimed system RAM savings or release acceptance follows.

Next work must reduce code growth, measure real cold VFS/provider
stack depth, establish exact PURE/TSFS/EXEC caller handling, link a
temporary KCORE and execute a synchronized grouped host/media/runtime
format migration. All production decoders and boot images remain
on D6LZ36 until those gates PASS; no permanent dual-codec fallback.
