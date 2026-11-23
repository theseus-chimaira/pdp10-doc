# DAIMOS KCORE feature state, v1508

Date: 2026-11-12. Incremental authoritative update to
`STATUS/KCORE-FEATURE-STATE-V1507.MD`: physical long-lived
checked BACKSTORE release refusal bounded by the existing
v1507 per-owner timer. Feature completion only, not optimization.

## New physically accepted boundary

Testkit v1508 builds a disposable full 256K KCC PDP-6 F07 guest
from the already accepted v1504 physical owner/TERM fixture.
After an authentic payload read it selects the first eligible
nonresident DISK owner; only that original BACKSTORE span's checked
release permanently returns refusal. Unlike v1494's reparable
corrupt-bit experiment, the real allocation bitmap stays intact.
All scheduler, PI6, MM pin/unpin, record rollback and deferred
TERM logic is production source.

Real selected slot: `010` octal, first block `02350`, count
`0330`. Live samples 19/20/21 record physical failed release
counts **1/4/7** and actual PI6 epochs **36599/38542/40484**
(decimal). Increments of 3 refusals over 1943 and 1942 ticks
are bounded by the v1507 0100-octal-tick retry policy.

Independent source-mapped snapshots preserved record
`002350000330`, RUN but nonresident VM, original stable u-area,
retry marker and pending TERM `000004004010`. Never falsely
published READY_ONCE or RESIDENT; zero repairs, private TERM
completion, fatal close, finish or zombie. Exact process and
BACKSTORE used accounting still includes that owner. Strict
physical verifier PASS; ten unsafe mutations rejected by the
current v1508 testkit verifier self-test. The historical source
report also records an extended 13-mutation validation, but that
is not the set exercised by the current v1508 self-test.
Genuine F07 emitted 18 RUN acknowledgments and one reap marker.
Sampling deliberately ended before HALT; no normal F07
EXIT/HALT acceptance is claimed for this unrecovered owner.

Source and five SHA256 evidence hashes:
`DAIMOS/design/kcore-backstore-persistent-physical-v1508.md`.
Testkit technical report:
`tests/validation/daimos-backstore-persistent-physical-v1508.md`.

**Scope limit:** this accepts one permanently refusing checked
release, not actual DEVICE electronics loss or its recovery.
The post-fault physical process table showed no other runnable
user: independently runnable bystander and healthy swapped
progress **after the failure** are not physically proved.
The owner remains allocated; terminal recovery/reclaim is OPEN.
No new KCORE production source, process state, ABI or RAM
allocation in v1508.

## Remaining KCORE features as of v1508 (historical)

These are the outstanding conditions *at v1508*, not a current
release backlog. Subsequent v1509 physically proved resident
bystander progress; v1575 closed P0-B, v1603 closed P1-C, v1601
closed P1-D, and v1803 closed K01-K04 within their documented
boundaries. Consult DAIMOS `TODO.MD` for current open gates.

1. Physical persistent-failure case with a distinct independently
   runnable healthy process or swapped contender; prove its
   progress while the failed DISK owner remains held.
2. Qualify true permanent DEVICE/media errors, provider recovery,
   and safe long-lived backing ownership without unsafe forced
   free or false parent WAIT completion.
3. Complete mixed DEVICE/VOLUME, DTFS tape-root, BCACHE,
   LOGSTORE/MEMFS and provider failure/recovery integration gates.
4. Complete F1 uncompressed swapless PURE text with exact
   immutable executable backing, mutable suffix, MM and APR
   reconstruction, rollback and owner lifetime.
5. Finish F3 dynamic FSINFO 1/4/8/26 mount enumeration and
   compatibility/physical acceptance (v1492 locally in progress).
6. Only after PURE and feature acceptance, assess optional
   compression and final KCORE size optimizations.

Previous v1506 repaired-fault owning TERM and v1507 native
retry/clock/exchange tests remain accepted. VERSION 0.5,
32K HOLD. Runtime MRES unload/reload remains POST-1.0,
unified TSFS and D6FS packer separate scope.
