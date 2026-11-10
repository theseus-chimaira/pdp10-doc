# v1406: strict PDP-6 256K guest acceptance of checked SWAP releases

Date: 2026-11-10.

The committed DAIMOS `e4c53780` v1405 SWAP-MRES source was booted again
from a fresh disposable 256K KCC PDP-6 DSK/D6FS guest with the original
strict testkit v1349 F07 suite. This run reached the original SIMH HALT
and EXIT, passed the original complete after-HALT postflight, and passed
both independent source-linked real-MM owner lifetime and physical
no-partner checks. No original guest assertion was disabled or relaxed.

## Whole-guest results

| Checked condition | Observed |
| --- | ---: |
| Successful swap-ins | 34 |
| Genuine MM allocation failures | 1 |
| No-partner cooldowns | 1 |
| Swapped-process exchanges | 0 |
| Outstanding process swap blocks at HALT | 0 |
| Outstanding BACKSTORE blocks at HALT | 0 |
| Independent 8K-word MM extents prepared | 4 |
| Independent MM extents freed | 4 |
| Remaining independent MM owners | 0 |
| Independent owner-accounting errors | 0 |
| Invalid owner-evidence negative controls rejected | 6 |
| Real process slots observed at OOM | 24 |
| Resident process extents at OOM | 8 |
| Fatal held-provider BUSY attempts | 3 |
| Normal held-provider return | 1 |
| Genuine spinner progress events | 8 |

The first physical-MM sample observes the four allocated dynamic extents
and a live process pin, before the real shortage. Sample 13 observes the
genuine `MM_ERR_NOMEM`/no-partner cooldown and independent release:
`prepared=4`, `freed=4`, `owner_count=0`, `error=0` and the process pin
released. The final HALT owner fields are
`prepared=4 freed=4 remaining=0 error=0`. The strict original F07 suite
also verifies 18 child RUN/ACK plus TERM/WAIT operations, the requester-
only fatal BRK OOM, protected witness, serialized filesystem provider,
foreign waiter and normal holder unlock.

## Source and evidence

- DAIMOS commit: `e4c53780a15ef2dc6b1cac8666c0e85f2da2c2ae`
- daimos-testkit commit: `ac4079e48afebc14b4bc4440026d8b22aa9ba7cc`
- Full-guest runner: `daimos-swap-physical-final-v1349.sh`
- Native checked-release source test: `daimos-swap-in-checked-release-v1405.sh`
- Versioned archive: `$TMPDIR/daimos-swap-checked-release-v1406-evidence-r1.tar.gz`
- Archive SHA256:
  `536c5069aec9fadffcad6381db3f530ded3d2fbd819a754c7b6788c52f626dc7`

The archive includes the actual guest `addresses`, `kcore.map`, `simh.out`,
live `simh-sampler.log`, original F07/owner postflight reports and
checksums, exact reproduction runner, relevant source and a SHA256
manifest. The file manifest was independently verified before packing.

## Boundaries and footprint

This is an accepted **normal physical raw swap/OOM/no-partner lifecycle**
for the tested KCC 256K PDP-6 profile, including the v1405 checked
BACKSTORE free ordering. It does not imply that a corrupted allocation
bitmap will be automatically repaired. The native v1405 direct-call test
proves failure-safe rollback and a manually driven retry; the production
slot-0 service still routes negative `proc_swap_in` results to
`SYS_EVENT_TERM`, which needs separate injected-failure acceptance.

No additional production changes were needed for v1406: fixed KCORE is
9687 words, movable SWAP MRES is 1762 words, and the strict 16K
single-root permanent-placement gate remains 31 words over. General
privileged MEMFS raw-XIP RUN/EXEC, swapless PURE-RAM scheduler integration,
and the grouped D6 codec are not part of this guest acceptance; the
codec remains deferred and production D6LZ36 unchanged.
