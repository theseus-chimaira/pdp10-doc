# KCORE v1563: DTFS staged PURE executable owner exclusion

Date: 2026-11-14. The native 256K KCC image and source-linked PDP-6
owner regressions pass. **P0-B remains OPEN; DTVOL2 remains READ-ONLY.**

## Problem and change

After v1562, the v1562 VFS unmount barrier blocked new FILE/cwd and
ordinary provider admissions but not executable backing publication.
RUN or EXEC can finish loading a DTFS executable after an earlier
filesystem read sleeps. An executable PURE backing record can exist
while a child is still SIDL and before it owns its stable u-area;
the earlier process-swap mount scan only considered processes with a
live u-area. In-flight processes were therefore missed by a VFS
unmount owner scan.

The production `proc_swap_attach()` now calls the real, non-sleeping
v1562 DTFS mount-owner guard before changing any executable swap
record or its fifth mount-ID bit. The two native PURE vnode/mount
ownership scanners consider SIDL first, including processes before
u-area allocation and processes with an attached but not yet
populated u-area. SIDL uses the already-reserved scheduler RH bit
0 for public mount slot 17..26; SRUN and swapped PURE continue using
the stable credential/saved-backing paths.

The production attach and detach paths synchronize this SIDL staging
bit even if a u-area already exists, without clobbering a runnable
SRUN queue link. No new kernel mount slab, process record or fixed
DATA/BSS allocation is needed.

## Acceptance

- C89 host check of actual attach, including denied DTFS V2, unchanged
  swap backing on refusal, high mount-ID staging and other providers.
- **11 native KCC executable-attachment + actual VFS admission cases
  PASS; 2 unsafe mutations rejected.**
- **26 native PDP-6 source-linked PURE vnode/mount owner scenarios
  PASS; 6 unsafe mutations rejected.** Cases cover public IDs 1/16/17/26,
  SIDL before/after u-area ownership, resident and swapped SRUN owners
  and the non-PURE boundary.
- Earlier host v960/v961, native v1382 PURE/MM, v1523 FILE/cwd, v1562
  DTFS mount/unmount and PIPE FIFO mount reference v19/v417 PASS.
  v1084 pinned FIFO OPEN rollback native PASS after fixture link and
  exact MM_ERR_BUSY (-5) expectation were aligned with production.
- Full KCC full-256K KCORE and DTFS MRES image build PASS. The fixed
  KCORE low-end moves from octal `024266` to `024330`, **+34 TEXT
  words**, no new persistent owner/data words. Strict 32K still HOLD.

## Remaining P0-B

The existing PIPE/FIFO paths first publish a FILE owner or retain a
named FIFO object whose mount ID participates in VFS owner scans.
Their independent mount/rollback regressions pass. Real asynchronous
full-guest scheduler overlap across FILE, PIPE, staged EXEC/RUN and
provider I/O has not yet been demonstrated. The final physical
DIRTY-to-CLEAN readback seal, removable Type-551 media generation,
partial writes, reinsertion and cold boot remain mandatory gates.
There is no writable DTVOL2 cutover and no 32K acceptance.

P0-A complete; **P0-B OPEN**; P1-C/D/E unchanged. VERSION 0.5.

Sources: `DAIMOS/design/kcore-dtfs-pure-owner-v1563.md` and
`daimos-testkit/tests/validation/daimos-dtfs-pure-v1563.md`.
