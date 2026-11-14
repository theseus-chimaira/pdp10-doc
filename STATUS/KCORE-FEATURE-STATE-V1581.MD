# KCORE v1581: source-selected real Type-236 snapshot PI2 fail-stop

Date: 2026-11-14. The full 128K KCC/PDP-6 guest physically exercised
the original direct DEVICE BACKSTORE error path while actual process
swap was owned, without modifying production code. P0-A and the
operator-trust P0-B contract remain closed; P1-C remains OPEN.

On an 8 MiB Type-236 physical AUXSTORE drum with DSK270 D6FS root,
the test created the original persistent MEMFS data and raw XIP,
physically appended/read a LOGSTORE event, started four independent
24K-word sleeping process children and entered root HALT. A disposable
snapshot guard verified real KCORE `proc_swap_blocks_used > 0`.

A test-only source selector tagged the first write of the invalid
snapshot commit header at the last BACKSTORE sector. The Type-236
driver bound the actual 3-word stack I/O descriptor, waited for its
true DR_DONE PI2 completion, and routed only that descriptor to the
original PI2 error publication. The real original BACKSTORE gateway
dispatched the v1515 reason-2 HALT, linked at octal `004011`, rather
than falsely returning success or silently releasing the owner.

After HALT, actual fixed KCORE diagnostic counters and physical drum
bytes agreed: selected error count 1, original DRM driver error
counter `mfsdev_storage_errors+5` also 1, active backing, `01374` octal
allocated swap blocks, last relative backing sector `07777`, Type-236
physical group address `0157670`, completed descriptor and cleared
trigger. Raw Type-236 physical sector `015767` had no valid committed
snapshot header. 248 lower backing sectors were nonzero; physical
LOGSTORE/BACKSTORE separation and AUXSTORE descriptor were intact.
Seven negative evidence mutations were rejected by the strict physical
verifier. The artificially selected completion error is not proof of
naturally defective magnetic media. Only the diagnostic source copy
gained five extra KCORE witness words, not the production kernel;
production fixed low end remains `024364` octal.

P1-C is still OPEN for independent process swap-in PI completion
*during* synchronous MEMFS snapshot I/O, mapped multi-member VOLUME
failures, controller return, and BCACHE/MM/owner rollback under
recoverable error conditions. P1-D alternate physical inventories,
P1-E uncompressed PURE, and final feature freeze remain OPEN.
VERSION 0.5, strict 32K/optimization HOLD.

Evidence: DAIMOS `design/kcore-p1c-snapshot-pi2-failure-v1581.md`,
testkit `tests/validation/daimos-p1c-snapshot-pi2-physical-v1581.md`.
