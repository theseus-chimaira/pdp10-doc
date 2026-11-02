# DAIMOS GCC disk storage integration — focused acceptance V53

## Scope and toolchain

On 2026-10-20 the previously failing **023 / storage** group of the
30-group, non-size functional matrix was rerun as one serial GCC testkit
invocation.  The image has 256K memory with diagnostic
`DSH_IMAGE_LIMIT=037000`; this is not a 32K or permanent-size acceptance.
The compiler is the isolated GCC backend `df27247`, not the unmodified
systemwide cross-compiler.  The existing DAIMOS production kernel sources
were not changed by this repair.

## Result: PASS, ten of ten tests

The complete test command reported exit 0:

- `test-daimos-monitorfs-devices-online`
- `test-daimos-d6fs-context-dispatch`
- `test-daimos-d6fs-multimount`
- `test-daimos-d6fs-direct-backing-drm`
- `test-daimos-bcache-multisource`
- `test-daimos-d6fs-remount`
- `test-daimos-d6fs-super-recovery`
- `test-daimos-vfs-storage-reservation`
- `test-daimos-badmap-map`
- `test-daimos-badmap-root`

The last test recovered a D6FS boot volume whose superblocks had been
destroyed, using BADMAP.  Previous failures in this group originated in
test oracles that omitted `vfs_mount_limit` and a MonitorFS probe that
left DECtape unit 3 attached while disabling the controller; these are
repaired in testkit commits `2804838` and `81b2610`.

Evidence: `$TMPDIR/daimos-gcc-v53/group023-v53.log` and
`$TMPDIR/daimos-gcc-v53/result-v53.tsv` (`group023 0`).

See also `STATUS/DAIMOS-GCC-FUNCTIONAL-REGRESSION-REPAIRS-20261019-V52.MD`.
Other failing matrix groups remain unresolved; this confirms group 023
only and does not assert full functional-suite acceptance.
