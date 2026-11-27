# KCC extension scope revision v60r

Date: 2026-11-27. Extends the v60g exclusions documented in
KCC-FRONTEND-REPAIRS-20261127-V60K.MD and the v60q status in
KCC-LEGACY-DECLARATIONS-AND-CONDITIONAL-20261127-V60Q.MD.

The following GNU extensions are explicitly outside KCC's support
scope, by project decision:

- Extended inline assembly with operand constraints, matching
  constraints and clobber lists (seven sources, 154 original cells).
  Existing native KCC assembly facilities remain supported.
- Over-aligned typedefs requesting alignment beyond one target word
  (three sources, 66 original cells). Ordinary supported alignment
  and member layout remain unchanged. No promise of arbitrary typedef
  alignment is made.

Testkit a82c46e marks the affected KCC profiles unsupported, not PASS.
The cumulative manifest is
manifests/PDP10-KCC-UNSUPPORTED-SCOPE-20261127-v60r.tsv
(57 source registrations, 1,220 original failed cells).
The historical v60g manifest remains intact.

The before/after runner selections confirm exactly 220 fewer KCC
cells: 40,798 to 40,578. GCC remains 85,799 cells. The ten affected
sources now select zero KCC and 440 GCC cells. A field-level matrix
comparison verifies that only 110 KCC profile entries changed
(each selects two optimization cells). No unrelated matrix entries
or test sources changed.

Original KCC compile_fail cohort: 2,196. Explicitly unsupported:
1,220. Repaired and verified at v60q: 430. Remaining active: 546.
This is a scope change, not a compiler fix or fresh full-matrix sweep.
The installed KCC remains 7efd7b6 (v60q).

Selection logs and exact newly excluded and remaining cell manifests
are under TMPDIR/kcc-exclusions-v60r.UfP4yw.
