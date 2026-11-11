# DAIMOS KCORE 0.9 release-scope audit v1488

Date: 2026-11-11. Scope: source-backed reconciliation of DAIMOS
KCORE features against dated TODO, project-state, build, testkit and
historical pdp10-doc status documents. This is durable documentation,
not a transient implementation handoff. The DAIMOS source and its
current PROJECT-STATE.MD / TODO.MD supersede any stale dated assertion.
DAIMOS VERSION remains 0.5; release 0.9 is NOT claimed.

## v1489 KCORE-only source sweep correction (2026-11-11)

The detailed KCORE-only audit is in DAIMOS
`design/kcore-feature-source-sweep-v1489.md`. It separates installed
KCORE capabilities from staged PURE/codec objects, and excludes MRES
module development, online D6FS packing, unified TSFS, general
userland work and size optimization from the **KCORE feature** checklist.
It also records the outstanding public mount-enumeration interface:
KINIT actually publishes dynamic `vfs_mount_limit`, while the legacy
`VFS_NMOUNT=4` still limits MOUNT/DF to four scanned slots. Do not
replace the live boot mount slab with the old four-slot implementation.
The unused `kpack_codec_v1` identity-only source pair has been removed;
testkit-referenced codec/PURE revisions remain for reproduction.
An isolated current KCC `system/kernel kcore` compile passes after this
source cleanup. This is not new full-system KCORE link or guest evidence.

## Accepted functional boundaries

| Feature | Current acceptance | Evidence / implementation |
| --- | --- | --- |
| Grouped D6 codec | Production cutover v1464 accepted | Production `core/d6block_vfs_fixed_v1458.s` exports `d6codec_decode_vfs_v1458`; EXEC, PURE backing and TSFS call grouped v1458 decoder; host block image producers are v1452 |
| MEMFS raw XIP RUN/EXEC | Accepted under strict root-only, nonwriting policy | v1469-v1485, physical KCC 128K/192K/256K cold boots, owner pins, failure rollback, unmount/remount, committed snapshots and corrupt-snapshot recovery |
| Physical low-memory boot | 48K non-XIP PASS (v1486) | DSK/D6FS ROOT/DSH, five separate DATE/MEMSTAT probes; 49152 total, 16271 resident, 23320 process words, four occupied of 12 slots, MEMFS capacity zero |
| Requester-only physical BRK OOM | 64/64 PASS in the defined 256K profile | v1298 F12/F13 release acceptance; no generalization to untested configurations |
| Persistent MEMFS snapshot recovery | Accepted for stated profiles | v1484 descriptor/LOGSTORE ABI correction; v1485 corrupted metadata refusal and same-boot volatile mount recovery |

MEMFS is not enabled below 96K; 32K and 48K never qualify for MEMFS
XIP tests. All 32K testing is explicitly ON HOLD until requested.
Do not silently reinstate old 32K boot/XIP/PURE release gates.

## The v1487 shared EXEC experiment (not a kernel feature)

The original v1388 common DXR loader candidate predates the v1469
privileged MEMFS XIP entry. v1487 regenerates an isolated candidate
that carries over the original XIP admission/pin/rollback path and
routes DXR1/DXR2 through the shared parser. Native PDP-6 DXR
success/error regression passes. The original XIP code is preserved
by source-generation checks, NOT proven in this native DXR-only test
as a fresh scheduled XIP guest.

Same-revision isolated PDP-6 TEXT size:

| Part | Words |
| --- | ---: |
| Current production EXEC | 218 |
| Candidate EXEC caller | 163 |
| Shared DXR parser/loader/adapter | 235 |
| Candidate net growth before other fold/link effects | +180 |

No corresponding gain in whole-system RAM is demonstrated. Do not
merge this candidate into production solely to reuse the parser.
Native PURE drop v1381, in-place prefix v1384 and rollback v1395/
v1397 pass their isolated PDP-6 tests, but these C transactions
and v1391 bounded FULL restore remain unlinked to the production
scheduler. Trusted RAM bitmap v1451 is a 30-word test-only codec.

## Remaining 0.9/KCORE development

1. Close genuine remaining combined BSTORE/BCACHE/LOGSTORE physical
   provider, storage fault and recovery cases, not previously
   accepted v1484/v1485 results.
2. Integrate swapless PURE-text omission, inactive-process ownership
   and complete APR restoration with scheduler and rollback.
   Only then consider opt-in compressed mutable RAM; prove actual
   net RAM savings including resident code, temporary buffers,
   reserved extent geometry and restoration headroom.
3. Complete the separate 0.9 online D6FS packer requirement.
   Unified TSFS is scheduled AFTER KCORE final optimization and
   BEFORE release 0.9; no historical format sharding mandate.
4. Run final matched-image KCORE and installed-MRES footprint
   optimization LAST, after the remaining functional freeze.

Raw disk swap remains unchanged; compressed disk-swap records were
cancelled v1375. The old v1029 grouped-codec staging report, v1269
feature audit, v1286 source audit, and earlier v512/v1395 TODO
checklists are **historical snapshots**, NOT present release gates.
The old assertions that D6LZ36 is still active, that XIP RUN/EXEC
is missing, or that OOM F12/F13 remains disabled are superseded.

## Future / explicitly not part of current release work

Runtime code unloading and reloading of MRES providers, including
MEMFS, is retained as a **POST-1.0 feature**. It is neither a
0.9 nor a 1.0 acceptance gate; do not investigate, implement or
test it before 0.9 ships. Boot-installed MRES remains permanent;
successful filesystem unmount does not imply module code unload.

The historical 32K minimum-machine design target is not deleted,
but all current 32K tests and associated shrink work are suspended.

## Measurement discipline

Never equate isolated TEXT, fixed KCORE exclusive-end address,
packaged optional MRES words, boot-time transient KINIT placement,
and live process/physical RAM accounting. A valid shrink comparison
must use the same compiler, same root/profile, same loaded provider
set and full guest/native regressions. v1487's +180 is an isolated
candidate cost, not a new KCORE endpoint. v1486's 48K live metrics
are not a 128K/256K resident-MRES inventory.
