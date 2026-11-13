# KCORE feature state v1525: real physical mounted native DTFS

Checked: 2026-11-13. DAIMOS 0.5, PDP-6 256K.
This is a **feature-correctness** increment, not an
optimization or additional installed module policy.

## Physically accepted in v1525

The real tape-selected KINIT reports `DTFS LOADED`,
and the accepted booted guest physically mounted an
independently attached and fully `DTVOL1`-stamped
DTC3 native filesystem. The guest test uses an
actual KCC executable as PID1 under the normal
KCORE, VFS, MM mount-table, Type-551/136 and optional
DTFS MRES code paths. Only the disposable TSFS root
manifest differs from normal boot so that testing
does not require minutes of unrelated DTC0
userland command loads.

Actual guest assertions all PASS:

- DTC3 physical native format admitted by KCORE
  with the nonzero high and low 36-bit identity
  halves and original directory layout intact
- FSINFO mount slot 0 reports TSFS provider 7 '/'
  and slot 1 reports **DTFS provider 5
  '/MOUNT/DT0'**
- STAT and GETCWD on the mounted target succeed
- an occupied cwd prevents UNMOUNT and leaves
  the FSINFO mount published
- a real directory FILE opened on DTFS independently
  prevents UNMOUNT after cwd moves back to '/'
- closing the final FILE permits UNMOUNT; FSINFO
  subsequently reports its slot empty
- `DTFSGATE V1525 PASS` and the physical
  testkit guest runner exit successfully

This proves **one-process physical mount/cwd/FILE
lifetime**; it does not claim actual multi-process
fault, media detach/replacement or partial-write
recovery.

## Root causes fixed in actual production source

1. `dtfs_mount_unit` checked `fs_block_workspace+014`
   for physical directory block 0144 owner. C
   `dir[14]` is decimal, and the PDP-6
   assembly offset must be octal `+016`.
   Host `mkdtfs`, target `MKFS.DTFS` and
   `dtfs_probe` already agreed on C index 14;
   the previous native v1519 and v1520
   source-linked test media accidentally mirrored
   the wrong index and now use the correct format.
   The v1519 suite rejects a new executable
   old-index negative mutation.
2. `tsfs_parent` and `tsfs_parent_name`
   incorrectly branched to error on *every nonzero*
   result of `tsfs_parent_common`, even though a
   valid parent vnode is necessarily nonzero.
   Both now check only negative returns (`JUMPL`).
   This restored mounted TSFS namespace path
   reconstruction, used by FSINFO and GETCWD.
   The new source-linked v1525 native PDP-6
   suite checks both valid parent kinds and
   SIXBIT `DT0` name, negative record lookup,
   and rejects two separate old-branch mutants.
3. The installed `mkdtfs` binary was stale and
   still produced legacy zero RBN0; current
   production host source already stamps a
   72-bit volume identity. It was rebuilt as
   versioned `mkdtfs-v1525`, packaged into
   `local.tar.gz` and installed into
   `PDP10_PREFIX`, preserving the old binary.
   The physical test independently checks the
   entire stamp and directory owner before boot.

The optional DTFS/TSFS MRES remains exactly
octal **003746 text / 000002 BSS** in a full
isolated KCC tape-root build. All source fixes
are operand/conditional-branch corrections:
**zero net new KCORE/MRES resident words**,
no extra permanent MM storage and no on-media
filesystem format change.

The native v1519 mount tests and all seven
negative mutations PASS. The complete v1520
native stamped-volume suite, v1522 identity
fault/return suite, v1523 all-process FILE/cwd
owner suite, new v1525 TSFS parent mutations,
and v1516 root physical-membership tests
all PASS with current production source.

## Outstanding feature-only KCORE gates

- **P0-A:** Authoritative physical permanent
  loss classification for selected TSFS root
  members and BACKSTORE; connect positively
  proven loss to the v1515 fatal HALT.
  Ordinary PI, select and timeout signals
  cannot be treated as permanent device loss.
- **P0-B:** Live **cross-process** FILE/cwd
  owners of the secondary physical mount,
  true DTC3 read errors and verified same
  72-bit ID return, mismatched tape hot-swap
  rejection with preserved open FILE state,
  and tested sticky WRITE_UNCERTAIN handling
  after partial/dirty writes with explicit
  checked administrative recovery.
- **P1-C:** Remaining mixed provider faults
  (DEVICE, VOLUME, BCACHE, LOGSTORE, MEMFS,
  backing and swap) not previously accepted.
- **P1-D:** Reconcile the existing local
  v1492 dynamic FSINFO/MOUNTS/DF capacity
  work with genuine mixed physical mounts,
  holes, errors and the 1/4/8/26 slot matrix.
- **P1-E:** Actual uncompressed inactive
  swapless PURE lifecycle with MM rebase,
  complete APR restore, atomic unwind and
  OOM/lifecycle transitions.
- Final matched test/source/documentation
  feature freeze. No optimization work;
  strict 32K gate on HOLD. Mutable-only
  compression strictly conditional on a
  measured total peak-RAM win.

Primary source: `DAIMOS/design/kcore-dtfs-physical-mount-v1525.md`.
Guest and native details:
`daimos-testkit/tests/validation/daimos-dtfs-kcore-physical-fullguest-v1525.md`,
`daimos-tsfs-parent-v1525.md`.
