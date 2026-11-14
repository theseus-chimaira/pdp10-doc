# KCORE v1557: unique physical DTFS unit ownership whenever V2 is mounted

Date: 2026-11-14. Production DTFS MRES source and native acceptance;
**P0-B OPEN, writable DTVOL2 remains disabled**.

Prior DTVOL2/CLEAN read-only mount and read-fault recovery were
physically checked, but the ordinary `dtfs_dtc_write` veneer
receives only a physical Type-551 unit and not the VFS mount slot.
The old V1/V1 multi-mount behavior would make a future per-mount
V2 DIRTY intent gate ambiguous if a new V2 or legacy V1 mounted
the same physical unit more than once.

v1557 introduces native `dtfs_v2_mount_exclusive_v1557.s` in the
real optional DTFS MRES. After the production physical native
directory/RBN0 validation and before VFS mount publication,
the routine checks the published DTFS root and existing MM media
slabs under the already-existing serialized provider lease.

- A new V2/CLEAN mount is refused if ANY other published DTFS
  mount, V1 or V2, references the same physical Type-551 unit.
- A new V1/zero legacy mount is refused if an existing V2 mount
  references its physical unit. Existing V1/V1 behavior stays.
- Unrelated DTC units, non-DTFS roots and retired stale media
  words do not block a new valid mount.
- Missing mount roots/MM slab pointers or invalid physical unit
  fail closed; no owners or media are changed by this guard.

Native testkit acceptance executes the **actual PDP-6 helper**:
**26 cases PASS, eight unsafe mutations rejected**, including
both conflict directions, later VFS slots, retired stale tags,
foreign providers, invalid units, null tables and V1/V1
compatibility. The expanded source-linked `dtfs_mount_unit`
test reads genuine independently formatted host `mkdtfs` V2
and V1 physical DECtape image bytes and now passes **24 native
cases, 11 unsafe mutations rejected**. Original v1519 direct
mount and v1520 volume-ID/target-MKFS tests remain PASS.
Complete KCC full-profile DTFS MRES assembly/fold/link PASS.

No new fixed KCORE word, syscall ABI, MM allocation, additional
physical I/O or default native-media format is introduced.
No DTVOL2 data/directory writes are yet allowed: the v1555
central write veneer still refuses them. The next mandatory
features are a provider-retained MM scratch owner, integration
of v1551-v1554 physical WRITE_INTENT before all DATA/DIR writes,
exclusive CLEAN/unmount owner quiescence during sleeping I/O,
real Type-551 partial-write, replacement/reinsertion and cold
boot tests, and a proved removable-media generation/fail-stop
contract for the read-then-write exchange boundary.

P0-A complete; **P0-B OPEN**; P1-C separate. VERSION 0.5;
strict 32K and optimization HOLD.

Source: `DAIMOS/design/kcore-dtfs-v2-mount-owner-v1557.md`.
Tests: `daimos-testkit/tests/validation/
daimos-dtfs-v2-unit-ownership-v1557.md`.
