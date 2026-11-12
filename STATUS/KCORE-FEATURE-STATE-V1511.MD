# DAIMOS KCORE feature state v1511: Type-236 read-error propagation

Date: 2026-11-12. This is an incremental source-linked **physical
acceptance** over v1510. It is not a general device-offline or
damaged-media recovery feature. Production KCORE remains v1507,
without additional process state, resident RAM or ABI changes.

## Booted 256K PDP-6 Type-167/236 error-path acceptance

The full disposable 256K KCC PDP-6 F07 v1511 testkit profile
selects a genuine backed DISK process (octal slot `010`) after
an actual successful Type-236 transfer. Its original BACKSTORE
allocation (`02350`/`0330` octal) remains correctly marked in
the bitmap. On the next two genuine reads of that same owner,
the actual live Type-236 DMA descriptor's `DR_DONE` completion
is routed through production `drm236_pi_error` and
`drm236_finish_active_v1079`, not fabricated in the swap service.
The original DRM error counter, event wakeup, negative BACKSTORE
read result and `proc_swap_in_read_fail` MM unpin/rollback are
then exercised. All subsequent reads retain their ordinary
checked-free refusal; healthy other-process I/O is unaffected.

The real guest generated 36 PI6-clock-progressing source-linked
samples, including 13 fault-on samples (24..36). The physical
v1511 strict verifier PASSed on final logs and exact owner
identity: `002350000330`, stable u-area and nonresident DISK
VM, original bitmap and used accounting, retry epoch and pending
TERM throughout. At the end there were **2** actual DRM error
completions and **25** refused checked releases, exactly **27**
negative source-linked slot-0 service returns. The independent
real DRM counter `mfsdev_storage_errors+5` registered both
errors. No premature READY_ONCE/RESIDENT, repair, fatal close,
finalization, ZOMB or backing release occurred.

Over the 18,228 real PI6 ticks, resident PID 5 executed 823
additional genuine SLEEP syscalls (58..881). Independent
PID `011` remained DISK-backed at fault onset, restored by a
checked real swap-in and executed **397** genuine user SLEEP
syscalls by sample 36. The v1511 source tests rejected five
invalid gateway/driver mutations, and the strict evidence
verifier rejected eleven false-owner/rollback/progress variants.
Existing source-linked v1079 DRM 300-tick watchdog, v1080
queue and v1247 PI-register tests also PASS.

Full per-sample table, guest location and SHA256 hashes:
`DAIMOS/design/kcore-backstore-device-error-physical-v1511.md`.
The accepted testkit source and result revisions include
`d46872d`, `37e6f8c` and `25df8fa`. An older v1511 attempt
targeting Type-270 DSK rather than Type-236 DRM was correctly
rejected as test setup error; it is not an accepted result.

The diagnostic deliberately stopped after the observation
window and **did not HALT**: its unrepaired owner cannot consume
private pending TERM. This proves physical Type-236 *controller
error-path* propagation and retained ownership, not actual
magnetic-media loss, missing PI2 completion, permanent hardware
offline, media reconstruction or eventual reclaim. A synthetic
PI2 error is injected at a real completion, not at a fabricated
swap return. No new quarantine process state is needed.

## Remaining ordered KCORE gates

1. Testkit v1512: lost Type-236 `DR_DONE` completion, genuine
   PI6 300-tick watchdog timeout, negative BACKSTORE return,
   rollback and independent healthy swaps (native/prep PASS;
   full physical gate OPEN).
2. DEVICE/media disappearance or unreadable extent: provider
   recovery/reconstruction and safe long-lived owned failure,
   never forced backing release or phantom parent WAIT/zombie.
3. Mixed DEVICE/VOLUME, DTFS/BCACHE, LOGSTORE/MEMFS provider
   errors, fail-closed routing and recovery.
4. F1 immutable swapless PURE/MM/APR reclamation and restore.
5. F3 FSINFO dynamic 1/4/8/26 mount enumeration; separate
   uncommitted v1492 work remains untouched.
6. Optional compression only after measured net RAM savings;
   final KCORE size optimization once features are complete.

Earlier v1506 repaired owning TERM and v1507 scheduler retry
semantics remain accepted. VERSION 0.5, 32K HOLD; MRES
unload/reload is POST-1.0, unified TSFS/D6FS packer separate.
