# DAIMOS D6FS HDD LOGSTORE: host member-count regression repaired (v119)

## Failed acceptance

The GCC non-size group-029 HDD runtime exercised basic D6FS operations
successfully, then failed with `logstore: invalid badmap` during post-guest
LOGSTORE inspection.  It was not an error returned by the guest D6FS mount.

The testkit's `tests/system/d6fs-hdd-v2/run.sh` built a **four-member**
interleaved D6FS image but subsequently invoked `logstore --dump -n 1`.
The one-member mapping points the host BADMAP reader at the wrong physical
sector.  A direct host comparison on the same disk set confirmed that
`logstore -n 1 --dump` failed with `invalid badmap` while
`logstore -n 4 --dump` succeeded and returned valid metadata.

## Corrected test and acceptance

Testkit commit **`16ccd39`** changes the LOGSTORE post-check to `-n 4`.
No DAIMOS kernel, D6FS on-disk layout or LOGSTORE implementation changed.

Fresh 256K PDP-6 GCC acceptance **v119**:

- `d6fs-hdd-runtime` exit status 0
- 16 of 16 guest D6FS filesystem probes PASS
- Host LOGSTORE check PASS with the correct member geometry

Evidence: `$TMPDIR/daimos-d6fs-logstore-v119/d6fs-hdd-runtime-v119.log`.

This validates **one** formerly failing subtest of group 029.  The complete
group, including multi-member, recovery and other checks, has **not** been
reaccepted by this run.  32K and dedicated size tests remain deferred.
