# PDP-6 P1-E v1647: V2 unmount and PURE executable admission

Status: physical source-linked testkit acceptance, 2026-11-17.
DAIMOS remains VERSION 0.5; this test does not enable PURE
swapless operation by default.

## Contract

When a non-root DTFS DTVOL2 filesystem enters final-unmount
quiescence, `vfs_dtfs_quiesce_begin_v1562` acquires the media
owner-admission barrier without yielding after its owner scan.
Even if provider I/O then sleeps, `proc_swap_attach` must not
publish a new RUN/EXEC PURE dependency on that mount. A failed
RUN cannot alter existing process swap records or MM owners.
Once unmount finishes, a fresh mount of the same exact medium
may admit a new executable again.

The V2 barrier is **not** part of the legacy DTVOL1 path.
The stock native DTC3 HOME reel was DTVOL1; tests which
observed only a retired mount (FSINFO 0), rather than an
in-progress quiescence refusal (FSINFO -1), were invalid.
The v1647 testkit builds a real DTVOL2/CLEAN reel with
original `mkdtfs --durable-v1550` and verifies the physical
record checksum, 72-bit volume identity, CLEAN bit, seal and
unused-word zeroing. Seven corrupted records are rejected.

## Real hardware/scheduler acceptance

The original PDP-6 KCORE uses 48K, an independent TSFS root
on DTC0..2 and a writable secondary DTC3 mounted at public
mount2. The C89 user fixture uses a compressed PURE DXR2
and two initial unpinned RESIDENT child images each
`012000` octal words. These processes are deliberately
not PACKED because the admission race does not require RAM
pressure; PACKED reclaim is tested independently in v1639/
v1645.

The disposable test build copies all original VFS assembly
and inserts just one real `proc_sleep_ticks(01400)` call
after acquiring V2 quiescence. No other production
owner/EXEC/VM/PI6/MM operation is changed. The source audit
requires this exact one-line behavioral insertion.

An independent TSFS-backed helper starts UNMOUNT while
the parent observes FSINFO -1, attempts the real compressed
PURE RUN (also -1) and observes FSINFO -1 again. Original
process swap records and physical MM extents are identical
before and after refusal. Selected TSFS root and RW DTFS
media ownership remain correct. The genuine CLK PI6 counter
advances **1004 ticks** between race start and successful
unmount completion. The executing slot at completion is
helper slot2; other snapshots sample the parent slot1.

Following successful unmount the parent remounts exactly
the same DTC3 medium and its compressed PURE RUN succeeds.
All original PIPE/EXIT-057/PID/WAIT and final unmount checks
complete, with no child MM owner remaining. The real SIMH
replay has one normal HALT PC000000 and no user failure.
Independent verifier rejects forged clock progress, swap
record changes and a false read-only secondary mount.

## Limits and next gates

This v1647 acceptance proves no concurrent **new** PURE
executable dependency can appear once a V2 barrier is held.
It does **not** prove a simultaneous *pinned FULL restore*
against provider I/O, uncertain MM unpin/free rollback,
general 32K/64K RAM layouts, transient physical peak RAM
or KA10 VM backend. Production kernel and fixed RAM are
unchanged. The initial two children were RESIDENT, so do
not count this result as new PACKED reclaim savings.

DAIMOS design and evidence:
`design/kcore-p1e-dtfs-v2-quiescence-physical-v1647.md`.
Testkit:
`tests/diagnostics/pure-quiesce-run-v1646/`, with physical
`check-v2-v1647.py`, negative media verifier
`test-v2-v1647.py`, source check and MM/PI6 mutation audit.
