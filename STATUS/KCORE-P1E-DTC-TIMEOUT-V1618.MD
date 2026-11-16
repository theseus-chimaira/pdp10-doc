# v1618: full-minute DECtape rewind timeout headroom

Date: 2026-11-16.
Feature: experimental P1-E, default OFF.

Type-551 DECtape transports may require up to
one minute to rewind or reverse direction. The
former 45-second search deadline was too short.
The new per-search timeout is 120 seconds,
7200 decimal ticks or 016040 octal ticks at
60 Hz. A search that finds its target block
completes without waiting for the deadline;
continued PI3 block marks rearm the timer.

The Type-516 MTC full-reel rewind/spacing
timeout remains 15 minutes. Existing 5-second
DTC and 90-second MTC transfer watchdogs
are unchanged.

The PDP-6 linked tape MRES changes only two
instruction operands, at offsets 000343 and
000405. The module image, BSS and relocation
counts are byte-for-byte layout-compatible;
there is no fixed KCORE or process RAM growth.

In a rebuilt real 48K PDP-6 no-AUX TSFS
guest, the Type-551 wait shows three
consecutive line-clock ticks/mark samples:

    Clock  015435 -> 015441 -> 015446
    Mark   0341   -> 0133   -> 0126
    Timer  777777761740 (rearmed)

These octal readings verify the exact 120s
deadline and live PI6/PI3 progress. The
source-compiled binary tests reject the
historical 45s variant, and the physical
timer audit rejects an old-count mutation.
Old v1617 evidence is always checked
against its matching 45s DXR.

Physical RBN/data transfer completion,
actual PURE restoration and source lifetime,
slot-0 resume/PIPE/WAIT and positive net
physical RAM acceptance remain unproven.
Do not enable P1-E by default or change
the tape driver further solely because
the search remains active in these samples.

Canonical DAIMOS record:
design/kcore-p1e-dtc-timeout-v1618.md.
