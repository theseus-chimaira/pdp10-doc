# DAIMOS and testkit branch consolidation v1809

Date: 2026-11-22. This is a repository state and engineering
acceptance record, not a change to the supported RAW PURE modes.

## Mainline histories

DAIMOS main was fast-forwarded from v1803 495060f6 through
the isolated RAW PURE integration tip ec4ec261, incorporating
97 additional commits. The matching daimos-testkit main was
fast-forwarded from 3a4604e to 4841f2e, incorporating 34
additional commits. Neither history was squashed, rebased,
or force-pushed. All named topic heads from both repositories
are now reachable from their published mainline.

The pre-retirement exact local/remote ref and commit-ID
inventory is in DAIMOS
`design/daimos-branch-ref-inventory-v1809.tsv`.
After verifying each deletion against the updated remote main,
88 local and 87 remote DAIMOS topic refs, and seven local and
seven remote testkit refs, were retired normally. Final counts
are four DAIMOS local branches with three remote heads, and
three testkit local branches with three remote heads. Every
remaining non-main branch is checked out in a retained worktree.
No branch tip remains unmerged in either repository.

Dirty and untracked mainline files were archived before the
fast-forwards. Other dirty worktrees, historic detached archive
tags, and Devspace recovery references were not changed.
The two clean integration worktrees remain available, including
their ignored build and test products.

## Verified technical scope

The source-exact DAIMOS ec4ec261 plus testkit 4841f2e
combination passed the complete opt-in v1808 original-source
native matrix: MM owner/refusal, finalizer, K03/K04 controls,
K05 lost-success-reply prefix and FULL restore, source identity
and mode3 compatibility negative tests. A fresh default-OFF
`make build` passed, including the PDP-6 Stage-1 assemblers
and host utilities.

The earlier v1805-v1808 physical witnesses are unchanged:
256K DSK-root default-OFF boot smoke, 64K TSFS PURE boot,
and independent 48K GENERAL/STRICT RAW pressure/restore
acceptance. This repository operation did not rerun or
expand their bounded physical claims.

All selectors remain OFF by default:
`DAIMOS_PURE_MRES_ENABLED=0`,
`DAIMOS_PBM_V1667=0`, and
`DAIMOS_SWAPLESS_ONLY_V1668=0`.
Compared with v1803, the v1804 linked default-OFF candidate
had -5 fixed KCORE words and +53 removable SWAP MRES words,
or +48 words to the permanent last address. This is not a
new fixed-resident KCORE increase and is not an accepted
32K full-boot fit.

K01-K04 remain CLOSED to their documented bounded contracts,
including v1803's requester-only OOM policy. There is no
approved cross-process OOM killer. K05-K09 production RAW PURE
acceptance, physical fault-pressure, alternate-root coverage
and bitmap mode3 remain OPEN. The v1806 D6FS dirty-root
unclean-reboot limitation predates this integration.

Primary implementation record:
DAIMOS `design/daimos-branch-consolidation-v1809.md`.
Historical validation:
`design/daimos-branch-integration-validation-v1807.md` and
`design/kcore-k05-restore-reply-v1808.md`.
