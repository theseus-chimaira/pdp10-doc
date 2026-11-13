# KCORE feature state v1539: one TSFS root member physically absent

Date: 2026-11-13. This is a completed physical acceptance gate for
the v1538 selected-root fail-stop policy, not a new KCORE feature or
a claim that 1960s Type-551 hardware detects permanent media loss.

The actual full 256K KCC PDP-6 with current TSFS/DTFS MRES boots
from a **three-member** TSFS root on DTC0, DTC1 and DTC2. PID1
accesses its root and prints `ROOTLOSS V1538 READY_DETACH`.
The v1539 host variant of the already accepted v1538 physical
root-outage test removes **DTC0 only**; DTC1 and DTC2 remain
attached. The guest resumes with instruction-delayed CTY input
and performs real uncached file-data reads from `/SYSTEM/INIT`.

The PDP-6 halts at `003743` (octal), the exact fixed KCORE v1515
root-specific HALT PC from the linked map. The acceptance oracle
rejects return from a failed root syscall, a user process HALT at
zero, a missing HALT and a wrong BACKSTORE/invalid-reason PC.
The v1539 run PASSed twice, including a second execution after
the version-specific transcript labeling was introduced.

Related acceptance, all on the revised root policy:

- Native source-linked D6FS 11 cases/5 source mutations and TSFS
  12 cases/4 mutations demonstrate selected-root fault versus
  secondary/EOF/bad-argument handling.
- The v1538 full-guest detached *all three* selected TSFS media;
  the required root data read HALTed at the same PC.
- Rerun of v1534 full physical mixed TSFS root, nonroot DTC3/DTC4
  DTFS and MEMFS demonstrated nonfatal DTC3 failed write, healthy
  other storage and checked return/retirement.
- Native v1513/v1515/v1516/v1517/v1537 and TSFS slab/multiextent
  regressions PASS.

No DAIMOS source changes in v1539, no BSS/permanent RAM change,
no change to on-media representation or user-facing ABI.
VERSION remains 0.5. Optimization/strict 32K HOLD.

P0-A is **not fully closed**: actual physical D6FS disk/drum root
failure acceptance, independent BACKSTORE unavailable policy and
critical-shared-device tests remain open. Root availability is a
service decision after bounded normal I/O recovery, **not** a
heuristic permanent physical-device-loss detector.

Primary testkit evidence:
`tests/validation/daimos-root-member-outage-fullguest-v1539.sh`
and `tests/validation/daimos-root-member-outage-v1539.md`.
Prior DAIMOS source/behavior report:
`design/kcore-root-io-failstop-v1538.md`.
