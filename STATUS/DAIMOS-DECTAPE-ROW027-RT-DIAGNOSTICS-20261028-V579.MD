# DAIMOS DECtape row-027 acceptance and ?RT forensic policy — v579

Date: 2026-10-28.

## Accepted test evidence

All ten row-027 targets have individual passes from matrix-order continuation
runs. This is not yet a single full-matrix strict acceptance run.

- v573 two-boot Type-551 native DTFS persistence, including second-boot
  `FSCK.DTFS -O NATIVE` pass. A disk-selected D6FS root intentionally leaves
  the DTFS mount MRES uninstalled: userland DTFS probe works, mount refuses.
- DECtape streaming and TSFS image tests passed unchanged.
- v575 TSFS scanner host test updated for direct `tsfs_scan_result` API and
  validates eight-member layout, member-7 directory metadata, and rejecting
  a declared ninth member. The old SYS_TSFS handoff ABI no longer exists.
- Eight-member TSFS boot, multiextent, D6LZ all pass.
- v577 true tape-root directory test passes listing root and nested entries
  and STAT for directories/files, replacing retired MOUNT.TSFS fixture.
- v578 **positive** DTFS secondary mount, FSCK, root STAT and unmount
  successfully exercised DTC3 on a DTC-selected root with installed MRES.
- v579 production Type-551 reverse single-block read/write passes.

Testkit commits: `e73ebb7` (v573), `878657d` (v575), `8a66c84`
(v577), `43aba54` (v578). DAIMOS TODO commit `8dddd58`.
No new permanent kernel code or RAM overhead was introduced.

## ?RT is a fatal DAIMOS early-boot diagnostic, not a SIMH reset indication

`kinit_boot()` emits `KINIT_ERR_RT` when the root selected by the
root-select MINIT is not successfully mounted/validated, or the mounted
root lacks a regular `/SYSTEM/INIT`. Its occurrence on a second boot is
**not expected behavior** and does not alone prove a particular cause.

For any recurrence, preserve the exact boot ini, simulator transcript,
KCORE map, selected-root/handoff state, and all mutable attached media.
Determine whether the failure was root selection, filesystem validation,
mounting, or missing/nonregular INIT. Reproduce with the failing image
before building a pristine control image. Never treat a rebuild or replacing
media as the repair; fix DAIMOS if the failure is in its code. A successful
fresh-image boot is useful only as a differential diagnostic.
