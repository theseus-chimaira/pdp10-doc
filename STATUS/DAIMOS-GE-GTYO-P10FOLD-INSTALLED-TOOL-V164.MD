# PDP-6 GE0 login and Type-340 refresh: installed P10FOLD v164

Date: 2026-10-22

The GE0 LOGIN startup regression was caused by an outdated installed
`$PDP10_PREFIX/bin/p10fold`. It folded a two-word GE `CONSO`/`JRST`
polling prefix whose skip path had a different successor from the
candidate anchor. Relocating that prefix changed the execution path.

The **current source** `tools/host/p10fold.c` already rejects folds
whose final `JRST` may be skipped by a PDP-10 I/O status test. A
fresh binary built from that source emits only the `P10FOLD1` plan
header against `ge_io.dobj`; the stale installed binary emitted an
unsafe `FOLD` record. The corrected optimizer was installed to
`$HOME/cross/bin/p10fold`, and the production PDP-6 image rebuilt.

After repair, `daimos-userspace-multitty-v145.sh` passes CTY, DCS0
and GE0 login/routing, respawn, resource stability and STOP isolation.
`daimos-dpy-native-refresh-smoke-v150.sh` passes login plus both
primary and shifted Type-340 native refresh tests. The testkit's new
`test-daimos-installed-p10fold-ge-v164` target assembles GE input
and checks the *installed* P10FOLD plan for this unsafe merge; it
passes, as does the prior source-only I/O skip regression.

Evidence:
- `$TMPDIR/daimos-ge-correct-p10fold-multitty-v161.log`
- `$TMPDIR/daimos-ge-correct-p10fold-dpy-v163.log`
- `$TMPDIR/ge-source-v159.fold`

This independently accepts KCC GE0 multi-TTY and Type-340 runtime.
GCC and the full group-030 regression remain open, while group 027
is active. The 15-second EXEC timing gate and DTFS/TSFS root-selected
MRES acceptance also remain open.
