# PDP-6 DECtape: sequential transfer and seek timing

## AAP Type-551/555 emulator measurements (2026-10-25)

The PDP-6 uses the Type-551 DECtape controller, Type-555 transport and
Type-136 data control. The AAP emulator models tape motion as one tape-line
step per scheduled controller event, at 30,000 events per simulated second
(`pdp6/emu/dt.c`, `makedt()` and `dtcycle()`).

An AAP controller-level experiment formatted a tape with the existing
`test_dt.c` harness and performed one continuous forward read. The test
counted Type-136 data words accepted and the number of scheduled DECtape
events. Its results were:

| Quantity | Result |
| --- | ---: |
| Forward-read tape-line events | 931,570 |
| Simulated elapsed time | 31.052333 s |
| 36-bit words transferred | 73,984 |
| 128-word data blocks transferred | 578 |
| Sustained transfer rate | 2,382.56 36-bit words/s |
| Equivalent packed data rate | 10,721.5 bytes/s (approximately 10.72 kB/s) |
| Mean time per 128-word block | 53.72 ms |

The duration here is **simulated tape time**, not emulator host-wall time.
This experiment exercises the emulated controller and data control, not a
DAIMOS guest filesystem read; guest CPU overhead, filesystem dispatch,
interrupt processing and buffering may lower effective application throughput.

## Seek/rewind speed

In the current AAP model, the Type-555 tape advances by the same amount on
each motion event whether the controller is reading useful words or simply
moving the tape to another block. **Seeking, winding forward and rewinding
therefore have the same linear tape speed as streaming reads**; a seek does
not have a separate higher-speed winding mode. Reads additionally incur
block framing and control overhead, so the useful-word rate is lower than
the raw tape-line rate.

The configured tape extent is 988,288 line positions (`DTSIZE` in
`pdp6/emu/dt.c`). At 30,000 line events/s, the calculated full extent takes
988,288 / 30,000 = **32.943 s** in either direction. This is a model-based
calculation, **not an independently timed end-to-end seek test**. Startup,
reversal, controller acquisition and stopping can add delay; the actual
distance between readable end blocks can be smaller than the total extent.

| Tape distance | Approximate AAP motion time |
| --- | ---: |
| 10% | 3.29 s |
| 25% | 8.24 s |
| 50% | 16.47 s |
| 75% | 24.71 s |
| 100% | 32.94 s |

For uniformly distributed independent source and destination positions, the
mean distance is one-third of the tape, or about 10.98 s of motion. From a
fixed end to a uniformly distributed destination, the mean distance is half
the tape, or about 16.47 s. These estimates exclude command overhead and
direction reversal.

## Implications for DAIMOS TSFS

* Avoid unnecessary reversals and long-distance seeks, particularly for
  metadata needed repeatedly during filesystem operations.
* Prefer contiguous sequential reading and arrange related data physically
  close together. Unlike a disk, DECtape seek latency grows directly with
  the distance traversed along the tape.
* At the measured rate, each avoided 36-bit stored word saves approximately
  0.420 ms of sequential tape-transfer time. Compression is beneficial only
  if its CPU/decompression costs and any extra seeks do not erase that gain.
* Type-136 transfers can overlap CPU work, but sustained decompression while
  streaming remains to be verified with guest-side DAIMOS measurements.

The timing figures above characterize the **current AAP implementation**.
They must not be presented as separately measured historical hardware
performance or as measured DAIMOS filesystem throughput.

## AAP work remaining: independent Type-555 transport motion

The Type-551 shares one Type-136 data-transfer path: only one selected
transport can read/write data through that controller at a time. Independent
transport movement should permit other units to position their tapes while
the selected unit transfers data. **AAP does not yet emulate this**:
`pdp6/emu/dt.c` explicitly notes `TODO: currently only selected drive will
move, this is wrong`, and `dtcycle()` calls `dxmove(dt->seldx)` only.

Required work, before treating simultaneous seeking as available in DAIMOS:

1. Verify original Type-551/555 selection, motor-control, start/stop and
   reversal semantics against hardware documentation. In particular, confirm
   how software commands an unselected transport to continue moving.
2. Keep motion state and relevant acceleration/reversal delays separately
   for each `Dx555`, instead of treating global `Dt551` fields `ut_go`,
   `ut_rev`, `ut_time` and `delay` as independent-drive state. The existing
   `Dx555.motion` field can be reused where its semantics permit.
3. Advance every running transport on the periodic tape tick, independently
   of which unit is selected. A deselection must not implicitly freeze a
   transport that hardware would leave running.
4. Preserve a single controller mark decoder, checksum/data buffers and
   Type-136 transfer path. On selection changes reacquire mark-track timing
   without resetting or changing unrelated transports' positions.
5. Extend regression coverage in `daimos-testkit`: concurrent transfer on
   unit 1 with other units seeking; independently stop/reverse a different
   unit; switch to a moving unit and verify block numbers and data; verify
   existing DECtape boot and TSFS paths.

Preliminary scope: a moderate change, tentatively 150-300 implementation and
test lines, **not an established estimate**. The historical command semantics
are the main uncertainty. Do not implement concurrent tape motion by merely
replacing `dxmove(dt->seldx)` with a loop: that would leave global start/stop
and direction behavior incorrect. No emulator change was made as part of
this documentation.

## Paper-tape bootstrap comparison

The PDP-6 Type-760 high-speed paper-tape reader is rated at approximately
**400 frames/characters per second**; the Type-761 punch is approximately
**63.3 characters per second**. Assuming six 6-bit tape frames per 36-bit
word (before format-specific overhead), the reader accepts approximately
66.7 words/s, compared with the measured 2,382.56 words/s DECtape stream.
The frame count depends on the actual paper-tape encoding; do not confuse
host file byte counts with physical tape frames without checking the format.

Build artifacts inspected on 2026-10-25 in the DAIMOS PDP-6 boot output
(artifacts timestamped 2026-10-24):

| Artifact | Measured size | Hypothetical Type-760 reading time |
| --- | ---: | ---: |
| `kcore.rim` | 10,476 `deposit` words | 157.14 s (2 min 37 s) |
| `kinit.rim` | 18,487 `deposit` words | 277.31 s (4 min 37 s) |
| Both stages | 28,963 words | 434.45 s (7 min 14 s) |
| `stage1-dsk.pt` | 4,213 host bytes | 10.53 s **if** one byte equals one tape frame |

The `.rim` artifacts here are **SIMH deposit-command text files**, not
physical RIM-format paper tapes. Their `deposit` line counts measure loaded
words, not encoded tape length. The full-stage times above assume exactly
six frames per word and omit tape leaders, checksums, record framing and
CPU/loader overhead. They are lower-bound reading estimates, not measured
paper-tape boot times. The smaller `stage1-dsk.pt` is the current disk-boot
bootstrap loaded via the paper-tape reader; it subsequently loads the rest
of the operating system from disk, rather than feeding both stages through
the reader. Its byte-based estimate is conditional on frame encoding.
