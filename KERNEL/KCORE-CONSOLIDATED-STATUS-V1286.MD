**2026-11-09 v1375 policy supersession:** The former fourth pre-sweep
track, opt-in compressed process **disk swap**, is CANCELLED. Raw disk
swap is retained. Its replacement is swapless-only PURE executable-text
dropping and opt-in in-RAM compression of mutable process memory. This
requires a new inactive process transaction, full APR reconstitution
and end-to-end acceptance. No production implementation is claimed.
See `KERNEL/SWAPLESS-MEMORY.MD`. The v1286 compressed-disk-swap
requirement below is historical, not current work. This does not affect
the separate, already-closed OOM test group named P0-D.

**2026-11-07 v1298 status update:** Requester-only BRK physical-OOM
F12 production activation and F13 post-enable signoff are now
accepted for the strict 256K KCC DSK/D6FS release profile, closing
64/64 mandatory OOM gates. See
`KERNEL/OOM-F13-PRODUCTION-ACCEPTANCE-V1298.MD` for full source,
guest, independent Android/Linux media and SHA256 evidence. The
v1286 source audit below remains a historical default-OFF snapshot;
its F12/F13 OPEN assertions are superseded, not current blockers.
The four pre-optimization functional feature tracks remain OPEN.

# KCORE consolidated source/status audit v1286

Date: 2026-11-07. DAIMOS checkout `e6993a76` on `main`; documentation
checkout was `f2829f0` before this report. This is a source-to-document
reconciliation, **not** a new guest acceptance or a fresh linked-size run.
The historical dated sections of `PROJECT-STATE.MD`, `KCORE.MD`, and
`TODO.MD` remain evidence of their original revisions, not current gates.

## Current authority and scope

* Source code establishes whether a feature is wired in production.
* `DAIMOS/PROJECT-STATE.MD` v1284 and `DAIMOS/TODO.MD` v1284 are the
  latest release acceptance record and actionable checklist respectively.
* `KERNEL/OOM-F11-RELEASE-ACCEPTANCE-V1284.MD` records frozen evidence;
  do not reinterpret an interrupted Linux suite as a continuous PASS.
* `KERNEL/KCORE-FEATURE-STATUS-V1269.MD` is a historical snapshot. Its
  F07/E13/F11-open assertions were superseded by v1274/v1276/v1284.
* Current `VERSION` is `0.5`; the planned feature-freeze milestone is 0.9.

## Direct source checks

| Area | Source checked | Current implementation conclusion |
| --- | --- | --- |
| Requester-only BRK OOM | `system/kernel/proc/syscall_dispatch_pdp6.s`, `native_sys_brk` | `PROC_OOM_BRK_ENABLED_V1000` is **0**. When enabled it discriminates actual BRK `-1` and AC2 cause `-1`; not yet production ON. |
| Writable EXEC/RUN | `system/kernel/proc/exec_lowmem_pdp6.s`, `system/kernel/proc/proc_run.s` | Both still call `vm_space_startup`; raw MEMFS XIP is not connected as a normal production process admission path. |
| MEMFS XIP primitives | `system/kernel/fs/memfs_runtime.s`, `system/kernel/proc/vm_pdp6.s` | MRES op `027` admission and `026` release exist; owned-XIP unpin errors are not misclassified as ordinary process-MM releases. Complete scheduled non-writing process lifetime is not demonstrated by these primitives. |
| Compression cutover | `system/kernel/proc/exec_load.s`, `system/kernel/fs/tsfs_runtime.s`, `system/kernel/core/D6CODEC-INTEGRATION-TODO-v054.md` | Active consumers still reference `d6lz36_decode_vfs` / `d6lz36_decode_buffer`. Grouped D6CODEC v053/v055 remains staged, not a production format cutover. |
| Process swap | `system/kernel/proc/proc_swap_pdp6.s` | Raw swap service, allocation-pressure handling, and fatal service failure paths exist. The older proposed compressed **disk**-swap records are cancelled by v1375. |
| Packed-process experiments | `system/kernel/proc/proc_packed_lifetime_v33.h`, `proc_packed_mm_registry_v35.h` | Prototype registry explicitly not enabled at boot; do not count as shipped process compression. |
| Mount policy | `system/kernel/fs/vfs_mount_access_v5.s` | Slot accessor validates at most octal `032` (26) slots; this is **capacity/interface**, not evidence of 26 live mounts. |
| Software clock | `system/kernel/Makefile`, `system/kernel/drivers/wall_clock.c`, `system/kernel/proc/syscall.h` | `WALL_CLOCK_HZ` selectable 50/60, SETTIME extension `037`; physical 50-Hz device qualification is separate. |

## Accepted capabilities and outstanding activation

