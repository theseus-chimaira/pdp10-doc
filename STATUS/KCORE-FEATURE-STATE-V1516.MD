# DAIMOS KCORE v1516: exact TSFS physical-root DECtape member identification

Date: 2026-11-12. KCORE feature acceptance, not device removal detection
and not a 32K deployment-size acceptance. Production fixed-KCORE data
and executable code are unchanged by this step.

## Why TSFS root identification needed one exact physical mask

TSFS can mount a root set of one through eight physical DECtapes.
The original per-mount `tsfs_file_shape` 36-bit word stored up to
eight three-bit logical-to-physical DTC unit selectors in bits 0..23.
Unused trailing fields were zero, which is also the valid physical
unit 0. Therefore iterating every possible field would falsely
classify DTC0 as critical for roots with no unit-zero member.

In v1516, KINIT's previously validated set membership mask is
retained in unused bits **24..31 of that SAME per-mount word**.
The original logical-to-physical selectors remain unchanged; high
bits 32..35 are zero. Duplicate physical unit designations are
rejected before publication. The redundant `tsfs_boot_select()`
mask-reconstruction loop is replaced by extracting the verified
member mask from the handoff. There is no additional data word,
filesystem ABI change, or on-media format modification.

The optional TSFS MRES now exports `tsfs_root_owns_dtc_unit_v1516`.
It accepts physical DTC unit 0..7, checks the actual namespace root
provider TSFS, validates the root public mount ID against the dynamic
`vfs_mount_limit`, and reads the saved high membership bits through
the existing `tsfs_file_shape_base`. It returns +1 for selected
physical TSFS root members, 0 for unrelated units or a non-TSFS
root, and -1 for invalid units or inconsistent root/mount metadata.

The existing `tsfs_read_words` data path still extracts three-bit
unit selectors from bits 0..23, unaffected by the new upper mask.
No root tape was removed in these native tests; this routine
performs **ownership lookup only**, never loss detection.

## Verified source-linked host, PDP-6 and target build tests

Host `daimos-tsfs-boot-eight-v1` PASS: selected root membership
with all eight units, one root tape on physical unit 0, and a
two-member root on physical units 0/5 with metadata on the second
member. The complete 36-bit handoff and existing KINIT root mask
are checked. Physical two-member mask `041` octal is exact.

Native PDP-6 `daimos-tsfs-root-member-v1516` PASS: selected
units 0 and 5; nonselected unrelated units; disk-root provider;
invalid -1 and 010 octal unit; out-of-range mount containing
plausible data; empty shape, null shape pointer; single-unit
physical-zero root; and all eight selected members. Four
executed source mutations were rejected: wrong mask shift,
wrong provider, missing mount-limit check, and accepting an
invalid all-zero physical member mask.

Existing source-linked TSFS slab and cross-member/cross-extent
read tests PASS. The slab case now exercises genuine high
root-mask bits while preserving the same low 24-bit data selectors.
Standalone stored-extent test fixtures gained only missing VFS
root/mount symbols and a fail-closed unused decoder symbol.
No production I/O success was fabricated.

Both **isolated 32K GCC low-memory** tape-root KINIT/DTFS MRES
and **256K KCC full** tape-root KINIT/DTFS MRES successfully
assemble and link. The separate existing 32K fixed-KCORE size
gate still **FAILS**: mapped `__kcore_low_end=023241` octal against
required `<020000` octal. Do not claim 32K size acceptance.

A controlled old/new optional DTFS/TSFS MRES relink with the
same KCORE map/other objects and independently regenerated fold
plans shows approximately **+25..27 optional TSFS instruction
words**, with **+0 permanent data words** and **+0 fixed KCORE
words**. The cost is incurred only in a tape-capable MRES.

Testkit acceptance revision `efac52c` and DAIMOS report
`design/kcore-tsfs-root-member-v1516.md` contain full details.

## Remaining required device loss work

The selected physical member is now unambiguous. However,
Type-551/Type-136 status still does not provide an accepted
authoritative "medium removed" classification. In the SIMH
model an unattached unit sets `DTB_ILL` on selection, but the
same bit also denotes an illegal command or unit. The existing
bounded DTC search and data-transfer watchdogs likewise do not
prove permanent loss. Do not trigger the v1515 fatal HALT on a
single generic negative I/O return, `DTB_ILL`, or a timer alone.

P0: identify verified hardware-compatible removal/change status,
cross-check only active TSFS root-member units using v1516, and
route **confirmed** root loss to the v1515 root fatal HALT; test
physical root and unrelated auxiliary tape removal distinctly.
P1: non-root DTFS checked media identity/replacement, BCACHE,
dirty/open FILE safety and independent root continuation.
Mixed storage, F1 PURE/MM/APR, F3 dynamic FSINFO are still open.
Compression/final RAM optimization follows feature acceptance.
