# DAIMOS AUTO-root: four-member drum fixture repaired (v91)

## Scope

The non-size GCC acceptance run v74 tested three physical root classes on
the 256K PDP-6 SIMH configuration. `ROOTCLASS DISK` and `ROOTCLASS TAPE`
passed, but drum-only AUTO failed before LOGIN with KINIT HALT `070761`.
The strict 32K and dedicated size gates are deferred and were not run.

## Root-cause confirmation

At the KINIT root mount-capacity checkpoint the return value was zero;
both `kinit_boot_handoff` words remained `0777777777777`, and
`d6fs_active_reader` was zero. The lower-level `drm236_read_block` call
returned success. SIMH memory inspection showed the root marker in block
zero and the packed member descriptor `000074040000`: D6FS membership
mask `017` (four members), logical index zero. The generated scanner read
mask 017 and index zero correctly. The rejection was therefore not evidence
of a GCC byte-extraction or Type-236 driver defect.

The testkit built a complete four-member drum root, but only copied
`build-drum/drum/dsk0.dsk` into its AUTO simulator media directory. Drum
units 1–3 retained unrelated volumes from the AUTO build, making the root
incomplete. Correct D6FS root selection rejected the member set.

Testkit commit `68d5664` replaces the single copy with copies of all four
physical drum members. No DAIMOS kernel sources changed for this repair.

## Focused regression result

Fresh copies of all four drum-root images were attached to the original
AUTO KINIT image under rebuilt PDP-6 SIMH. The boot showed `D6FS LOADED`
and `INIT V1`, reached ROOT login and DSH, and the probe:

`CAT /CONFIG/SYSTEM`

returned `ROOTCLASS DRUM`. The v91 harness reported **1 PASS, 0 FAIL**.

Evidence:

- `$TMPDIR/daimos-root-four-member-v91/report-v91.md`
- `$TMPDIR/daimos-root-four-member-v91/run-v91/dcs0.log`
- `$TMPDIR/daimos-nonsize-acceptance-v74/006.log`

The DISK and TAPE paths passed in the v74 full test; DRUM passed in the
separate v91 focused retest. An atomic fresh-image rerun of the corrected
three-class test remains a final acceptance task. The other non-size
regression failures and 32K memory constraints are separate work items;
**32K remains deliberately deferred**.
