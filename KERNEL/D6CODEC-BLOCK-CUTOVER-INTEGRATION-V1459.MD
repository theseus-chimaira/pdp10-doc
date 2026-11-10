# v1459 block-only decoder integration: measured checkpoint

Date: 2026-11-10. Status: **integrated in a versioned branch;
not yet accepted for every boot device or memory profile**.

This checkpoint follows the validated 53-word fixed decoder in
`KERNEL/D6CODEC-FIXED-LOW-CORE-V1458.MD`. The v1452 production
baseline already removed the D6LZ36 decoder. The v1459 work
selects the smaller v1458 fixed block decoder and its VFS
adapter in the actual KCORE and Stage-1 builds. The compressed
block v1421 format and host encoder `d6block-v1452` are unchanged.

## Single fixed decoder image

KCORE links the 53-word core as its first TEXT object, beginning
at octal `000060`. All five Stage-1 sources install their own
embedded copy at that same address before decompression.
An independent native word-for-word comparison found exactly
one 53-word match in each Stage-1 read-in image against the
first 53 linked KCORE words. The byte pointer for `LDB` is
part of those 53 words and is therefore also identical.

The kernel uses one 58-word VFS/direct-buffer frontend, which
calls octal `000060` directly. The `EXEC`, PURE backing and
TSFS direct-buffer callers have been updated to the v1458
entry points. No second block core or D6LZ36 decoder is
linked. The VFS frontend has a 39-word temporary stack frame
and no static DATA/BSS.

## Reproducible size comparison

Both KCOREs were built fresh from the corresponding source
branches with the same full-profile KCC configuration:

| Measurement | v1452 control | v1459 integrated | Saving |
|---|---:|---:|---:|
| Fixed core TEXT | 61 | 53 | 8 |
| VFS frontend TEXT | 58 | 58 | 0 |
| KCORE DXR initialized words (octal) | 022627 | 022617 | 010 |
| KCORE DXR initialized words (decimal) | 9,623 | 9,615 | **8** |
| KCORE BSS (octal) | 000415 | 000415 | 0 |
| KCORE relocation entries (octal) | 000414 | 000414 | 0 |

The actual full-profile KCORE map ends at octal `023314`
instead of `023324`. This is a physical eight-word reduction
in initialized image and permanent end address, not merely
an object-file source-line or symbol-count change.

## Cold-boot results

Every successful test used freshly built images and isolated
copies of all attached writable SIMH media. The tests required
the PDP-6 to reach `LOGIN:`, accept ROOT login, start DSH and
execute a guest ECHO command.

| Configuration | Result |
|---|---|
| 256K full profile, DSK Stage-1, disk root | **PASS** |
| 256K full profile, DRM Stage-1, disk root | **PASS** |
| 256K full profile, PTR Stage-1, disk root | **PASS** |
| DTC Stage-1, disk root | Not accepted: timeout before boot banner |
| MTC Stage-1, disk root | Not accepted: timeout before boot banner |
| 48K GCC lowmem, DSK Stage-1, disk root | Reaches INIT V1, no login within 180s |
| 32K GCC lowmem | KINIT image/strict KCORE gates fail |
| 256K full profile, DSK Stage-1, TSFS root | Reaches INIT; `INIT: RUN FAILED` |

The **v1452 control** exhibits the same 48K GCC lowmem
no-login result and the same TSFS-root `INIT: RUN FAILED`.
These are not evidence of a new v1459 decoder regression,
but neither configuration may be counted as accepted.

For the 32K lowmem profile, KCORE ends at octal `023034`,
above the strict octal `020000` gate. The 32K KINIT image
end is octal `103605`, above the stack-safe ceiling
`077400` by 2,181 decimal words. The lowmem 48K build
fits with `BOOT_CORE_WORDS=0140000` and
`SIMH_CPU_KWORDS=48`, but runtime progress is still
insufficient. No memory-limit workaround is applied.

## Remaining acceptance work

Before marking v1459 production-complete, independently
validate DTC/MTC cold boots, TSFS compressed restart extents
and executable restore, and the strict 32K/48K memory
profiles. The test harness and 53-word identity comparison
are in `daimos-testkit/tests/validation/d6codec-v1459-integration/`.
The source checkpoint is on the DAIMOS branch
`d6codec-v1459-integrate-20261110`.
