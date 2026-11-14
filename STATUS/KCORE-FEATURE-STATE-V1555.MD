# KCORE v1555: production DTFS DTVOL2/CLEAN read-only admission

Date: 2026-11-14. Real optional DTFS MRES implementation.
**P0-B remains OPEN**: no writable DTVOL2 transaction yet.

The production `dtfs_mount_unit` now uses the existing fresh
physical Type-551 RBN0144 directory and RBN0 reads to admit the
new six-word DTVOL2 record **only when CLEAN and explicitly
READ-ONLY**. It invokes the unchanged native v1550 parser,
requires nonzero 72-bit identity, checked identity/state/seal,
122 zero reserved words, and the VFS RO flag. DIRTY, torn,
corrupt, unreadable or requested writable V2 cannot publish a
mount. The FORMAT_V2 flag is recorded in the existing MM-backed
media word only after verification; both ID halves remain in
the existing per-slot SHAPE/EXTENT words. Old DTVOL1 and
all-zero legacy behavior remain unchanged.

The central real `dtfs_dtc_write` veneer now scans all published
DTFS mount slots by physical DTC unit before allowing any raw
DATA/DIR transfer. Any live V2 mount on the target unit denies
the write before BCACHE eviction and controller I/O, even from
internal callers or a second legacy mount on that unit. No
new owner is released and no artificial write-fault poison is
set for this intentional refusal. The v1551-v1554 physical
DIRTY services are still deliberately outside the live provider.

The C89 source-native userspace probe recognizes valid
DTVOL2/CLEAN, rejects DIRTY/bad seal/padding/ID, and keeps V1
and legacy probe behavior. Final mounted permission is always
checked again inside KCORE, not trusted from this probe.

The actual generated full-profile **KCC DTFS MRES assembly,
folding and link against the KCORE map PASS**, with v1550
validator included in `dtfs_runtime.s` by the kernel Makefile.
Boot Makefile tracks the additional validator as input.

Native testkit acceptance:

- 15 real-host-image source-linked PDP-6 V2 admission cases
  PASS, 9 unsafe source mutations rejected.
- 12 genuine PDP-6 common-write-veneer cases PASS, 7 unsafe
  source mutations rejected, including later-slot and shared-
  physical-unit V2 mount bypasses.
- 12 source-linked C89 native probe outcomes PASS.
- Legacy v1519 physical directory and mount (7 mutants),
  v1520 identity and target MKFS (5 mutants), and v1522
  mounted fault/revalidation (8 mutants) all PASS.

**Remaining P0-B feature gates:** writable V2 must be bound to
the previously tested pinned-MM first-write and already-armed
DIRTY check, with all DATA/DIR and resize/chain writes forced
through it. The provider must retain any refused MM scratch
ownership. Final CLEAN requires exclusive VFS unmount quiescence
against new FILE/cwd/PIPE/swap owners across sleeping physical
I/O, followed by fully checked physical Type-551 partial-write,
old-tape reinsertion, replacement and cold-boot guest tests.
Removable-media exchange between RBN0 check and DATA transfer
is still an unresolved safety boundary. A full guest V2 RO boot
test has also not yet been completed.

No new fixed KCORE data words, syscall ABI, or default native
formatter format. P0-A complete, P0-B OPEN, P1-C separate.
VERSION 0.5; strict 32K/optimization HOLD.

Source: `DAIMOS/design/kcore-dtfs-v2-ro-admission-v1555.md`.
Tests: `daimos-testkit/tests/validation/
daimos-dtfs-v2-readonly-admission-v1555.md`.
