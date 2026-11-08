# v1363: direct-linked grouped decoder for all five PDP-6 Stage-1 loaders

Date: 2026-11-08. Status: staged native loader-handoff prototype;
production D6LZ36 images are unchanged.

An audit of the real DSK, DRM, DTC, MTC and PTR Stage-1 code finds
that every loader already reads and stores the **complete** compressed
KINIT payload above the KINIT destination starting at octal 030000.
Stage-1 does not call the existing D6LZ36 resumable core between
input refills: the decoder is only invoked after all compressed
words have been staged. The old decoder was fixed at 000060 because
it is also used by the runtime VFS path, not because Stage-1 needs
resumability.

The smaller staged complete-buffer `d6codec_v053_decode` can instead
be linked and called at its real Stage-1 text address. There is no
decoder-copy BLT to the first 40 low-core words, no 230-word streaming
engine, and no 26-word continuation frame inside Stage-1. The runtime
VFS path still requires bounded source refills and a resumable codec.

The testkit v1363 generates alternate Stage-1 sources *without
editing production*, assembles original and grouped variants with
`das -F -K`, constructs proper read-in artifacts at load base 000060
and checks text/BSS footprints against the stack start at 020000.

| Loader | Original text | Grouped text | Increase | Staged BSS |
|---|---:|---:|---:|---:|
| DSK | 350 | 432 | +82 | 879 |
| DRM | 342 | 424 | +82 | 879 |
| DTC | 140 | 229 | +89 | 0 |
| MTC | 106 | 195 | +89 | 0 |
| PTR | 139 | 224 | +85 | 3 |

The DTC/MTC handoff copies the already-retained compressed length
from AC4 into AC2, then places the uncompressed length in AC4.
AC5 (entry address) and AC13 (output size) are saved across decoding.
PTR saves AC16 (entry). Existing KINIT entry and error behavior is
otherwise preserved. No new Stage-1 BSS is introduced.

Ten native PDP-6 simulations execute the actual generated handoff
code at address 000060: one valid grouped-literal KINIT transfer and
one invalid-control rejection per loader. All ten pass; the valid
cases also check destination value, KINIT stack address, and (for
DSK/DRM) the member count. Malformed input fails without publishing
or jumping into a partially decoded KINIT image.

The test-only native harness does not simulate physical disk, drum,
DECtape, magnetic tape or paper-tape transfers. Before production,
Stage-1 must validate source/output staging geometry prior to media
writes; all grouped KINIT/host producers, runtime VFS/EXEC/PURE/TSFS
consumers and five real Stage-0 boot paths need a synchronized
rebuild and regression acceptance. The production fixed D6LZ36
decoder, compressed images and permanent KCORE size remain unchanged.
