# KCORE v1549: DTFS durable write-intent design tested offline

Date: 2026-11-13. P0-B protocol research and model validation.
**P0-B remains OPEN. This is not a production kernel feature.**

Existing native DTFS writes a DTVOL1 72-bit identity to reserved
RBN0 and marks `WRITE_UNCERTAIN` in the volatile mounted media word
after a failed WRITE. The v1531 root-only checked discard requires
safe owner release and different stamped replacement media, but a
later cold boot or reinsertion can still accept the old physically
damaged tape's original identity/directory. The v1529 full-guest
Type-551 experiment actually left one new data word and one old
word after a failed two-word write.

The v1549 candidate uses a checked DTVOL2 RBN0 clean/dirty intent
seal. The intent must be written and physically read back *before*
the first native data/directory write; after a failed data write
the media remains durably dirty. Only a successful filesystem
transaction with no active owners can reach a final checked CLEAN
seal at unmount. Ordinary sync is not sufficient because another
owner may open the mount while sync sleeps. The current C/assembly
DTFS writer uses shared `fs_block_workspace` as its data source;
RBN0 scratch must not overwrite that buffer or the live directory.

Legacy all-zero and DTVOL1 tapes have no previously recorded intent
history and therefore cannot be *silently* upgraded or treated as
certified clean. An explicit salvage/reformat process is required.
Neither matching the old ID nor a valid directory is evidence of
unmodified file data.

Testkit v1549 PASSes 16 format/protocol invariants, 256 simulated
prefix-partial seal writes, and five tests using actual generated
`mkdtfs` DECtape image bytes with two 18-bit lanes per word.
The latter includes an RBN0145 partial data image, dirty RBN0,
new replacement identity, and cold image-file reopen. No
production formatter/provider/mount changes were made; these
offline models do not prove real PDP-6/Type-551 durability.

Remaining P0-B: coordinate DTVOL2 formatter, provider and probe,
use a physically read-backed dirty seal and owner-safe unmount,
prove real 256K PDP-6 partial-write/reinsertion/cold-boot behavior,
and reject mutations bypassing the dirty marker or retaining stale
owners. P0-A remains complete; P1-C is separate and still open.

Source: `DAIMOS/design/kcore-dtfs-durable-intent-v1549.md`.
Tests: `daimos-testkit/tests/validation/daimos-dtfs-durable-intent-v1549.md`.
VERSION 0.5; strict 32K/optimization HOLD.
