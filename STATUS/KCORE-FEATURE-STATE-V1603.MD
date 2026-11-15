# KCORE P1-C v1603: final storage/swap/BCACHE physical feature acceptance

Date: 2026-11-15. DAIMOS VERSION 0.5, original PDP-6 KCC-built
KCORE, unchanged production MM/BCACHE/BACKSTORE/LOGSTORE/filesystem
code and critical owner-failure policy. **P1-C COMPLETE** within
the explicit physical and native source-linked matrix below.

## Final v1603 same-guest physical intersection

Two genuine separate 128K PDP-6 cold boots of a four-member Type-236
DR0..DR3 D6FS root, interleaved mapped BACKSTORE VOLUME and actual
LOGSTORE/MEMFS persistent snapshot. Both boots independently prove:

| Original source-linked physical observation | Boot 1 | Boot 2 |
|---|---:|---:|
| Independently original PI6-woken DISK owners | 6 and 7 | 6 and 7 |
| Authentic wakeup count for each owner | 1 / 1 | 1 / 1 |
| Original mapped swap READ attempts | 5 | 5 |
| Actual Type-236 DR_DONE PI2 completions | 1120 | 1120 |
| Checked original post-DMA MM/DISK/bitmap rollbacks | 1 | 1 |
| Physical LOGSTORE torn uncommitted WRITE observed | 1 | 1 |
| Original committed same-slot retry after selected error | 1 | 1 |
| Changed original clean D6FS cache words (failed / retry) | 0 / 0 | 0 / 0 |
| Authentic clean-cache fetches after both phases | HIT / HIT | HIT / HIT |
| Original MM clean private cache base (octal) | `377577` | `377577` |

The D6FS cache contains 28 nonzero physical data words and all
128 values are unchanged through both LOGSTORE writes. The persistent
MEMFS snapshot is committed on boot 1 and restored on boot 2;
the original header is on DR3:`017777` octal, pool 3648 blocks
with 283 other nonzero physical backing sectors. Original backing
ownership remains 936 process blocks, the MM rollback is 20->19
allocations, original bitmap differences zero and no false RESIDENT
publication for the refused owner. Both guests HALT normally
at `022011` octal. A complete source-linked original post-HALT
KCORE and raw-media oracle rejects **44** false observations:
15 new cache/LOGSTORE witnesses, 16 dual PI6 owner/rollback witnesses
and 13 original mapped physical PI2/media witnesses.

Physical testkit reference:
`tests/validation/daimos-p1c-two-owner-cache-logfault-physical-v1603.md`.
Exact accepted raw evidence:
`$TMPDIR/daimos-p1c-composite-v1603.Z4dCmb`,
`$TMPDIR/daimos-p1c-v1603-fullguest-run1.log`.

## Clean BCACHE and failed authoritative WRITE

There is **no dirty BCACHE**: the production cache has two clean
slots (the shared workspace plus MM-managed private slot), not a
writeback queue or dirty-page flusher. The original D6FS writer
invalidates the current workspace cache identity before sending
its authoritative physical WRITE to the selected DSK270 or DRM236
adapter. It creates the next clean cache identity **only on a
successful write**; a failed write returns via the original
secondary-mount vs selected-root critical ownership classifier.

New v1602 actual native PDP-6 source-linked test includes original
D6FS reader commit, entire original BCACHE implementation,
FS_BACKING direct DSK/DRM adapter and original error classifier.
It passes failed and successful DSK/DRM operations (four cases),
verifies complete preserved independent 128-word private cache and
authentic fetch, and rejects five unsafe source mutations. Only
the final controller completion is test-controlled, so this is NOT
misrepresented as another full physical-media detach.

## Additional already-accepted physical owner matrix

* v1599: real Type-236 mapped root VOLUME, separate slots 6/7 each
  actually PI6-woken, exactly one selected physical swap READ slot6
  fails with real PI2 and original **reason-2 BACKSTORE HALT
  `004011` octal**; backing owner retained, no false RESIDENT state.
* v1600: independent DSK270 root and direct DRM0 DEVICE BACKSTORE
  (original `400000005770` mapping, physical DR0:`006421`), two
  original PI6 wakeups and selected fatal slot6 READ/PI2,
  original reason-2 HALT `004011`. Neither fatal case has post-HALT
  recovery, multiple overlapping DMA READs or further root work.
* v1596: physically detached high Type-551 DTC6 mounted DTFS ID5,
  other mounts remain identifiable while independent writable MEMFS
  actually writes and reads data; same physical tape returns after
  authentic identity revalidation. True Type-551 simulator error
  corrected at source in `sims` v1598; no kernel workaround.
* v1593/v1588: independent real timer-woken owner slot7 RESIDENT
  publication and exact checked slot6 MM/bitmap/cache/record
  rollback with retained original ownership.
* Fresh v1592: two full physical PDP-6 boots with independent
  PI6 slots 6/7 and 5 real mapped READs/1120 PI2 per boot,
  one original rollback; contemporaneous other host build/load.
* Fresh v1590: two full physical PDP-6 boots with a live populated
  D6FS cache through physical LOGSTORE torn/commit retry and
  later normal PI6 swap. Native v1544 DSK/DRM root/nonroot
  source-linked 24 cases + six mutants, native v1490 authentic
  DTFS security gate/cache and v1538 root HALT/nonroot return PASS.

The legacy v1518 DTFS cache test still uses a pre-v1561 source
mutation anchor and stops at fixture generation with current code.
It is NOT counted as passing and the mandatory authenticated
DTFS write-intent/security gate has not been circumvented.

## Closure status

**P1-C CLOSED** for the tested physical mapped VOLUME/direct DEVICE
swap owner failure classes; original independent PI6 and
MM/BCACHE/cache ownership; noncritical mounted provider physical
failure with unrelated healthy writes; authentic write-through
error and clean-cache identity; recoverable LOGSTORE physical
torn-write retry with populated cache and two simultaneous live
swap-backed processes; repeated loaded physical guest runs.

**P1-D** is already CLOSED by v1601 dynamic FSINFO/root-controller
acceptance. **P0-B** Type-551 physical reel-retention/interlock is
a different still-OPEN correctness gate. **P1-E KA10 PURE/MM/APR**
review and 32K RAM-footprint optimization remain on HOLD. VERSION
0.5, fixed KCORE low end `024364` octal. No permanent source,
RAM, syscall ABI, module or media-format change in v1602/v1603.

DAIMOS full detailed matrix:
`design/kcore-p1c-final-physical-acceptance-v1603.md`.
