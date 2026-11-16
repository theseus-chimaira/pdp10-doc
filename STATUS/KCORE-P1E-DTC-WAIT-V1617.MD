# PDP-6 PURE restore DECtape I/O wait, v1617

Date: 2026-11-16.
Accepted: physical instruction/module attribution
and entry to DECtape block search.
Not accepted: successful restoration or BRK return.

The v1616 interrupted PC 030762 is not inside
the optional PURE service. Original physical
KCORE scheduler jump instructions select PURE
MRES base 032027. The original linked tape
module's SKIPN storage_count is at offset
000331; its installed base 030431 places
the instruction at PC 030762.

A new physical 48K PDP-6 simulator breakpoint
at that exact tape address succeeded. It found:

    storage_state  000200
    storage_count  777777772564
    current slot   000000

The count is signed -05214 octal, the original
45-second Type-551 RBN search timeout at 60 Hz.
Its mere initialization does not prove clock
tick advancement, successful mark interrupts,
or a bounded read failure.

The same physical MM table contained four
PACKED owners: slots 5, 4, 3 had 007000-word
unpinned suffixes; slot 2 had a full-size
010000-word claim with one pin. The latter
is legal while a PURE prefix is loaded.
No permanent leak is inferred.

The standalone testkit v1617 audit requires
the original KCORE and PURE maps, original
tape DXR, original tape source, physical
trampoline examination and interrupted/live
SIMH logs. Both a one-word shifted service
export and a mutated tape countdown fail
the executed source-linked assertions.

No production kernel, tape driver, MRES, MM
or process state changed; optional PURE
is OFF in normal builds. The next milestone
must observe clock aging and PI3/tape
progress, followed by prefix commit or
rollback, correct RUN/WAIT and physical
net RAM. No codec or alternate CPU backend
is introduced.

Authoritative DAIMOS record:
design/kcore-p1e-physical-dtc-wait-v1617.md.
