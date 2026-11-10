## 2026-11-10 v1405: BACKSTORE refusal is prepublication, retryable

The v1398 checked `backstore_free(first,blocks)` returns 0 on complete
bitmap/used-accounting release, -1 on refused ownership. Production
`proc_swap_in` now invokes the checked release before committing the
new physical resident base and before decrementing process swap blocks.
If the release fails, its newly allocated MM extent is freed by the
existing after-unpin rollback, the TRANSITION lock is removed, and the
stable disk record, PURE/executable backing, bitmap and storage mode
remain unchanged. If the MM rollback itself fails, the existing fatal
invariant preserves the lock instead of resuming an untracked process.

Native testkit v1405 source-links the exact transaction, BACKSTORE
verifier and free code; it tests a deliberately incomplete allocation
bitmap, retention of all disk-owner accounting and storage identity,
then a successful retry with the missing bit restored. A source mutant
that discards the release return is rejected by the executable assertions.
No duplicate allocator implementation or new persistent owner field.

# DAIMOS v1318: native source-authentic swap-in transaction and rollback

Date: 2026-11-07. Baseline: DAIMOS `4b2369d1` with unchanged
production kernel code; testkit `7ffd840e`. Scope is the PDP-6
native integration test, not a booted system acceptance.

## What is real production code

The v1316/v1317 fixture compiles the authoritative PDP-6 assembly
directly from the current DAIMOS source, without substituting its
swap-in transaction:

* `proc_swap_in` and `proc_swap_mres_service_v481`;
* `mm_find_fit`, `mm_alloc_aligned_noreclaim`, bounded
  `mm_alloc_aligned`, `mm_compact`, `mm_pin`, `mm_unpin`, `mm_free`;
* `proc_storage_expect_v30`, `proc_storage_publish_v30`, the packed
  resident/DISK storage-mode representation;
* `backstore_free`, including the actual backing bitmap;
* `proc_sched_pi_tick` timekeeping prefix and the production
  `proc_select_rt_normal_swap_v879` scheduler gate.

The simulated 010000-octal-word managed physical arena is initially
fully occupied by a real kernel-dynamic MM extent. The pending
swapped process has a properly initialized u-area, DISK storage
mode, extent size, packed backing descriptor and eight allocated
0200-word sectors. The first actual swap-in allocation returns
MM_ERR_NOMEM; the test's filesystem and process reclaim providers
cannot remove the fixed owner, and three exchange candidates are
intentionally rejected. The production slot-0 service preserves
the original image, enters its 0100-octal cooldown and does not
terminate the process.

After 63 calls to the production PI6 timekeeping prefix the
cooldown is still active and the scheduler selects the resident
path. The original physical owner is released through real `mm_free`.
On tick 64 the actual scheduler gate requests another swap-in.
The production transaction then allocates an aligned MM process
extent, pins it, reads into its physical destination, unpins,
publishes the resident VM base and mode, releases backing blocks
with the production bitmap allocator, and grants READY_ONCE.
The test checks the correct physical extent/owner, both endpoint
payload words, zero pins, bitmap cleared and both swap accounting
counts zero. It executes with authentic PDP-6 36-bit instructions.

## Read failure and rollback

The test disk provider separately returns an I/O failure. The
unchanged `proc_swap_in_read_fail` path releases the pin and the
newly allocated process extent with production MM code. The
original disk record, u-area backing offset, DISK storage mode,
eight-sector bitmap allocation and swap counts remain intact.
TRANSITION clears only after the memory rollback succeeds; the
service invokes exactly one fatal TERM event.

Three executable mutations are rejected by PDP-6 runs:

* skipping the rollback `mm_free` (detectable orphan extent);
* skipping `backstore_free` (bitmap/usage remains allocated);
* skipping resident storage-mode publication (DISK mode persists).

The successful/failed-read scenarios and all three mutations PASS
their respective acceptance or expected-rejection checks. Earlier
v1315 real-MM/PI6-scheduler, v1301 no-partner and v756 typed-result
regressions continue to PASS on the same production source.

## Limits and next step

The backing read provider is **test-only**: it writes recognizable
words into the real allocated physical image, but does not perform
an actual DSK/D6FS read. Exchange candidate admission, independent
runnable-path progress, and the scheduling half of the hardware PI6
interrupt are likewise test-controlled. Therefore this is a native
production-transaction verification, **not** proof of a booted
256K PDP-6 full-system no-partner forward-progress case.

The remaining gate must trigger genuine MM allocation pressure
inside a booted guest, verify the no-partner branch, independent
process execution through the clock delay, recovery using real
backstore I/O, exact ownership and final rollback/cleanup, without
injecting a fabricated allocator result. The historical v1208/v1211
16-word fallback is still excluded. No additional production kernel
code or permanent RAM was introduced by this work.

## Evidence

Native extracted assembly, source maps, successful and failed-read
SIMH reports, three executable negative mutation reports, tool/source
hashes and a verified SHA256 manifest are retained on the Termux host:

`$TMPDIR/daimos-swap-in-real-transaction-v1318-evidence-r1.tar.gz`

SHA256:
`4459b6c916c52d7cc71c5062ff727bb76989edb4a2b6e93cb056f962dc47d8eb`.
