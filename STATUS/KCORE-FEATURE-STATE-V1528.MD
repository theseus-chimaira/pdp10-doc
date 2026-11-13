# KCORE features v1528: physical DTFS failed-write quarantine

2026-11-13. Feature completion, not optimization.
Actual PDP-6 256K DTC-root guest with the
unchanged v1527 fixed KCORE and installed
DTFS/TSFS optional MRES.

The new real physical test mounted native
stamped DTC3 **read/write**, created a FILE,
wrote a word, and completed metadata writes.
At a synchronized host checkpoint, SIMH
physically detached **only** DTC3 while
TSFS root DTC0-2 remained attached.
The target attempted another metadata commit
through normal KCORE/DTFS and received an
error. The host then reattached the
**original, identically stamped 72-bit-ID
volume**.

The original filesystem remained quarantined:
FSINFO/STAT failed, two FILE CLOSE attempts
refused without silently releasing the
descriptor, and ordinary UNMOUNT refused.
Real PID1 reported `DTFSWRITE V1528
WRITE_FAILED`, `WRITE_STICKY`, `PASS`;
host runner exited zero. An independent
diagnostic repeat with Type-551 CONO logging
also PASSed and captured DTC3 write commands.

This establishes a real failed physical write
and the conservative `WRITE_UNCERTAIN`
fail-closed state on same-media return.
It does **not** prove an interrupt partway
through writing a 128-word block, and does
**not** implement or validate administrative
recovery. The test detached the tape before
the unsuccessful commit. Source-linked native
v1522 validation separately tests the exact
mounted poison bit, while the physical test
checks its visible lifetime and owner behavior.

The correct administrative contract must
handle the FILE/cwd owners explicitly, prove
full media repair or replacement independently
of a matching old identity, and only retire
unowned mounted state once checked. Merely
ignoring failed SYNC, forcibly reusing a mount
slot, or clearing the uncertain bit after a
matching-ID read is unsafe.

Detailed physical test and design evidence:
`DAIMOS/design/kcore-dtfs-write-uncertain-physical-v1528.md`
and `daimos-testkit/tests/validation/
daimos-dtfs-write-uncertain-v1528.md`.

Production code and on-media format unchanged
since v1527: optional MRES octal `003747`
text, `000002` BSS; **zero KCORE fixed
word or persistent RAM growth**.
VERSION 0.5; strict 32K gate HOLD.

Remaining features: P0-B mid-write physical
partial commit and safe checked administration;
P0-A authoritative critical root/BACKSTORE
permanent-loss HALT; P1-C mixed storage
faults; P1-D dynamic FSINFO capacity/inventory;
P1-E swapless PURE/MM/APR; final feature freeze.
