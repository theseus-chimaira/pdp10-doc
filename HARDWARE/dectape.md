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
