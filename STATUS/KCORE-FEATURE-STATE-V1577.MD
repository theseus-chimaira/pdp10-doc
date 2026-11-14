# KCORE v1577: LOGSTORE failure/rollback with persistent MEMFS

Date: 2026-11-14. One accepted P1-C storage-fault increment;
overall P1-C OPEN. P0-A and trusted-operator DTVOL2 P0-B CLOSED.
VERSION 0.5, strict 32K/optimization HOLD.

## Executed failure path

Native PDP-6 source-linked acceptance invoked the real production
LOGSTORE MRES append. An injected backing writer physically modeled
a partially committed record with an invalid last-word trailer and
returned a negative result. The original append left ring slot,
producer sequence and notification unchanged. Its real retry wrote
the complete record to the *same* slot, advanced exactly once, woke
the producer-event consumers once, and delivered matching data
through the real MRES READ_BLOCK entry. Four executed unsafe source
mutants failed: ignore the error, skip sequence advancement, skip
slot advancement and skip wakeup.

Full 128K KCC/PDP-6 booted twice with the actual DSK270 root and
DRM236 AUXSTORE physical image. In both boots a disposable, copied
LOGSTORE MRES intercepted only a uniquely marked test event; it
performed a genuine physical Type-236 DMA with the event's commit
trailer zero, then reported a **simulated completion error** to the
unchanged append path. The guest's privileged LOGCTL READ_BLOCK
confirmed the whole invalid record was on the actual drum. LOGCTL
STATUS retained the original sequence and ring slot. A normal retry
then committed the record physically at the same slot and updated
the cursor once. The test also created and persisted MEMFS data and
XIP files, used ordinary HALT, and booted the same backing media
with new RAM. Boot 2 recovered the first event and the original
MEMFS files, then repeated the fault and same-slot retry. Both
boots exited status zero; AUXSTORE descriptor geometry remained
unchanged after each boot and byte-identical after boot 2.

The actual controller did **not** report a naturally occurring
failure, and the injected fault writes a full invalid sector rather
than claiming Type-236 supports sub-sector partial writes. Nor does
this test reboot with an unretried orphan-invalid log record. These
limits matter when evaluating future P1-C hardware acceptance.

No production kernel, optional MRES, physical driver, RAM, ABI,
filesystem or on-media format was modified. All new fault code and
its two counters exist exclusively in the testkit's disposable
MRES. Fixed `__kcore_low_end` remains `024364` octal. P1-C still
requires truly simultaneous swap and MEMFS snapshot commit, natural
LOGSTORE/device errors, and in-flight DSK270/DRM236/DTC physical
versus mapped VOLUME ownership/PI rollback.

DAIMOS design `design/kcore-p1c-logstore-fault-v1577.md`; testkit
`tests/validation/daimos-p1c-logstore-torn-v1577.md`.
