# KCORE v1542: real physical BACKSTORE failure HALT accepted

Date: 2026-11-13. KCORE features, not optimization.

The common v1541 fixed BACKSTORE gateway treats failed required
backing I/O, after normal driver recovery, as an unrecoverable
system-service loss. Enabled BACKSTORE and disabled-but-owned
BACKSTORE are both critical. Invalid requests and inactive/unowned
optional snapshot probes are not. No permanent media-loss detector
or additional state word is necessary. This is the same selected-root
fail-stop policy already accepted in v1538.

## Physical Type-236 / PI2 acceptance

In a complete 256K KCC PDP-6 disk-root guest, a test-only instrument
selected the first genuine owned BACKSTORE WRITE and forced its
real Type-236 drum completion through the existing PI2 error handler.
The original KCORE gateway then HALTed immediately at the exact
BACKSTORE reason-2 PC, octal **003747**; the root HALT is 003743.

Current linked v1515 entry = 003740. The actual post-HALT native
controller error count = 1, BACKSTORE enabled = 1, owned blocks =
octal 0320; four live pre-error snapshots have zero injected errors.
The verifier accepts this physical evidence and rejects nine
independent corrupted source/evidence mutations. This is a real
PI2 completion and KCORE HALT, not a direct software fake return.
It does not claim a permanently detached physical drum.

Older v1511/v1512 negative completion paths properly retained
owners and allowed scheduler retry. Their **final negative I/O**
continuation is intentionally superseded by the explicit strict
fail-stop policy, while successful internal driver recovery before
return remains accepted. No process state was fabricated or released.

Native source-linked v1541 had separately PASSed 17 cases and six
mutations for direct DSK270/DRM236 and mapped BLOCKSET VOLUME,
success/invalid/probe controls, and disabled-but-owned service.
No new production source, resident DATA/BSS or ABI in v1542.

## Remaining P0-A

Physical full-guest alternate DRM-root profile, unrelated secondary
DSK/DRM failures and mixed root/BACKSTORE shared DEVICE/VOLUME
faults remain. P0-B and P1-C/D/E remain distinct feature work.
VERSION 0.5; strict 32K/optimization HOLD.

DAIMOS: design/kcore-backstore-physical-failstop-v1542.md.
Testkit: tests/validation/daimos-backstore-physical-failstop-v1542.md.
