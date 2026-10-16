# AAP PDP-6 headless boot diagnostics — 2026-10-16 V1

## Scope and status

DAIMOS tape-root Stage1 can boot to `INIT V1`, `LOGIN: ROOT`, and, intermittently,
`DSH V1` on the AAP PDP-6 Type-551/Type-626 simulator. Boot is **not reliable**;
reaching the DSH banner does **not** prove shell commands work. SIMH previously
reached DSH and executed `ECHO TAPEROOTOK` successfully.

Observed independent symptoms include `INIT: BAD SYSTEM POLICY`, `INIT: MOUNTALL
FAILED`, `INIT: NO INITTAB`, `INIT: RUN FAILED`, `INIT: RESPAWN FAILED`, LOGIN
incorrectly rejecting `ROOT`, and CPU RUNNING without shell progress. An INIT
INITTAB diagnostic showed a successful open followed by a failed S6REC read;
another run showed ordinary EOF reads. These observations are not yet tied to
one root cause.

## Validated findings

- TSFS root tape image integrity check passed (53 files, 373 extents in the
  latest build). AAP DTR conversion round-tripped byte-for-byte to the SIMH
  TAP image; the actual on-controller read path can still fail.
- Guest LOGIN readchar tracing received `R O O T` and newline, with octal codes
  `122 117 117 124 012`. Independent AAP Type-626 `DATAI` tracing acknowledged
  the complete PTY-provided `ROOT` plus carriage return (`015`).
- In a separate run, the shell command driver sent `ECHO AAPBOOTOK` but DSH
  received `APBOTOK` and reported `DSH: NOT FOUND: APBOTOK`. The first loss
  point (PTY, Type 626, kernel terminal queue, or shell) is unknown.
- Type-551 status snapshots from one stalled boot showed `GO=0`, `INFO ERROR=1`,
  unit 1, fixed tape position `077206`; the CPU repeatedly executed around
  octal `017531–017536`, near process/swap management code. These are
  snapshots, not evidence that storage caused the scheduler behavior.
- In another boot, DSH appeared despite `INIT: RUN FAILED` and **zero**
  Type-551 checksum/active-state/mark-track error trace events. There is no
  established causal relationship between DTC error and INIT failure.
- AAP invalid-UUO trace had no opcode-zero UUO in its observed runs. The
  previously seen `UUO 000000000000 050000000001` is a separate unresolved
  event, not evidence of a particular bad emulator instruction.

## AAP diagnostics introduced

- `emu/tty.c`: do not replace unacknowledged input (`tti_flag`/`tti_busy`);
  `PDP6_AAP_TTY_TRACE_V1=1` reports arrival and `DATAI` acknowledgement.
- `emu/apr.c`: `PDP6_AAP_UUO_TRACE_V1=1` captures opcode-zero UUO context.
- `emu/cmd.c`: `aprwhere` captures read-only PC/fetch/memory address, mode,
  interrupt state and run state.
- `emu/dt.c`: `xexamine dt0.state dt0.pos dt0.phase` returns read-only packed
  controller/position/mark-phase state; `PDP6_AAP_DT_ERROR_TRACE_V1=1`
  reports checksum, active-state and mark-track errors with unit/position.
- `tools/dtr2dta.c`: warning-free C99 host converter changes; full round-trip
  comparison passed.

These are diagnostics, not a boot fix. AAP simulator and DAIMOS kernel behavior
must still be distinguished experimentally.

## Next investigation

1. Capture a *single* shell input sequence with host writes logged bytewise,
   simulator PTY reads and Type-626 DATAI, and guest terminal queue consumption.
2. Distinguish lost/duplicated bytes from mis-echo and driver interleaving.
3. Once shell input is reliable, repeat boot and record first failing physical
   DTC block read and process startup result. Avoid changing unrelated kernel
   startup state until the first cause is demonstrated.

Diagnostic harnesses and console captures currently reside under
`$HOME/tmp/aap-dtc-boot-v1/`; they are temporary artifacts, not committed
regression tests. Production tests belong in `daimos-testkit`.
