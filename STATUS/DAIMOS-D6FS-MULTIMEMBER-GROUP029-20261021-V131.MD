# DAIMOS D6FS non-size regression group 029 (v131)

Date: 2026-10-21

## Root cause and testkit repair

The original multimember acceptance fixture failed in two ways. The
generated SIMH script attached fresh members 0..N and later reattached
the stale DSK1-3 media from the production boot.ini, defeating the
layout under test. The synthetic test root also installed only
INIT, LOGIN and DSH without MOUNTALL, FSTAB/SYSTEM policy or the
external programs exercised by the test probes.

The resulting `INIT: MOUNTALL FAILED` and `DSH: NOT FOUND` outputs
were fixture failures, not evidence of a D6FS implementation error.

The versioned testkit `run-multimember-v126.sh` (commit `552428c`)
constructs explicit D6FS images containing MOUNTALL, DSH support,
configuration files, and the nine external programs required by the
original 16 guest probes. Its simulator script skips the old
disk-member attachments. The supported two- and four-member roots,
the invalid unequal-member layout, and the missing-member case retain
their original positive and negative acceptance criteria.

## Verification

| Test | KCC | GCC |
| --- | --- | --- |
| D6FS HDD runtime, 16 operations, LOGSTORE badmap | PASS v128 | PASS v119 |
| Two-member interleaved D6FS (16 operations) | PASS v126 | PASS v127 |
| Four-member interleaved D6FS (16 operations) | PASS v126 | PASS v127 |
| Unequal-member layout rejected | PASS v126 | PASS v127 |
| Missing-member set rejected | PASS v126 | PASS v127 |
| Multi-member LOGSTORE verification | PASS v126 | PASS v127 |
| Dual-superblock recovery, dirty reboot rejection | PASS v130 | PASS v129 |

Independent logs are under `$TMPDIR/daimos-d6fs-multi-kcc-v126.log`,
`$TMPDIR/daimos-d6fs-multi-gcc-v127.log`,
`$TMPDIR/daimos-d6fs-hdd-runtime-kcc-v128.log`,
`$TMPDIR/daimos-d6fs-recovery-gcc-v129.log`, and
`$TMPDIR/daimos-d6fs-recovery-kcc-v130.log`.

No DAIMOS kernel code was modified. Group 029 is independently accepted
under both compilers; group 027 is deferred by request. Group 030,
the full atomic non-size matrix and strict 32K acceptance remain open.
