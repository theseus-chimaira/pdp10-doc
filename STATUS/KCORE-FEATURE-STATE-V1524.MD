# DAIMOS KCORE v1524: physical DECtape block-zero first-mark correction

Date: 2026-11-12. v1524 is a **verified simulator correctness
increment** discovered while accepting the v1522/v1523 KCORE
non-root removable-media features, not a new KCORE facility,
filesystem format change or RAM optimization. DAIMOS 0.5.

## What actually failed

The new persistent native DTFS identity introduced in v1520
resides in the already reserved physical RBN0 of its tape.
The full v1523 tape-root guest booted and reached DSH but
the first secondary DTC3 target MKFS or FSCK command did
not return in the diagnostic window.

Isolating the Type-551 DTC3 path with the **real** DAIMOS
physical driver and a complete host-produced native DTFS
image exposed a precise native failure:

| DTC3 128-word transfer | Old installed SIMH | Corrected SIMH |
| --- | --- | --- |
| Physical RBN0 | FAIL first write | PASS write/read and compare |
| Physical RBN1 | PASS | PASS |
| Physical RBN0144 | PASS | PASS |

This is not a fabricated media flag or a cache hit. A
freshly selected, stationary Type-551 transport started in
`DTB_REQ` and reached the **first forward RBN mark** before
SIMH had promoted the request to `DTB_ACT`. The simulator's
`FNC_SRCH` handler therefore did not deliver RBN0 to the
already prearmed Type-136; later marks worked.

The corrected SIMH `PDP10/pdp6_dtc.c` promotes pending
`FNC_SRCH` to active **before the first mark is forwarded**.
The Type-136 protocol, existing DAIMOS tape search, DTFS
format and kernel source were not changed.

## Executed verification

- SIMS PDP-6 rebuild with `-Werror` and simulator register
  sanity test PASS.
- `daimos-testkit/tests/validation/daimos-dtc-first-mark-v1524.sh`
  actual physical native DTC3 one-block RBN0/1/0144
  read/write/check all PASS.
- Known pre-fix simulator reproduces a native `__test_exit`
  assertion failure **specifically at RBN0**.
- Existing real DTC counted two-block stream, reverse
  transfer, simulator DTC select/detach status and
  data watchdog tests PASS with corrected SIMH.
- Existing v1522 native mounted-ID revalidation and
  v1523 actual process FILE/cwd/unmount scans PASS
  with their negative controls.

`sims` commit `96394c5` is pushed. The corrected
`pdp6-v1524` was installed from a versioned local
`local.tar.gz` package into `PDP10_PREFIX`;
previous `pdp6-v1470` was preserved for regression
comparison. **Zero new KCORE resident instructions or
allocated/data RAM words.**

The full tape-root guest was observed doing extensive
DTC0 root-volume operations after the MKFS command was
entered, but no actual DTC3 request was observed before
the diagnostic ended. A separate 256K KCC auto/disk-root
guest **did** physically run target MKFS on DTC3 and
reported `FSCK.DTFS NATIVE OK`.

That disk-selected root deliberately omits DTFS/TSFS
MRES at runtime by `module_minit.c` policy, despite
linking it into the build. `MOUNT.DTFS` was therefore
refused; a subsequent STAT of the pre-existing D6FS
`/MOUNT/DT0` directory is not proof of a DTFS mount.
The old v578 full-guest runner **falsely returned PASS**
despite that MOUNT error and subsequent UNMOUNT errors.
The runner has been tightened to check the actual
FSINFO/MOUNT inventory, command error reports, and
post-unmount disappearance. An independent negative
disk-root policy test requires the intended refusal.
Its first rerun reached a pre-login `?RT` HALT
on previously used scratch root media. After
regenerating the disposable D6FS root image,
the exact negative-policy full guest **PASSED**:
target MKFS/FSCK succeeded, MOUNT.DTFS refused,
and actual MOUNT inventory showed D6FS/MEMFS
but no DTFS. Installed-DTFS mounted/FILE/dirty
recovery therefore remains OPEN. Enabling optional
DTFS MRES on disk roots only to make a test pass
would unnecessarily increase resident RAM and is
not part of this change.

## Outstanding KCORE feature gates

1. **P0-A:** authoritative root/BACKSTORE/selected TSFS
   member permanent-loss evidence and qualified v1515
   HALT caller. Ordinary timeout or PI/status error is
   not positive evidence of permanent loss.
2. **P0-B:** full booted physically mounted DTFS failure,
   matched/mismatched 72-bit identity return, real FILE/
   cwd lifetimes and dirty/partial-write write-uncertain
   recovery. Never silently accept legacy zero-ID media.
3. **P1-C:** remaining genuinely unaccepted cross-provider
   DEVICE/VOLUME, cache, LOGSTORE/MEMFS and swap faults.
4. **P1-D:** reconcile the existing local FSINFO/MOUNTS/DF
   1/4/8/26-source changes with production and physically
   accept mounted holes/errors and dynamic inventory.
5. **P1-E:** production uncompressed swapless inactive
   PURE drop, MM rebase and full APR restore with
   atomic unwind under process lifecycle/pressure.
6. Feature regression freeze and documentation; optional
   mutable-only compression **only** if measured net RAM
   wins. Strict 32K and optimization remain on HOLD.

Detailed primary source in
`DAIMOS/design/kcore-dtc-first-mark-v1524.md`;
testkit `daimos-dtc-first-mark-v1524.md`. Supersedes v1523
only on the observed transport/simulator physical defect.
