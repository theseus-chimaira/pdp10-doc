# KCORE v1566: actual owner-exclusive Type-551 DTVOL2 CLEAN seal

Date: 2026-11-14. **Native and real physical CLEAN PASS.**
P0-B remains OPEN; the mandatory production DTVOL2 read-only guard
continues to prohibit writable media.

## Implemented physical durability prerequisite

The DTFS provider now registers a private CLEAN operation (octal
027) invoked by resident VFS UNMOUNT only when the v1562 per-mount
quiescence barrier is held and the V2 volume has previously verified
an on-media WRITE_INTENT. The service independently rechecks its
published vnode/provider/slot, VFS RW policy, saved 72-bit volume ID,
normal native media, and lack of retained MM scratch ownership.

The v1552 owned MM lease now supports CLEAN using its same dynamic
128-word scratch, PIN across physical Type-551 I/O, and strictly
checked unpin/free. Immediately after PIN and before any raw media
operation, it marks the mounted unit WRITE_UNCERTAIN. The raw CLEAN
service then rereads the original DIRTY RBN0 (not BCACHE), requires
a complete valid record and both saved ID halves, writes only the
CLEAN state and independently salted checksum, then wipes its scratch
and physically rereads and verifies the complete CLEAN record.

Only after the physical readback, MM release and owner/identity/
policy revalidation does the mounted provider clear WR_UNCERT and
INTENT_ARMED, leaving QUIESCING held. VFS checks media, all FILE/cwd,
PIPE and swap/PURE owners and MM again after the potentially sleeping
operation, then releases quiescence and retires without another
scheduler yield. An ambiguous or partial physical CLEAN preserves
the mounted owner plus WR_UNCERT/QUIESCING and cannot be reopened as
a healthy old volume. Pre-I/O MM failures can refuse safely.

## Accepted source/physical evidence

- Raw RBN0 CLEAN: **18 actual source-linked PDP-6 cases**, 11 unsafe
  mutations rejected.
- Mounted owner/identity/policy guard: **37 cases**, 10 mutations.
- Shared pinned-MM CLEAN wrapper: **11 cases**, seven mutations.
- Resident VFS unmount rollback/owner exclusion: **18 cases**, six
  mutations. **Total 84 accepted scenarios and 34 unsafe mutations.**
- Earlier v1552 MM, v1554 physical VERIFY MM, v1555 RO admission and
  physical write veto, v1556 media recovery, v1561 intent, v1562
  quiescence and v1563 PURE/executable owner tests PASS.
- Full production 256K KCC KCORE/optional DTFS MRES image PASS. Fixed
  `__kcore_low_end` moves `024330` to `024364` octal (+28 TEXT words).
  **Zero additional fixed DATA/BSS/mount/process words.**
- Both normal real-guest Type-551 DTVOL2/CLEAN read-only lifecycle and
  DTVOL2/WRITE_INTENT physical mount refusal still PASS, complete
  tape unchanged under SHA-256.
- A source-matched **disposable KCORE/MRES test image** additionally
  armed physical DTC3 DIRTY using the existing v1551/v1552 producer
  and temporarily set RW only for that test's exact VFS slot. The
  otherwise unmodified v1566 production CLEAN path then physically
  restored CLEAN and reread it. The full physical guest completed
  two such sequences, and complete tape SHA-256 was unchanged.
  These test-only hooks do not enter production source or builds.

## Current release gate

The Type-551 controller still needs a verified removable-media
generation/exchange guarantee; raw DATA/DIR and CLEAN-prefix physical
faults, return/reinsertion, replacement media and cold boot after
actual RW need acceptance before changing production V2 RO policy.
This stage proves a CHECKED physical CLEAN path, not writable DTFS.

**P0-A closed; P0-B OPEN; P1-C/D/E unchanged. VERSION 0.5.**
Strict 32K optimization HOLD.

Sources: `DAIMOS/design/kcore-dtfs-clean-seal-v1566.md` and
`daimos-testkit/tests/validation/daimos-dtfs-clean-v1566.md`.
