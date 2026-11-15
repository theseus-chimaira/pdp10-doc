# KCORE P1-E v1604: PDP-6 / KA10 swapless PURE review

Date: 2026-11-15. **Cross-machine architecture reviewed, feature
implementation NOT yet approved or accepted.** P1-C/D retain
their completed physical acceptance; VERSION 0.5 and strict 32K
size/optimization HOLD are unchanged.

## Hardware differentiation

PDP-6 DAIMOS `vm_pdp6.s` currently maps a complete private
process APR image through one 02000-octal (1024 decimal) word
aligned physical base and protection limit, stored as user words
in `vm_state` LH and physical base in RH. `vm_activate_current`
uses `DATAO 0000,vm_pdp6_apr`. Original `proc_pdp6.s` context
restoration specifically handles the separate VM_NONE fatal
executive path, and must never activate user mapping there.
PDP-6 has no independently read-only user text segment.

DEC KA10 documentation instead describes relocatable low and,
with the appropriate KT10A hardware, high user segments; a high
segment can be protected against writes. The optional high segment
is not present on every KA10. The common process contract cannot
write the existing PDP-6 APR control word on a future KA10 target.
There is currently no accepted DAIMOS KA10 full KCORE VM/backend
and no KA10 physical acceptance in this review.

Primary historical material: DECsystem-10 *Monitor Calls Manual*
(1974) section 1.2.1, and DECsystem-10 *Technical Summary* (1975)
hardware discussion. Source links and detailed design:
`DAIMOS/design/kcore-p1e-pdp6-ka10-architecture-review-v1604.md`.

## Recommended compatibility-first boundary (not approved yet)

* Common: validated immutable PURE executable source, retained
  complete RAW mutable suffix, stable u-area AC/PC/flags, exact
  word counts, one authoritative MM/process storage owner, the
  existing VFS/executable guards, and fail-closed transitions.
* Machine-specific: physical extent allocation/alignment,
  relocation/protection programming, context restore, and
  actual atomic runnable publication. Restore *full* process
  image before activating the chosen processor's mapping.
* PDP-6: quiesce a process first; omit reconstructible PURE
  words only from the inactive image, never a live executing
  segment. Use only proven MM metadata rebase and guarded
  full-image restoration with no partial-image publication.
* Initial KA10 candidate: one low segment with verified
  relocation hardware and native KA10 execution proof. Do not
  presume KT10A high segment, hardware read-only text, or
  shared HIGH code in the portable initial implementation.
* Preserve the existing raw disk/drum swap path. No new fixed
  per-process registry, no RAM compression, no kernel size
  optimization before real measured net whole-system RAM benefit.

## Actual native regression and repaired fixture dependency

Three historic isolated native PDP-6 PURE/MM test runners had
ceased linking: current original `proc_storage_runtime_v30.s`
references `vfs_namespace_root` in the appended real root-fatal
classifier. The symbol belongs to the aliased original
`system/kernel/drivers/device_state.s` accounting BSS. All three
runners now link that **complete original** file, not a fake
test symbol or a changed fatal classifier.

The fresh current-source native outcomes are PASS:

| Test | Status |
|---|---|
| v1381 owner-checked in-place PURE representation | PASS |
| v1384 APR prefix reserve/restore | PASS |
| v1395/v1397 MM/pin refusal and rollback | PASS |
| v540 real MM compact/VM APR activation/runqueue | PASS |

See testkit
`tests/validation/daimos-p1e-pure-prototype-regression-v1604.md`
for invocation and preserved output logs. These tests do **not**
constitute scheduler-integrated PURE, actual swapless cold boot,
physical KA10 or peak 32K RAM acceptance.

## Current gate

**Review complete; P1-E implementation stays HOLD** until the
proposed machine-neutral owner/source transaction and separate
per-CPU restore contracts are approved, with actual KA10
target-specific evidence required before enabling that target.
P1-E needs real PDP-6 swapless low-memory owner/PI6/EXEC/WAIT,
MM and failing-source tests, and a future KA10 low-segment VM
boot/restore regression. P0-B Type-551 physical reel-retention
remains independently OPEN. No production code, permanent RAM,
ABI, media format or 32K optimization change in this review.
