# P1-E v1649: source-linked FULL mount and backing ownership

Date: 2026-11-17. Native PDP-6 acceptance; no production
KCORE/MM/PURE/VFS modification and no permanent RAM increase.

The original v1391 bounded FULL restore keeps a PINNED
PACKED_PROCESS RAW suffix and separately allocates and pins a
complete PROCESS APR while restoring immutable executable text.
Before the suffix is released, `PROC_TRANSITION` and the saved
executable backing remain authoritative.

The native v1649 testkit directly executes the original
`proc_swap_mount_busy` and `proc_swap_backing_busy` assembler
scanner leaves inside a v1391 FULL callback at precisely this
two-pin state. It uses a genuine source-linked provider-5 DTFS
executable identity and checks the original MM owner/type/size,
pins, storage mode, backing and VM base.

Both guards return busy on the owning mount/executable, without
altering either pinned MM descriptor. The same guards allow
unrelated mount/file identities. The unchanged FULL loader then
completes and publishes a single unpinned RESIDENT PROCESS APR
with correct executable text and original writable suffix.

Two separately assembled scanner mutants execute and are
rejected at octal exit `002002`: inverted PURE text guard,
and inverted exact backing-file comparison. Original v1563
owner/staging tests also PASS 26 native cases and reject six
independent mutants.

The v1523 VFS UNMOUNT path checks the original mount-owner
scanner before the v1562 DTFS V2 quiescence bit is acquired,
without a scheduler yield between those operations. The
existing PACKED dependency during FULL should consequently
prevent ordinary unmount from reaching the provider teardown.
This guard proof does not substitute for an independent user
process attempting UNMOUNT during real blocked provider I/O
under PI6. That physical interleaving and clock/MM audit
remain OPEN.

DAIMOS:
`design/kcore-p1e-full-guard-during-pin-v1649.md`.
Testkit:
`tests/validation/pure-full-mount-guard-v1649/`,
commit `5b51d68`, branch
`test-kcore-p1e-full-mount-guard-v1649-20261117`.

Other open P1-E boundaries: permanent uncertain MM ownership,
whole 32K/48K/64K transient RAM peak and distinct KA10 VM.
PURE OFF by default; strict 32K optimization HOLD.
