# DAIMOS GROUP 030: EXEC REAL-TIME OWNER ROLLBACK (V191)

Date: 2026-10-23

## Defect and root cause

The resident PDP-6 `exec_load_process` used one kernel stack word for
two simultaneously live objects: `struct vfs_stat` was passed to
`vfs_stat` at `-011(17)` and occupies seven consecutive words through
`-003(17)`; `-003(17)` was also the marker denoting an RT_REQUIRED
reservation newly acquired by this executable load. The final stat
field, `mtime`, overwrote that marker. On a failed EXEC, an existing
RT owner could be released erroneously even though no new ownership
had been acquired by the loader.

This was a real runtime defect, not an assembler-space optimization.
DAIMOS v191 fixes it by reusing `-012(17)` for the marker after moving
the prior vnode scratch value into AC12. Failures before this move
return without consulting the marker. The VFS stat scratch region is
now disjoint from the RT transaction state. No new kernel stack word,
KCORE word, BSS word, or ABI surface was introduced.

## Native verification

`daimos-testkit` v191's `test-daimos-exec-rt-owner` boots one fresh
PDP-6 image containing a normal INIT, an RT_REQUIRED DXR and a normal
replacement DXR. It creates a two-word test executable with a valid
DXR1 RT_REQUIRED header but an incomplete body; executable admission
occurs before its file shape is rejected. The regression verifies:

1. Failed RT_REQUIRED EXEC releases a newly claimed reservation.
2. Failed RT_REQUIRED EXEC retains an already held reservation.
3. Successful RT_REQUIRED EXEC enters its image owning the reservation.
4. Successful non-RT replacement releases the previous RT reservation.
5. RT can be acquired and released again afterward.

The old source consistently failed the second assertion, after the
first assertion passed. With the v191 kernel correction all stages
PASS, emitting `<abRN` followed by HALT. Evidence:
`$TMPDIR/daimos-exec-rt-owner-v191-fixed-native.log`.

The KCC/p10fold KCORE exclusive endpoint remains `025657` octal,
full KCORE 11135 words, and KCORE+MRES 18670 words. The strict 16K
permanent-memory gate still fails. Native testing uses the explicit
functional-only `DAIMOS_EXEC_OBSERVE_SECONDS=35` and
`DSH_IMAGE_LIMIT=037000`; the 15-second and `036000` production gates
have not been relaxed. Further failure injection after VM creation,
the native 32K path and complete group-030 acceptance remain open.

## Follow-up negative-launch regression (V192)

After DAIMOS commit `e2248c0` corrected the marker lifetime,
the existing native 28-case malformed EXEC/RUN regression and normal
current-image replacement both PASSed on a fresh PDP-6 simulator image.
Evidence: `$TMPDIR/daimos-exec-v192-malformed-retest.log`.
This confirms the fix does not regress the established invalid-record
checks. The functional-only 35-second and `037000` settings were used;
strict production limits remain separate and open.
