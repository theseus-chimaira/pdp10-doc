# KCORE v1530: physical test media and SIMH script isolation

Date: 2026-11-13. KCORE feature-test infrastructure only.
The accepted physical interrupted-write evidence is
recorded separately in STATUS/KCORE-FEATURE-STATE-V1529.MD.

## Correctness problem

The old full PDP-6 physical guest launcher rewrote the
prebuilt KCC build's DTC0-DTC3 tape media and shared
boot.headless.ini on every run. Two concurrent jobs could
replace the same attached DTC3 image, producing a false
preimage-verification failure. It was not a demonstrated
kernel error. Retries and longer physical timeouts do not
restore the required single-owner media invariant.

## Implemented v1530 repair

The testkit's daimos-dtfs-tape-root-mount-v578.sh now
creates process-private DTC0-DTC3 writable tape images,
including disposable native DTVOL1-stamped DTC3 and
TSFS root DTC0-DTC2. Optional DTC4 and replacement
volumes are also private. All were previously attached
writable from a common reused build directory.

The generated boot-headless-v1530.ini is also private to
the test run. It preserves the production KCC boot file
except for exactly four native DTC attachment paths,
which point to the process-private tapes. Every actual
DETACH/ATTACH and the v1529 128-word before/after
physical-media verifier uses the same unique DTC3 image.
The prebuilt KCC image is now immutable for these tests.

## Physical regressions verified

With the corrected fixture, all four real 256K PDP-6
KCORE/PID1 guest suites PASS without weakened assertions:

* v1529 partial Type-551 RBN0145 write: exactly one new
  data word and one original data word coexist on the
  flushed physical DTC3 image. Negative write return,
  sticky WRITE_UNCERTAIN, FILE and mount retained.
* v1528 physical failed WRITE and matching-ID return:
  poisoned FSINFO/STAT/CLOSE/UNMOUNT stay refused.
* v1527 replaced-volume ID refusal, independent healthy
  DTC4 and authenticated same-volume return: preserved
  FILE becomes reusable and both mounts retire cleanly.
* v1526 independent child process cwd/FILE ownership:
  unmount denied while owned, accepted after clean release.

Native v1529 new-file resize and independent real RBN0145
Type-551 write/read regressions also PASS.

## Remaining features and footprint

No production KCORE/MRES, simulator code, or media-format
change was needed for v1530. P0-B still requires explicit
checked administrative repair/discard of WRITE_UNCERTAIN
mounts and safe handling of retained FILE/cwd, PIPE and
swap ownership. This cannot rely solely on matched 72-bit
physical media identity. P0-A and P1-C/P1-D/P1-E remain
open. VERSION 0.5, 32K/optimization HOLD.

Existing optional DTFS/TSFS MRES remains octal 003747 text,
000002 BSS, zero additional fixed or permanent RAM words.

Detailed evidence: DAIMOS/design/
kcore-physical-test-media-isolation-v1530.md and
 daimos-testkit/tests/validation/
daimos-dtfs-isolated-media-v1530.md.
