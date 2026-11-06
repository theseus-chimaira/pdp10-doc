# F07 v1274: full 64-slot physical OOM and descriptor-pressure acceptance

## v1275: bounded timeout for subsequent 64-slot regressions

The accepted v1274 guest completed within the original 900-second
deadline. Subsequent full 64-slot F07 tests now allow 1800 seconds
(`daimos-testkit` v1275 `4a6a3c0`). This setting applies only when
`V1271_TEST_FULL_SLOTS=1`; the 24-process F07 and smaller standalone
guests retain their earlier timeout limits. The extended deadline does
not change admission markers, physical OOM classification, snapshot
checks, negative mutants, or cleanup/reap requirements. A longer
deadline alone cannot turn a stalled guest into accepted evidence.

## Release scope and result

The fresh 256K KCC PDP-6, DSK Stage 1, four-member D6FS/disk-root
S07/S08 release-profile guest PASSED. It was built from clean DAIMOS
`226e7a5f` with disposable testkit source `027fc3f`; the subsequent
independent strengthened verifier is committed in testkit `89a8efa`.
The guest retained the original 900-second emulator deadline and
reached the expected HALT without extending it:

```
<+++++++++++++++++++++++++++++++++++++++++++++++++++++++++^@%&*?{}|~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~KWP
HALT instruction, PC: 021377 (HALT 21377)
```

The 57 `+` acknowledgments come only from distinct successful real
RUN and BRK children, PID 7..63. After the original physical BRK OOM
selected only requester PID 3, all 57 extra children received actual
TERM events and each was reaped by a separate WAIT/status check (`~`).
INIT, witness, gateway waiter, spinner and original held filesystem
lease owner survived until their required normal release stages.
Final FILE/MM/process/physical swap backing accounting passed.

## Quantitative simultaneous OOM snapshot

| Measurement | Observed |
| --- | ---: |
| Configured process slots (slot 0 is kernel) | 64 |
| Live user processes | 63 |
| Actual resident APR process extents | 55 |
| Nonresident/swapped user processes | 8 |
| Allocated MM descriptors | 127/136 |
| Allocated managed physical words | 239576/242085 |
| Managed free physical words | 2509 |
| Largest free gap | 760 words |
| Exact summed resident BRK/APR rounding | 31642 words |
| Actual disk swap blocks allocated at OOM | 232 |
| Shared FS/provider workspace | 128 words |
| Private kernel stack high-water/capacity | 180/206 words |
| Idle kernel stack high-water/capacity | 63/256 words |

The original provider owns its lease at the same synchronous MM
snapshot. The v1150 and v1159 error fields are zero; only PID 3 is
selected for fatal OOM. The v1203/v1205/v1207 independent verifiers
reconcile the actual MM extent table, VM/APR mapping, each real user
descriptor and its distinct logical BRK, real disk-backed swapping,
the 128-word serialized workspace and stack watermarks.

Unlike the separate early v1204/v1206 occupancy experiments, this is
one genuine guest with maximum process-table load and near-full MM
descriptors, resident and swapped APRs, held provider and a physical
requester OOM decision simultaneously. No process slots or MM
descriptors are synthesized by the checker.

## Test-only load design and negative controls

The first 64-slot trial copied INIT's old 16K-word BSS pressure array
into all 57 additional executable instances. It progressed to 18
admissions in about seven minutes, but created unrealistic swap-region
pressure for a maximum-slot test. This trial was controlled-stopped,
not reported as a kernel hang. The final **64-slot mode only** uses
a 1K-word BSS with a verified corruption sentinel; other F07 modes
retain their original 16K-word array. Real distinct per-child BRKs,
APR rounding, swap backing and final resource accounting remain
mandatory. The accepted 64-slot test met the same 900-second limit.

The v1274 verifier requires every user slot to retain a stable
u-area, near-full physical MM and descriptor occupancy, the correct
foreign provider workspace, valid private/idle stack bounds, real
swap use and zero final backing leaks. It rejects 13 mutated forms
of the captured guest evidence (including a missing last-child u-area,
wrong logical BRK, reduced descriptor occupancy and stack overrun).
The earlier 24-process regression also passes the tightened verifier.

## Evidence and remaining OOM gates

Archive: `$TMPDIR/daimos-f07-64slot-acceptance-v1274-r1.tar.gz`

SHA256:
`0fb22920bfd9f66c155ef3bb61f8db0c0f146f80f89bbc3e6e1b3406d9599e1b`

The archive retains original boot configuration, maps, build log,
SIMH output, profiles, host tool and media checksums, raw counters,
and strict v1203/v1205/v1207 observations. Every evidence manifest
entry has independently verified against the saved bytes.

**F07 is CLOSED for the bounded S07/S08 256K release profile.** This
does not close E13 (independent exact-revision host/media coverage),
F11 (final exact-candidate gate review), or F12/F13 (production-enable
and rollback). The production requester OOM flag remains OFF. The
separate permanent KCORE/MRES 16K fit is not established by this
dynamic 256K guest.
