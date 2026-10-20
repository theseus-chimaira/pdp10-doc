# AAP PDP-6 headless boot diagnostics — 2026-10-16 V1

## Scope and status

**UPDATE 2026-10-18 V23C:** The original AAP startup-to-DSH problem is
resolved with the native Type-626 bounded CTY input FIFO (v23b) and an INIT
hardware-presence check (v23c). The *unaltered* default INITTAB still lists
CTY0, DCS0 and GE0; INIT now starts LOGIN only for controllers confirmed by
MINIT in the existing MonitorFS `/DEV` namespace. A fresh AAP tape-root
diagnostic boot reaches `INIT V1`, accepts `ROOT`, and enters `DSH V1`
without respawn or authentication errors. Native/host v23c tests and the
captured AAP block-word verifier pass (234 forward, 21 reverse, zero errors).
No kernel residency is added in v23c; transient INIT gains 67 text words,
with unchanged BSS. The default INITTAB was **not** reduced to a single
session in production. The observations below are retained as historical
investigation evidence, not the current boot failure status.

**UPDATE 2026-10-19 V37:** The later, independently rerun AAP DECtape-root
test did execute external `ECHO RETESTED_OK_V24` and `PWD` successfully and
returned a fresh DSH prompt after each command. ECHO finished after 300.2
seconds and PWD after 548.2 seconds from test start. The old v43 240-second
timeout is historical evidence, not a current command-execution failure.
This was a 256K test using the experimental PI3 search path and a diagnostic
DSH image ceiling, not a 32K boot or permanent size acceptance. See
`STATUS/PDP6-SIMH-TYPE551-REGRESSION-20261019-V34.MD` and
`daimos-testkit/tests/validation/daimos-aap-boot-basic-v37.py`.

Historically DAIMOS tape-root Stage1 could boot to `INIT V1`, `LOGIN: ROOT`,
and intermittently `DSH V1` on the AAP Type-551/Type-626 simulator. Those
older runs were **not reliable**;
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
