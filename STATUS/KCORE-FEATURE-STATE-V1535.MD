# KCORE v1535: physical hot-swap CTY synchronization accepted

Date: 2026-11-13. Strict KCORE feature correctness only.
The change affects physical testkit console delivery, not
production KCORE, loaded MRES, actual media format or RAM.

## Verified cause of intermittent v1527 regression stall

An additional v1527 physically replaced DTC3 guest run
timed out after `READY_SWAP`, without a negative
provider result. The old test sent `CONT` and `A\r`
consecutively over the simulator monitor's stdin.
Monitor command parsing and running CTY keyboard polling
are distinct input consumers. A v1535 guest witness
after the actual CTY READCHAR isolated the missing
acknowledgment. An early SIMH `SEND` trial could
still lose this signal: a follow-up monitor break
showed the KCORE idle loop and `SHOW SEND` reported
no pending data while the guest was waiting.

The physical simulator source `sims/PDP10/kx10_cty.c`
sets a 10000-instruction transmit interval per CTY
output character. When the host detects `READY_*`
the guest may still be transmitting its CR/LF, before
entering READCHAR. Queueing an acknowledgment after
only 10000 simulated instructions is insufficient.

The v1535 runner queues
`SEND AFTER=200000,DELAY=20000,"A\r"`
through SIMH *before* issuing `CONT`. It checks
the monitor accepted SEND. The first character
is injected after twenty CTY output periods,
and CR after two more periods. The guest reads
the simulated console keyboard, not monitor stdin.
No host sleep, extra retry, fake controller reply,
weakened guest assertion or production hook.

## All physical regressions PASS

On the same real 256K KCC PDP-6 full guest build:

* v1527 original/different/returned DTC3 and
  independent DTC4, full physical 72-bit identity,
  held FILE and both normal unmounts. PASS on two
  separate delayed-SEND runs; additional witness
  `SWAP_ACK_RECEIVED_V1535` reached.
* v1528 physically detached DTC3 negative WRITE,
  returned matching ID still WRITE_UNCERTAIN,
  FILE/mount ownership preserved. PASS.
* v1531 root-only checked administrative discard
  and replacement with another stamped DTC3,
  same poisoned original refused. PASS.
* v1534 mixed writable DTFS DTC3/DTC4, writable
  MEMFS and TSFS root: independent data survives
  physical DTC3 metadata-write failure. PASS.
* v1529 opt-in Type-551 diagnostic partial WRITE:
  after detach and flush, actual RBN0145 contains
  exactly one new 36-bit data word and one unchanged
  old requested word; reattachment remains poisoned.
  Physical media verifier and PID1 both PASS.

The simulator is the accepted versioned Type-551
`pdp6-v1529r5` for the intentionally interrupted
physical write; other cases run the normal
`pdp6-v1524`. The prebuilt booted KCORE and
process-private tapes/command files remain unchanged.

The v1527 stall was a test transport bug, not an
accepted KCORE data-integrity regression. P0-A
authoritative critical root/BACKSTORE loss,
P0-B durable abandoned-media admission, P1-C
mixed DSK/DRM/logging and swap, P1-D additional
inventory profiles, and P1-E uncompressed swapless
PURE/MM/APR remain open. VERSION 0.5, 32K and
optimization HOLD.

See `DAIMOS/design/kcore-physical-console-ack-v1535.md`
and `daimos-testkit/tests/validation/
daimos-physical-console-send-v1535.md`.
