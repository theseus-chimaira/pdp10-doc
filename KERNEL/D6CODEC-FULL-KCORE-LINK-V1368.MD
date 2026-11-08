# v1368: grouped runtime decoder full KCORE link and MRES relocation

Date: 2026-11-08. Status: isolated linker/MRES/boot-stream tests,
**not** a production codec, media or KINIT runtime migration.

The v1368 testkit evaluates the actual PDP-6 `KCORE_FIXED_OBJS`
and `KCORE_FOLD_OBJS` lists from the DAIMOS boot Makefile. A full
baseline rebuild reproduces the production KCORE DXR, SIMH RIM and
initialized-word stream **byte-for-byte**. The baseline has 9,601
text words, 269 BSS words and exclusive low-core end `023276`.

The grouped experiment takes the tested v1367 native decoder and
VFS adapter and publishes the original `d6lz36_decode_vfs` and
`d6lz36_decode_buffer` ABI labels only within a temporary assembler
copy. The generated KCORE then links against exactly the same
preexisting EXEC, swap, memory manager, VFS and other core objects.

Two true single-grouped-codec layouts are compared. With the former
40-word D6LZ36 decoder removed and its low-core address region
reclaimed, the grouped KCORE has 9,817 text and 269 BSS words:
**+216 text**. Retaining a non-executable 40-word reservation instead
preserves early EXEC layout and yields 9,857 text/269 BSS: **+256**.
The reservation is *not* a second codec. A third measurement-only
link keeps the legacy decoder executable alongside grouped code
and has the same 9,857-word text size; it is not a suggested
production fallback.

Regenerating `p10fold -m 2` with the grouped objects changes the
plan from 29 folds/71 words to 31 folds/75 words, saving just four
more words. With the original fold plan the two size deltas are
+220 and +260. This resolves the difference between previous
standalone +220 estimates and the actual complete linked results.

The no-slot option moves `exec_replace_current_lowmem` from octal
`000407` to `000337`. The reserved option retains `000407` exactly.
Both options start KCORE at `000060`; the old low-core ABI/lifetime
assumptions still need actual KINIT installation and post-entry
runtime tests before the smaller option can be accepted.

Five real relocatable MRES packages have been re-linked against
both grouped KCORE maps. Each baseline original-map link is byte-
identical with its existing production package. Their resident
word counts remain unchanged: SWAP 1,757; DTFS/TSFS 1,834; D6FS
2,175; MonitorFS 1,592; tape 405. The new KCORE also converts to
SIMH RIM and initialized-word stream, with low-core initialized
ends `023211` (no-slot) and `023261` (reserved). These are *test
artifacts*, not a verified boot or complete grouped filesystem.

The existing `vfs_read_words` path dispatches real filesystem reads
through `fs_provider_reg_call`. The serialized provider can retain
a seven-word private stack frame across waits and real I/O. v1367's
67-word adapter frame and 71-word leaf-provider canary do not bound
this nested call graph. The true process kernel stack has only
`0315` octal words, making cold-read high-water verification a
required next step along with partial-failure cleanup.

The production D6LZ36 codec remains active and KCORE is unchanged.
The +216 minimum linked code growth remains a significant RAM-first
release blocker. Next work must validate low-core ownership,
decrease codec size, measure real provider/caller stack depth,
exercise grouped EXEC/PURE/TSFS and perform an atomic host and
runtime format cutover without a permanent dual-codec fallback.
