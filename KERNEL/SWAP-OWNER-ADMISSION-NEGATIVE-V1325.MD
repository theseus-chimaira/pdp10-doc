# DAIMOS v1325: booted PDP-6 owner-extent pressure negative result

Date: 2026-11-07. This records a completed *negative test* in the
ongoing full-system no-exchange-partner swap-in investigation; it
does not claim full-system liveness acceptance. The DAIMOS production
kernel, MM allocator, swap service, scheduler, and permanent RAM
footprint remain unchanged.

## Actual v1322 booted 256K guest

The disposable v1322 F07 guest used the existing 24K-word-pressure
INIT image with 18 additional child admissions and genuine drum/
DSK/D6FS swapping. It attempted one real APR-aligned `020000`
octal-word (8192-word) `MM_TYPE_KERNEL_DYNAMIC` physical extent,
owned by test owner `075`, through production
`mm_alloc_aligned_noreclaim`. Preparation occurred *after* all 18
child admissions. The producer's allocation returned failure;
original guest output emitted `!t` and HALTed. The test owner had
not acquired any extent, so it did not leak one.

The SIMH postflight counters are authoritative: test owner `error=-1`,
`prepared=0`, `count=0`, `freed=0`; successful `swapout=063` octal
(51 decimal), `swapin=043` octal (35 decimal); swap-in allocator
failures, no-partner cooldown and exchanges all zero. These values
prove failure of *fixture admission*, not the production swap-in
allocation path. The strict guest runner exited nonzero as required.

The testkit v1325 classifier reopens the original address map and
SIMH transcript and verifies all these fields, the explicit `!t`
marker and HALT, while rejecting four modified counter states.
Its postmortem result PASSes. This is no basis for introducing the
previously rejected v1208/v1211 16-word swap fallback.

The historical v1308 completed 24K guest measured 15840 managed
physical words free at its BRK OOM snapshot. An extra 8192-word
extent can still fail earlier because the instantaneous extent map,
APR alignment, and allocation order at the admission stage differ
from the later BRK snapshot. Free-word totals alone cannot prove
an extent is placeable.

## Diagnostic sampler correction, v1324

The original v1306 read-only SIMH sampler required an exact number
of periodic samples. A genuine guest HALT before that diagnostic
budget was exhausted could produce an *additional* sampler error,
obscuring the actual guest's failure or completion. The versioned
v1324 sampler treats `--count` as a maximum after observing a real
guest HALT, completed SIMH postflight EXIT, and process exit zero.
It still rejects a simulator that exits without HALT, incomplete
postflight without `Goodbye`, and errors/timeouts. The sampler never
replaces the original guest acceptance validators. Both the new
early-HALT/truncation self-tests and the original v1306 checks PASS.

## Evidence and status

The original failed-guest log, address map, memory/swap counters,
read-only samples, boot configuration, and SHA256 manifest are
archived on the Termux host as:

`$TMPDIR/daimos-swap-physical-owner-v1322-setup-failure-r1.tar.gz`

SHA256: `388b09a28389749931eb8208ac008ff2f642fba753aeff933de44ad3ef56fd55`.

The calibrated v1323 1024-word (APR `02000`) single-owner fixture
was created as a new independent experiment and passed preflight;
at this documentation checkpoint it has **no final guest outcome**.
Full-system physical no-partner liveness remains OPEN.
