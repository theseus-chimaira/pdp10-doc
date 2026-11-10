# v1461: source-matched TSFS host tools and strict extent validation

Date: 2026-11-10. Status: host-producer and validator fix
verified; full-profile TSFS-root login, DSH and ECHO accepted.

## Failure mechanism

The v1459 TSFS-root image was generated with an `mktsfs`
binary installed on 2026-10-16, while the source tree's
`d6block-codec-v1452.c` was updated on 2026-11-10. The
old producer emitted an incompatible compressed restart
extent for the actual LOGIN executable. The independent
v1421 reference decoder rejected the extent with an invalid
zero run. The 53-word v1458 PDP-6 decoder was not at fault.

The installed `tsfscheck` was also outdated. In the source,
`validate_extent_data` compared the entire right half of
`PARENT_FLAGS` against `TSFS_FILE_FLAG_REG`. That right half
contains the file mode (for example 0555), so ordinary
regular-file extents were silently skipped. The stale image
was incorrectly reported as valid.

## Changes

* Mask with `TSFS_FILE_FLAG_MASK` before selecting regular
  files for extent validation.
* Build `mktsfs-v1461` and `tsfscheck-v1461` from the
  checked-out host sources in the boot build directory.
* Make the TSFS-root image depend on these binaries and
  verify all generated TSFS members with the strict checker
  before completing the image build.

No on-disk format change, KCORE growth, new permanent RAM,
or runtime decoder change was required.

## Evidence

The old image is rejected by the corrected checker. A fresh
image built with the current producer passes: 53 files and
401 extents. Its LOGIN final restart contains 149 decoded
words and 64 compressed words. The independent host decoder
reconstructs the actual LOGIN DXR bytes; the native PDP-6
v1458 fixed decoder and VFS frontend pass on that same
compressed payload.

The testkit also independently reconstructed complete
LOGIN (3221 words, 13 extents), DSH (15760 words, 62 extents)
and ECHO (1035 words, 5 extents) from the generated TSFS
media. Every reconstructed word matches the linked DXR.
The external-command runtime problem therefore cannot be
attributed to mismatched ECHO file contents in this image.

With the corrected image, the full 256K TSFS-root system
reaches INIT, ROOT LOGIN and DSH V1. The guest external ECHO
command completes and returns to DSH within the extended
240-second command allowance (`V1461_DSH_EXEC_PASS`). The
initial 70-second allowance timed out, but the same media
and unchanged guest code passed the extended acceptance.
The entire isolated-media cold-boot test took about 307
seconds of wall-clock time. This does not imply that the
individual command consumed 307 seconds.

Regression tests are in
`daimos-testkit/tests/validation/d6codec-v1461-tsfs-exec/`.
