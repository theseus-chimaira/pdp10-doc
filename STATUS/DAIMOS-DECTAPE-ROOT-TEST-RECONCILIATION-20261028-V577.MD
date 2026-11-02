# DAIMOS native DECtape regression reconciliation, v573–v577

Date: 2026-10-28.

## Root-selected module contract

`module_minit.c:dtfs_minit()` explicitly does not install the DTFS/TSFS
MRES when `root_select_class() != KINIT_ROOT_DTC`. A D6FS disk-root test
cannot demand a successful runtime `MOUNT.DTFS`, even if the kernel's direct
DECtape I/O and userspace `FSCK.DTFS` work. Under an actual tape-selected
root the provider is installed. `MOUNT.TSFS` is not an available executable
in the current userland; historical test fixtures referring to it are stale.

## Accepted regressions

- **v573**: `daimos-dtc-online-v1.sh`, two fresh disk-root boots with
  preserved Type-551 media, formatting and native checking on boot one,
  independent `FSCK.DTFS NATIVE OK` on boot two. Both disk-root `MOUNT` and
  `MOUNT.DTFS` reject as specified by root-selected module policy. PASS.
  Testkit `e73ebb7`.
- **v575**: `daimos-tsfs-scan-v2-test.c` validates eight-member direct
  scanner result and metadata, and rejects a ninth member; deleted the
  old `SYS_TSFS_*` mount-handoff ABI assertions, since that ABI was retired.
  PASS, testkit `878657d`.
- **v577**: `daimos-tsfs-root-directory-v577.sh` uses a real tape-selected
  TSFS boot and extended root manifest to validate `/` and nested
  directory lookup, `LS`, regular-file and directory `STAT` on SIMH.
  PASS. Matrix target `test-daimos-tsfs-dir` now uses this supported path
  instead of retired secondary `MOUNT.TSFS`; testkit `8a66c84`.

## The meaning of early `?RT`

`kinit_boot()` emits `?RT` if root mount/validation fails or the selected
root does not contain a regular `/SYSTEM/INIT`. It is **not** a special
second-boot diagnostic. V573 completed a second boot successfully after
restoring the pristine DSK root but retaining formatted Type-551 media.
Reusing mutated media without establishing the state and selected root
can lead to an independent root failure; determine the precise cause from
root-selection and filesystem evidence rather than treating every `?RT`
as a generic DTC error.

A positive secondary native DTFS mount under a tape-selected root is a
separate acceptance objective and is not claimed passed here.
