# KCORE v1576: P1-C physical MEMFS/LOGSTORE coexistence

Date: 2026-11-14. Accepted **P1-C partial storage-isolation slice**;
the larger P1-C acceptance is OPEN. P0-A and operator-trust P0-B
remain CLOSED, VERSION 0.5, strict 32K optimization HOLD.

The current production 128K-word KCC/PDP-6 kernel and MRES booted
from a real simulated DSK-root and mapped AUXSTORE DRM236 physical
backing. The guest mounted persistent MEMFS at `/TEMP`, created and
checked an XIP executable and exact three-word data file, and called
the real privileged LOGCTL STATUS/APPEND/READ_BLOCK service. It
physically appended a uniquely marked LSREC1 record, read back its
physical mapped LOGSTORE slot, verified the 36-bit magic, sequence,
code and payload, and established a single producer-cursor advance.
The ordinary HALT then committed MEMFS snapshot state.

A completely separate cold boot with **new RAM** retained the exact
same DSK and 8 MiB AUXSTORE drum media. The guest restored original
MEMFS file bytes and XIP RUN/WAIT, found exactly one LSREC1 record
from the previous boot by reading LOGSTORE slots, appended a second
uniquely marked LSREC1 and read it back. Both guest test invocations
exited status zero; the AUXSTORE descriptor was unchanged after each
boot and byte-identical after boot 2. Physical regions remained:
BACKSTORE `5770+10000`, LOGSTORE `5370+400`, CACHESTORE
`15770+2010`, D6FS `1+5367` (octal region coordinates).

Separate source-authentic PDP-6 tests also PASS: v1536 KINIT v1/v2
MEMFS snapshot sector reservation before swap with seven rejected
unsafe source mutants (including genuine PDP-6 BACKSTORE allocator),
and v1316 actual MM pin/restore/backing-bitmap swap-in rollback with
three rejected mutants. LOGSTORE native append and direct READ/WRITE
PASS. v1316 mocked its actual backing read, so the tests must NOT be
combined into an unsupported claim of physical concurrent swap.

Testkit `tests/validation/daimos-p1c-logstore-snapshot-v1576.sh/.md`;
DAIMOS `design/kcore-p1c-persistent-mix-v1576.md`. No production
code, MM data, ABI, slab, driver or filesystem layout changed;
`__kcore_low_end=024364` octal unchanged.

**P1-C still OPEN:** one full guest must prove physical process
swap-out/swap-in while MEMFS snapshot commits, plus injection of
in-flight DSK270/DRM236/DTC hardware-versus-provider faults and
LOGSTORE commit failure with rollback of PI, pin and BCACHE owners.
