# DAIMOS KCORE feature state, v1505

Date: 2026-11-11. Incremental update to the authoritative
`STATUS/KCORE-FEATURE-STATE-V1503.MD`. This update concerns KCORE
failure correctness, not optimization, 32K, standalone MRES or
general userland work. The historical v1503 status remains intact.

## Newly accepted native boundary

`daimos-testkit` commit `b0e5075` v1505 runs real production
`proc_swap_in`, `proc_swap_mres_service_v481`,
`backstore_verify_allocated_v795` and `backstore_free` under PDP-6.
With one missing backing-allocation bit, six *explicitly requested*
swap-ins all fail closed: no false READY_ONCE or RESIDENT publish,
no changed DISK descriptor or bitmap, no lost pin/temporary MM owner,
no backing count decrease and no invariant violation. Each attempt
invokes an observed TERM request. When the fixture repairs that
single bit, a seventh request succeeds, releases exact backing and
publishes the VM once. Prior v1493 two executable negative checks
remain passing. MM and I/O backends and event observer are controlled
native fixtures; this is not another physical guest result.

The successful scheduler return includes `PROC_SCHED_READY_ONCE` in
AC1, e.g. octal `01002` for slot 2. The production function comment
now states that exact return. On refusal, AC1=-1 signals that TERM
was requested, not proof of synchronous destruction: v1503 physical
evidence showed two requests coalescing with an existing pending TERM
and retaining the original DISK owner. The source comment was also
corrected for this distinction; no runtime code/RAM change.

## Remaining, ordered KCORE feature gates

1. **F2a physical owner completion**: v1504 256K KCC guest is
   collecting target-specific real owning-return pending TERM
   consumption, private fatal close and selected VM/finalizer state.
   No result should be claimed before completed HALT and checks.
2. **F2b permanently corrupt backing**: prove recovery policy if a
   bitmap/DEVICE cannot be repaired, including bounded scheduler
   work while preserving DISK ownership and the private executive
   continuation. Six forced native attempts do NOT show production
   retry cadence or terminal progress.
3. **F2c end-to-end physical fault gate**: actual bad bitmap ->
   checked refusal -> deferred or immediate TERM -> exact owner
   finalization and account-zero release, preserving the separate
   normal v1336 no-terminal-swapin acceptance.
4. **Other F2 device/media**: remaining mixed DEVICE/VOLUME,
   BCACHE/DTFS tape-root, LOGSTORE/MEMFS and recovery integration.
5. **F1 swapless PURE**: complete scheduler/VM/MM rollback and exact
   APR reconstruction without the +180 TEXT-word pilot.
6. **F3 dynamic mounts**: v1492 source changes uncommitted; physical
   1/4/8/26 mount contract and consumers not qualified.
7. **F4 optional compression**: only if PURE accepted and same-profile
   resident RAM measurement is positive; thereafter remaining
   KINIT/clock/device physical gates and final KCORE optimization.

32K remains HOLD, MRES runtime unload/reload POST-1.0, unified TSFS
and online D6FS packer separate from this KCORE feature phase.
