# v1336: real physical no-partner pressure observed in booted PDP-6

Date: 2026-11-07. Scope: observed intermediate state in a genuine
256K KCC PDP-6 guest. **Not full recovery or final guest acceptance.**

The existing testkit v1327/v1334 fixture reserves four real 8192-word
`MM_TYPE_KERNEL_DYNAMIC` extents through the unmodified production
`mm_alloc_aligned_noreclaim` before starting the 18 additional
24K-word children. These four allocations occupy 32K physical words;
the actual MM and swap code, process representations and disk root
are unchanged. Read-only SIMH samples capture linked kernel symbols.

At the twelfth sample of the v1335 run, after 17 admissions:

| Observation | Decimal count |
| --- | ---: |
| Successful swap-outs | 54 |
| Successful swap-ins / restores | 37 |
| Actual swap-in resource shortages | 93 |
| Exact MM resource errors | 93 |
| No-partner cooldown entries | 93 |
| Exchanges | 0 |
| Fatal swap-ins | 0 |

The last MM error was the 36-bit signed `-1`, meaning
`MM_ERR_NOMEM`, for a 26624-word process image owned by slot 5.
Both LOW and HIGH free-fit results were zero. No fallback was
present or executed. The kernel progressed through periodic PI6
clock samples while repeatedly preserving disk-backed processes
and entering the production no-partner cooldown branch.

The independently committed `daimos-swap-live-pressure-v1336.py`
testkit verifier decodes the raw SIMH trace against the linked
symbol address manifest, checks MM typed-error, requested length,
owner and free-fit fields, and checks pressure, cooldown, no-exchange
and read-only sampler agreement. Eight unsafe mutated counter sets
are rejected. It deliberately does not accept interrupted guest
traces as full-system lifetime evidence.

In this live-pressure situation the probe named `swapout_fail`
counts unsuccessful **reclamation attempts** reaching the
`proc_swap_out_fail` label, not necessarily fatal published swap
transactions. This field cannot be interpreted using the normal
full-guest final-cleanup rule at an intermediate snapshot. This
observation does not relax final leak-free or rollback requirements.

## What remains open

INIT was synchronously waiting for the eighteenth child RUN/ACK,
while also owning the four physical MM reservations. The next
admission could not complete under the reserved memory pressure,
and INIT could not reach its later owner-release operation. This
creates a test workload circular dependency. The guest was
interrupted, with no complete HALT, independent pressure release,
subsequent disk restore or zero-leak final accounting.

The staged v1336 independent-process fixture uses the existing
real spinner process, with a genuine `mm_pin`/`mm_unpin` on its
resident process extent. Once a real cooldown is observed, that
user process requests release of the four test-only extents through
the original exact-owner `mm_free` path. This fixture **must** pass
a fresh booted full-guest test and strict after-HALT verifier; its
source preparation alone is not acceptance.

No production KCORE/MM/SWAP/MRES code or footprint was changed.
The historical speculative 16-word v1208/v1211 fallback remains
excluded until a genuine production defect has been demonstrated.

Evidence on the Termux host:
`$TMPDIR/daimos-swap-early-owner-pressure-v1335-evidence-r1.tar.gz`.
The archive has a verified internal SHA256 manifest and includes
original raw SIMH observations, linked addresses and kernel map.