EXEC/RUN/EXIT/WAIT, PI6 scheduling, APR VM/BRK, typed MM errors,
session/TTY/HUP, FILE/VFS, backed raw swap, D6FS disk root and boot-time
MRES dispatch are implemented and have previously recorded native/full
guest acceptance. This is not a statement that all media/platform modes
have passed every grouped regression.

The v1284 acceptance record closes **62 of 64 OOM gates** for the frozen
256K KCC PDP-6, DSK Stage 1, four-member D6FS root, 60-Hz PCLK,
headless CTY, writable RUN/EXEC profile. F07 64-slot maximum pressure,
E13 two-host release media, and F11 exact-source integration are closed.
**F12** (production switch and comprehensive both-policy regression) and
**F13** (post-enable signoff, revert on any failure) remain unchecked.
Raw XIP admission and 32K DSH are outside this first OOM profile.

## Open functional feature-freeze tracks (required before size sweep)

1. **D6 codec:** atomic replacement of D6LZ36 in all boot/EXEC/VFS
   consumers; streaming decoder, format producer, rollback, native tests,
   and resident decoder/workspace/stack measurement.
2. **Shared BSTORE/AUXSTORE:** finish remaining production backing,
   BCACHE/LOGSTORE consumer migration, mixed DEVICE/VOLUME/REGION and
   BLOCKSET integration, fault/recovery ownership, and RAM accounting.
   Existing AUXST1 primitives already implemented must not be rewritten.
3. **Root-only MEMFS raw XIP:** no-user-write/trusted-image validation,
   restricted RUN/EXEC, nonwriting startup, swap/BRK exclusion, robust
   pin/unpin and rollback, scheduled PDP-6 lifecycle and security tests.
4. **Superseded (v1375):** This snapshot's opt-in compressed-disk-swap
   requirement has been retired. Current fourth track: swapless-only
   PURE prefix omission and explicitly admitted mutable-RAM compression,
   with MM ownership, source immutability, safe restore and native tests.
   The staged RLE codec v1376 passes host/PDP-6 component tests but is
   not linked into production or evidence of process-lifecycle acceptance.

## Other open correctness, integration, and documentation tasks

* Preserve and diagnose actual failed boot media (`?RT`) before cleanup;
  verify the particular root and INIT failure rather than assuming cause.
* Finish safe swap pressure/admission liveness and remaining provider
  recovery, with explicit limits on nonreturning owner operations. The
  v750-v754 allocator/backing deadlock is historical evidence; newer
  pressure paths require **fresh regression against the current source**
  before declaring the exact case fixed or still reproducible.
* Complete grouped GCC/KCC/KINIT and relevant storage/media/provider
  fault tests, safe MKSWAP provisioning, online D6FS packer, physical
  50-Hz qualification and full allowed-mount/provider metadata wiring.
* DTFS tests are **not current acceptance blockers**: DTFS/TSFS are
  designated for rewrite. Existing TSFS boot compatibility remains a
  relevant KCORE interface contract; TSFS v1.2 feature implementation
  is explicitly post-KCORE and before 0.9.

## Memory: do not mix unrelated profiles

The v1174 KCC 256K installed inventory measured 9944 KCORE words, 272
fixed MM descriptor words and 8837 installed MRES words (19053 words
plus origin 060), or 2717 words beyond permanent address `040000`.
This is the **last documented inventory**, not a fresh measurement of
`e6993a76`. The old v735 GCC size and v513 GCC figures belong to other
revisions/configurations and cannot serve as current deltas. The strict
KCORE below-octal-`020000` target and full 16K permanent-residency
target remain separate from the first 256K requester-OOM release.
MRES are boot-installed and not runtime unloadable/reclaimable.

After the four **revised** functional tracks and relevant regressions, remeasure
the exact current installed-MRES and linked endpoints, dynamic physical
high-water and APR slack, then apply only source-backed RAM-saving
changes. Cold-path JRST cross-domain folding must exceed approximately
20 **net permanent words saved**, with no KCORE growth/hot-path penalty.

## Explicitly deferred or outside current KCORE freeze

TSFS v1.2 sharded metadata is after KCORE completion/optimization but
before 0.9. Runtime MRES unload, general overlays, hardware paging,
Type-340 light-pen/input, foreign TS 1.4 DECtape and full 32K DSH are
separate later scopes. Do not turn a historical unchecked TODO into an
active gate without reviewing the latest accepted scope.

## Next actions

1. Complete F12/F13 on frozen exact production sources and record
   positive/negative acceptance without widening the release profile.
2. Reproduce or close the remaining swap-pressure and provider-lifetime
   issues against current source; distinguish old experiments from bugs.
3. Complete D6 codec, shared storage, restricted XIP and swapless-only
   PURE/RAM reclamation, while holding RAM and execution-time constraints.
4. Complete the grouped non-DTFS regressions, freeze KCORE interfaces,
   then perform and verify the final KCORE/MRES footprint reduction.

No production code was changed or executable/guest test claimed by this
documentation-only audit.
