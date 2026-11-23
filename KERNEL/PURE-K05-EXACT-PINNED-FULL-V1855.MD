# PDP-6 RAW PURE K05: exact pinned FULL commit owner check v1855

Date: 2026-11-23. Scoped native PDP-6 and normal 48K whole-machine
PASS; K05-K09 OPEN. Production source fix: DAIMOS `1893c84b`.
PURE and optional mode3 are disabled by default.

## Correctness defect and repair

The original FULL transaction used the success of
`mm_unpin(target)` as evidence that the provisional complete PROCESS
extent was now unpinned. That call removes **one** pin only. When a
loader callback acquired a second target pin during FULL, the original
transaction unpinned once, freed the authoritative PACKED RAW source,
and could publish a still-pinned PROCESS APR. A new source-linked
KCC-built PDP-6 native testcase actually reproduced that defect with
original native exit `000000002002` (octal), before the correction.

The corrected FULL transaction, inside its existing short PI-excluded
commit window, now requires an exact original PACKED source extent
and one separate complete PROCESS target extent, both owned by the
selected slot, both with exact expected sizes, and **exactly one**
pin each. Unexpected pins, missing/duplicate/wrong-sized extent,
foreign owner or alien type fail with `MM_ERR_BUSY` without unpinning
or freeing the authoritative owner, publishing RESIDENT, or clearing
the process's TRANSITION quarantine. This does not guess the number
of pins belonging to the restoring code.

The new check reuses a *single* authoritative MM extent scanner with
the v1839 failed-source-free result reconciliation. Precommit
expects pin=1 for both known owners. If `mm_free(PACKED)` returns an
error after both verified unpins, its recovery pass expects pin=0:
sole exact target means deletion actually committed; source plus
target means rollback can safely release only the provisional APR;
anything else quarantines. This avoids a duplicate scan in optional
PURE MRES, no extra resident process word, owner registry, BSS or
new KCORE code.

## Independent native and physical evidence

Testkit `tests/validation/pure-k05-pinned-commit-v1855/` compiles
the current production DROP, PREFIX and FULL transactions for native
PDP-6 with KCC, preserving the original MM PURE assembler and
storage-map interfaces. It checks two persistent `MM_ERR_NOMEM`
allocator refusals followed by healthy retry, positive FULL commit
beside a separately owned pinned PROCESS bystander, and deliberate
second source/target pins leading to exact TRANSITION quarantine,
untouched RAW suffix and unchanged bystander. Two independently
compiled unsafe production mutations (no precommit check and weak
pin acceptance) are executed and rejected at `000000002002`.

The complete native suite `make test-pure-integration-native-v1855`
PASSes, including original v1677-v1839 fault/mutation matrices.
The historical v1654 late-unpin mutant source anchor was adapted
to the new intervening pin validation *without removing the guard*;
its actual unsafe late-publication ordering still fails the native
oracle. That testkit-only compatibility correction is `23ea31c`.

The independently source-built opt-in PURE 48K PDP-6 TSFS-root guest
also boots a real five-child pressure workload (one root-backed and
four secondary DTC3-backed compressed PURE children), and completes
RUN/PIPE/EXIT/WAIT with all five `w` markers, `Z`, then HALT PC 0:

    aaaaax[B013060/043565]<010000+010000>[A013060/043565]{013060-013060}LwwwwwZ

The simulator/lifecycle runner exits zero. This is normal physical
restoration, NOT physical injection of a second MM pin. Do not claim
whole-machine quarantine/bystander scheduling from the native case.

Matched linked RAM sizes: fixed KCORE DXR is byte-identical to
v1845/v1847/v1848 (SHA256
`33bf700218a73fefcb36ebb1f1d835c1817f3a291a642898ccb3cbd5d9039d8b`).
Opt-in PURE MRES image grows `004000` to `004027` octal words,
**23 decimal words**; BSS remains zero. The physical guest budget
changes `043536` to `043565` octal, agreeing with the exact MRES
increase. New PURE DXR SHA256
`a0fb25c51cf1c57f4bc70ca1dc83d8b9ea39b1ac3ddb3bba7b9ffcea17e5b3fd`.
Original physical console trace SHA256
`c090f8c2aa9475d876286f37c7cd6ab036a4f77b64353730f98a785c95335c8e`.

## Limits and next K05 gate

Source-linked native failures prove transaction fail-closure; the
48K physical run proves healthy integration after the correction.
Outstanding gates: genuine 48K physical injected extra-pin and
permanent allocator/free uncertainty, independent scheduling progress
beside quarantined processes, all PI6 timing interleavings,
continuous physical peak, alternate roots and system profiles.
K05-K09 remain OPEN; there is no basis to enable PURE by default.

DAIMOS state: `design/kcore-k05-exact-pinned-full-v1855.md`.
See also `KERNEL/PURE-MM-UNCERTAIN-V1658.MD` and
`KERNEL/PURE-K05-MIDFULL-PI6-V1848.MD` for other acceptance boundaries.
