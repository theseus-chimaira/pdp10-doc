# P1-E slot-0 RAW PURE boundary, v1610

Date: 2026-11-16. This status records accepted source-linked
PDP-6 *mode selection*, not accepted RAW PURE reclamation.

The removable SWAP MRES now checks the per-process two-bit
storage map before disk swap-in. A DISK process retains
the original disk path. PACKED retains its exact inactive
MM owner and saved executable record, and the service
defers the pending wake request for its existing 0100-
octal cooldown, with no disk I/O, exchange, TERM or
READY_ONCE. An invalid mode also refuses. A PACKED
process cannot yet be successfully restored from the
production MRES; there is no producer in production.

The shared staged PURE preflight now separates DROP,
TERMINAL and RESTORE. Only RESTORE may accept a RUN
process when slot 0 owns the selected TRANSITION-held
target; DROP still requires SLEEP or STOP. The existing
v1384 fast prefix and bounded v1391 FULL restore
source-linked native tests exercise RUN wake and
non-slot-0 refusal. They also verify actual MM owner,
mode and mutable-suffix preservation.

The production MRES mode test and unsafe bypass mutant,
existing v1316/v1317 disk swap and rollback, v1608 source
admission, v1609 terminal PACKED cleanup and the v1605
PDP-6 machine source selector PASS. Full KCC ROOT=tape
KCORE and SWAP MRES package builds and links. Permanent
KCORE DXR and map match v1609 byte-for-byte; low end
remains `024534` octal. The mode gate adds eleven
removable MRES instructions, no permanent DATA/BSS or
per-process words.

Still OPEN: actual no-BACKSTORE PURE MRES, immutable
executable lifetime/failed source handling, scheduler
RAW DROP, fully restored RUN/READY_ONCE, all PI6/EXEC/
WAIT/BRK/EXIT/finalizer/physical guest/peak-RAM tests.
Only after those should mutable RAM compression be
evaluated. Neither KA10 nor KI10 support is implemented.

See DAIMOS `design/kcore-p1e-slot0-mode-v1610.md`,
`TODO.MD`, `PROJECT-STATE.MD`, and
`STATUS/KCORE-REMAINING-WORK-V1607.MD`.
