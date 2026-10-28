# DAIMOS GCC multiprocess preemption regression v563 — 2026-10-28

## Accepted focused regression

The existing unmodified testkit `tests/validation/daimos-scheduler-multiproc-v1.{c,sh}`
was rerun from a fresh full PDP-6 SIMH image using the current GCC compiler.
The test runner retains its 8M-instruction/second simulator limit: an
unrestricted host-fast simulation may finish a CPU-bound loop before the
60 Hz target scheduling tick. It does not relax any guest marker-order,
process-private state, syscall/nice, or blocking disk-I/O assertions.

**PASS**: `preemption, process context, nice ABI, sleep/wakeup`.
The guest serial trace was:

    <abB2cCxxxxdDeEyyyyAxxxxyyyy314D5

All five boot users emitted their start markers (`a`–`e`) before the
first long CPU-bound checkpoint `A` of slot 1; disk wake/continuation
and per-process private context checks also passed. Earlier source notes
record a different failing trace
`<aAbB2cCxxxxxxxxdDeEyyyyyyyy314D5`, which was **not** reproduced
by the current image and throttle.

The passing test does not identify which intervening kernel/compiler/
configuration change resolved the historic failure. No code or test
oracle was changed during v563. The focused GCC preemption gate is now
satisfied; full GCC/KCC grouped-matrix acceptance is a separate open gate.
