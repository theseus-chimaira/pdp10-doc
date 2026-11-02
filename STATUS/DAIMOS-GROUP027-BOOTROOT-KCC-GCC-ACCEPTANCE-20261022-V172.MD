# DAIMOS group 027: DECtape and TSFS boot-root acceptance (V172)

Date: 2026-10-22

## Supported contract

TSFS is a boot-root-only filesystem. Image generation belongs in
userspace and writing media belongs to `dd`/raw DECtape tools.
Secondary `MOUNT.TSFS` is neither supported nor required. The
shared DTFS/TSFS MRES is installed only when DECtape is the selected
root; a D6FS-root kernel must not gain a resident tape filesystem
merely to satisfy a post-boot test.

## Independent KCC/GCC acceptance

The eight active group-027 targets all PASS independently under KCC
and GCC against fixed DAIMOS commit `40bc094`:

| Test | Coverage |
| --- | --- |
| dtc-online | Native DTFS created and verified in userspace, persisted on DECtape, verified again after pristine-disk reboot |
| dtc-stream-online | Production Type-551 counted two-block write/read |
| tsfs-image-online | Userspace-created TSFS image read by production driver |
| tsfs-scan | Eight-member mapping; ninth member rejected |
| tsfs-boot-eight | KINIT eight-member boot-root selection |
| tsfs-multiextent | Cross-extent and cross-member reads |
| tsfs-d6lz | D6LZ restart and transient memory release |
| dtc-reverse-online | Actual reverse-direction I/O verified by SIMH trace |

The original `dtc-online` fixture incorrectly required a secondary
DTFS mount from a D6FS root. `daimos-dtc-online-v171.sh` replaces that
oracle with a second-boot `FSCK.DTFS -O NATIVE /DEV/DTC0`, which
verifies persisted media without altering provider-residency policy.
The runner passes the selected compiler to the complete guest-image
build and uses an explicit SIMH selection. The KCC two-boot pass used
the functionally equivalent v170 runner; GCC used v171.

Evidence:

- `$TMPDIR/daimos-group027-dtc-online-v170-isolated2.log` (KCC)
- `$TMPDIR/daimos-group027-dtc-gcc-v171.log` (GCC)
- `$TMPDIR/daimos-group027-v167-*.log` and `v168-*.log` (other KCC)
- `$TMPDIR/daimos-group027-gcc-v172-*.log` (other GCC)

The DAIMOS source was tested in a DevSpace2 isolated worktree to
avoid interference from ongoing KCORE work in the main checkout.

## Remaining integration gates

Group 027's **independent non-size functional acceptance is complete**.
The complete serialized regression matrix and strict `036000`
DSH image-size acceptance have not been rerun and remain separate.
