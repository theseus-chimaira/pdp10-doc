# KCORE v1560: retain exact DTFS MM scratch owner across VFS lifecycle

Date: 2026-11-14. Production VFS/KCORE MM ownership boundary;
**P0-B remains OPEN, DTVOL2 is still read-only**.

The v1552 physical RBN0 WRITE_INTENT helper needs an exact,
caller-retained one-word `basep`. If its 0200-word pinned scratch
cannot be freed, the base must survive the syscall and cannot be
discarded with a mounted volume or reused mount ID. KINIT already
allocates a single six-array MM-backed VFS mount slab, including
one provider-private auxiliary word per mount. This is currently
the D6FS reader pointer on D6FS slots. v1560 exposes its **same
KCORE pointer word** under the alias `vfs_mount_aux_base_v1560`
for retained DTFS scratch ownership on DTFS slots. Both symbols
have the exact same address in the real KCC `kcore.map`.

No extra mount array, MM allocation, fixed KCORE data word,
syscall ABI or media-format change is introduced. D6FS's boot-
reader and mounted-reader lifecycles are still provider-disjoint;
DTFS cannot overwrite a live D6FS reader in another provider slot.

Production `vfs.s` includes `vfs_dtfs_aux_empty_v1560`, which
rejects missing/out-of-range mount slots or a still-owned MM
scratch base without freeing or clearing it. Its call sites:

- Before publishing a DTFS mount into a recycled VFS slot;
- After all sleeping SYNC/PREPARE and final FILE/cwd/PIPE/swap
  busy checks, immediately before DTFS unmount owner unlock and
  mount retirement;
- Both before the poisoned-DTFS physical replacement provider
  read and after it, following the final owner check but before
  administrative discard/retirement of that mount.

Native daimos-testkit acceptance executes the **actual PDP-6
helper instructions**, with 16 cases PASS and five unsafe source
mutations rejected. The real VFS mount-publication and final
unmount slices execute with 14 native cases PASS and five unsafe
mutations rejected, including legitimate D6FS root reader reuse.
The source-order test rejects five additional mutations that
drop the administrative discard, mount or unmount owner checks
or split the identical MM base pointer alias.

Historical native v1514 failed-media unmount/retry and v1523
all-process FILE/cwd ownership regressions PASS. Complete KCC
full-profile KCORE and DTFS MRES assembly/fold/link PASS.

This change prepares the correct retained MM **storage and VFS
lifetime rules**, but the actual v1552 `basep` is NOT yet passed
from the live DTFS provider. The v1551-v1554 DIRTY-before-data
gateway remains outside mounted DATA/DIR writes, and physical
V2 CLEAN/unmount cannot yet be committed. Type-551 partial write,
replacement/reinsertion and cold-boot acceptance, including the
physical tape-exchange race, remain open. Do not enable V2 RW.

Next: use the checked `&mount_aux[slot]` as the v1552 scratch
basep under the serialized DTFS provider lease, then protect all
mutating paths and implement exclusive final CLEAN/unmount.

P0-A complete; **P0-B OPEN**; P1-C separate. VERSION 0.5;
strict 32K/optimization HOLD.

Design `DAIMOS/design/kcore-dtfs-retained-aux-v1560.md`.
Tests `daimos-testkit/tests/validation/
daimos-dtfs-retained-owner-v1560.md`.
