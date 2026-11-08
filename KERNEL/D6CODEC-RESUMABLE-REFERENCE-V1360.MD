# v1360: grouped D6CODEC resumable interface reference

Date: 2026-11-08. Status: validated **host** continuation model,
not PDP-6 low-core assembly, system boot or production activation.

The currently active Stage-1 and KCORE D6LZ36 decoder is installed at
octal address 000060. It supports the VFS decoder's 64-word refill
window. The staged grouped v053 decoder consumes a complete stream
and does not implement that resumable ABI.

The v1360 testkit models a grouped-state continuation that can return
NEED_INPUT and resume at any compressed word boundary, including
mid-descriptor-table or mid-literal-group. Groups contain up to 30
tokens. Their packed 18-bit descriptors require at most **15 36-bit
scratch words**; the output buffer itself provides copy history. No
46-word whole-group staging buffer is required. Exact input and
output counts must both be exhausted before successful completion;
malformed, truncated and noncanonical streams fail rather than
publishing a partial output.

`daimos-testkit/tests/validation/d6codec-stream-v1360` PASSes all
27 current kernel/MRES DXRs at six input-window sizes (1, 2, 3, 7,
16, 64 words), a total of 162 explicit resumptions; plus ten
malformed stream patterns at each width and a constructed 30-match
group using exactly 15 descriptor words. It also tests empty-stream
framing. This is test-reference evidence only, not native boot.

Before production: allocate actual PDP-6 continuation state, prove
register/stack and physical-address invariants, measure low-core
relocation size, and test provider failure/restart and all five Stage-1
bootloaders. Then migrate KINIT, VFS, EXEC/PURE, TSFS and the host
producers together without a persistent legacy codec.

## v1361: first PDP-6 native resumable implementation

DAIMOS now stages `system/kernel/core/d6codec_stream_v1361.s`, a
native assembly-only grouped decoder returning 0, +1 NEED_INPUT,
or -1 on failure. The caller owns a 26-word state with 15 packed
descriptors. Native PDP-6 tests PASS one-word refill boundaries for
literal, zero-run, a 30-match stress group, malformed control and
invalid output geometry. A host-golden KCORE DXR 128-word prefix
also roundtrips with one-word refills. The object measures 230 text
words, zero data/BSS, 82 relocations. It is **not** installed at
octal 000060 and does not yet establish low-core placement, real
VFS-provider nested stack, full native error handling or any boot
loader integration. Production remains D6LZ36.
