# P1-E v1616: real physical PDP-6 RAW PACKED evidence

2026-11-16. Physical diagnostic completed; no production
acceptance or KCORE/MRES source change. PURE remains OFF
by default.

The no-AUX TSFS guest is a real 48K PDP-6 running a
PURE DXR2 program. The source-matched breakpoint at
022104 proves the BRK entry executes on the actual
CPU. A SIMH console EXPECT stop immediately before
the 020000 BRK captures five resident PROCESS extents,
zero PACKED and no 020000-word aligned free gap.

A separate replay, armed at the source-verified
post-BRK instruction 020761 only after the final
BRK request marker, did not reach that breakpoint
before the controlled timeout. Its asynchronously
interrupted physical MM table nevertheless
contains two PACKED owners:

* Slot 5: 067000..076000, 007000 words, zero pins.
* Slot 4: 076000..106000, 010000 words, one pin.

These are real physical owners. The full-size pinned
PACKED type is an expressly valid intermediate
representation during original v1384 in-place
prefix restoration: the immutable prefix has
been claimed, and the potentially slow VFS reload
has not yet committed or rolled back. This
snapshot alone cannot prove a stranded pin.

The exact MM table source-map auditor in
daimos-testkit diagnostics v1616 PASSes on
both saved physical simulator traces. The latter
still lacks a normal BRK return, AC2 classification,
completed slot-0 restoration and PIPE/WAIT.

Next must observe the restoration commit or
rollback and subsequent exact MM pin release.
No change to production OOM policy, no new
process memory, no speculative codec or new
machine backend is justified by an interrupted
snapshot. Net physical RAM gain remains open.

DAIMOS canonical design:
design/kcore-p1e-physical-packed-v1616.md.
