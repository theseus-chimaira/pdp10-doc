# v1367: PDP-6 grouped runtime decoder RAM optimization

Date: 2026-11-08. Status: staged native decoder, not KCORE/runtime
production activation. Production files still use D6LZ36.

## ABI and compressor-format compatibility

The grouped v1367 stream engine and VFS adapter preserve the v1366
contract: normal AC1..AC4 VFS and exact-buffer entries, saved
AC10..AC16, 0/-1 completion, `+1 NEED_INPUT` within the decoder,
and AC3=0 trusted immutable VFS backing. The grouped v053 bitstream
format and its 30-token groups/15-descriptor bound are unchanged.

The decoder now shares its one descriptor-position state word across
mutually exclusive input ingestion and token output phases. Its
continuation has 25 words, including 15 packed descriptors. Other
size savings use PDP-6 left-half address checks (`TLNE`), shorter
phase dispatch, a common bounded/trusted source-word fetch, and
indexed descriptor memory references. The VFS adapter has shorter
bounded-read and input/output/frame geometry checks without dropping
physical-address safety.

## Object size and temporary RAM

| 36-bit words | D6LZ36 runtime | Grouped v1366 | Grouped v1367 |
|---|---:|---:|---:|
| Engine instructions | 40 | 237 | **225** |
| VFS/buffer adapter instructions | 57 | 96 | **92** |
| Combined instructions | 97 | 333 | **317** |
| Static data/BSS | 0 | 0 | 0 |
| Grouped continuation state | n/a | 26 | **25** |
| Per-call VFS stack frame | 72 | 68 | **67** |
| Fake-provider stack canary, octal | not measured here | `0110` | **`0107`** |

The result is **16 fewer code words** and one fewer temporary state
word than v1366. It remains **220 resident instructions larger**
than the current D6LZ36 engine plus adapter. This is a significant
RAM-first KCORE concern. Standalone object figures have not been
replaced with a measured full KCORE link or p10fold result.

The testkit `tests/validation/d6codec-ram-v1367` reuses the current
linked KCORE/MRES DXR host-golden corpus, the v1366 fake VFS
ABI/errors, the 30-match maximum descriptor stress, and five native
128-word DXR prefix categories (KCORE, D6FS, MonitorFS, SWAP and
tape) at three refill windows (1, 7 and 64). The canary depth `0107`
octal equals 71 decimal words **including CRT and leaf stub calls**;
this is not real cold VFS/provider/EXEC/PURE stack depth.

The complete process private stack holds only `0315` octal words,
and nested provider/EXEC caller depth is yet to be measured. Before
production, reduce the remaining code increase, measure a temporary
full KCORE link and its folding, run provider failure and partial
output rollback tests, and migrate host producers plus all consumers
in one atomic format cutover without an installed D6LZ36 fallback.
