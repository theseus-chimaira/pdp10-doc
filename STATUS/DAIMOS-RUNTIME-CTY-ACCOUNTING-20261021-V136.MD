# DAIMOS runtime and CTY accounting validation v136

Date: 2026-10-21

## MonitorFS CTY0 accounting bug

CTY0 device STAT exposes three fixed-width, twelve-octal-digit
READCHAR lines (reads, writes, errors). The prior resident
`cty_putchar` and `cty_getchar` driver routines never advanced
their existing MonitorFS counters even when actual character I/O
completed. The focused guest test first reproduced the failure as
`<C!4`: valid counter formatting but no output count increase
after a write.

DAIMOS commit `08625cf` updates CTY0 output accounting only after
PI4 transmission completion, and input accounting only when getchar
returns a queued or directly consumed input byte. It reuses the
existing resident counter slots and does not enlarge BSS or any
terminal queues.

`daimos-testkit` commit `77f7d89` adds
`test-daimos-monitorfs-cty-stats-v132`, which checks all three
12-digit octal READCHAR lines, CR/LF, EOF and a strictly increasing
output count. The completed guest test passes under both KCC and
GCC, with logs at
`$TMPDIR/daimos-monitorfs-cty-kcc-v133.log` and
`$TMPDIR/daimos-monitorfs-cty-gcc-v134.log`.

## Runtime probe compatibility

The older runtime suite asserted a CHARS field in STAT, although
current STAT prints TYPE, WORDS, MODE, UID, GID and MTIME. Its
DF assertion also used a retired MEMFS format rather than the
current FILESYSTEM/MOUNT/TOTAL/USED/FREE table.

MonitorFS STATS is a character stream, so calling CAT on it
cannot replace the READCHAR target test: CAT consumes S6REC
word-stream records. The new
`probes/daimos-runtime-probes-v132.txt` verifies current STAT
and DF formats and STATS metadata, while the focused CTY
READCHAR test protects the data content.

The full revised **66-probe runtime suite passes under both
compilers** on fresh production 256K disk images. Evidence:
`$TMPDIR/daimos-group030-runtime-kcc-v135.log` and
`$TMPDIR/daimos-group030-runtime-gcc-v136.log`.

Only the runtime/MonitorFS portion of group 030 is accepted.
Other group-030 targets and the atomic full non-size matrix
remain open. Group 027 is deferred and strict 32K gates remain
deferred.
