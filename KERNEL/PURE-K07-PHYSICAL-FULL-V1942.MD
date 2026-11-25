# PDP-6 K07 physical pressure and FULL acceptance, v1942

Scoped physical PASS, 2026-11-25. Unchanged v1939 production kernel,
GCC lowmem 48K TSFS image. One plain and four compressed PURE
children pass actual RUN and sleeping-state checks. A sixth RUN
refuses and the parent survives. The historical six-child success
fixture is not appropriate for the current 48K image; the adapted
fixture explicitly requires the observed refusal.

At the B/A no-growth BRK checkpoints, MM18 is identical, parent
slot1 has its unpinned 010000-word resident APR at 130000..140000,
and five children have unpinned 07000-word PACKED extents:
121000..130000, 111000..120000, 101000..110000, 071000..100000,
061000..070000 (all octal). Four 01000-word alignment gaps separate
the children; slot2 is adjacent to the parent. Aggregate reclaimed
prefix storage is 05000 words (2560 decimal). This is observed
child VM ownership, not a new optimization or peak/net system RAM
measurement. No outstanding executable reservation.

Independent current service bindings establish PURE base 033136.
Linked and installed FULL entry opcode 200000000004 at PC040756
matches the actual native breakpoint. At this point scheduler slot0
owns the restore continuation. Slot3 has its original exact pinned
PACKED source 111000..120000 and a distinct pinned PROCESS target
120000..130000, each pin count one. Stable u-area 051274, destination
capacity010000. The original mutable source remains authoritative
during loader execution. Source and destination do not overlap.

All five original child data sentinels and exact PID/EXITED/status
WAIT checks succeed. At parent Z, MM8, no child VM/u-area and no
outstanding reservation. PCLK advances 10389 decimal ticks from
B to Z. Exactly one normal HALT PC000000, simulator exit0.
This verifies the ordinary physical pressure/restore/exit lifecycle,
not a concurrent source-loss fault or forced MM rollback.

Testkit commits 863489f and 389de60. Actual monitor controller and
verifier pass; seven synthetic geometry negatives and nine altered
accepted-transcript controls reject. These are verifier negatives,
not nine executed guest faults. One exploratory boot interrupted
to use explicit nobreak all; both explicit and bare commands pass
isolated monitor smoke. No production source or RAM growth.

Accepted trace SHA256:
e31bf43619c8bb91b42694739a2ca89930a964c042ee0a21dea11870240d2e0d
KCORE SHA256:
4177db78eabee2b54a1cb3d6e2cec3eee6e4b0b2d9e330a22f3d6acf052c85e1
PURE SHA256:
e488f1a93315ff7039326c948d1e9b39632507850c999fffa1285f975c6b0c41

Testkit: tests/validation/pure-k07-full-v1941/README-v1942.md.
K07 remains open for genuine PI6 concurrent executable reservation/
retirement and integrated source loss/recovery. RAW PURE and mode3
remain disabled by default.
