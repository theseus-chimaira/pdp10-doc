# PDP-6 SIMH interactive PC corruption: symbol namespace correction (v115)

Date: 2026-10-21

## Verified defect

An interactive SIMH console command could overwrite the PDP-6 program
counter after a startup command file had set or advanced PC.  For example,
the startup file's `DEPOSIT PC 123` followed by `EXAMINE PC` reported
`000123`, but the next interactive `EXAMINE PC` returned `000000`.
Similarly, a three-instruction PDP-6 guest stopped on a breakpoint at
`000102` and a subsequent interactive examine returned `000000`.

The cause was an ELF symbol collision.  The KX10 CPU implementation used
a global variable named `PC`; the host terminal capability library also
defines a writable symbol `PC` for terminal padding.  SIMH's interactive
readline path initialized the latter through the simulator's exported
symbol, clobbering the emulated program counter.  This was reproduced
without executing any PDP-6 instructions, distinguishing it from an
interrupt, UUO, or simulator instruction-state defect.

## Source correction

SIMH commit `1a41f45` renames the global CPU register to `kx10_pc` in
all KX10 CPU and device source references.  The externally visible SIMH
register is still named `PC`, so existing console commands, breakpoint
scripts and saved CPU register operations retain their syntax.

No CPU data structure, RAM allocation, instruction semantics, or timing
heuristics changed.  This is a symbol-namespace correction, not a runtime
PC restoration workaround.  The corrected binary has a `kx10_pc` symbol
and does not export the old global `PC` symbol.

## Validation

The two previously failing PDP-6 reproductions now pass:

| Check | Before fix | After fix |
| --- | --- | --- |
| Scripted `DEPOSIT PC 123`, then interactive examine | `000000` | `000123` |
| Breakpoint at `000102`, then interactive examine | `000000` | `000102` |
| One step from that breakpoint | Incorrect continuation risk | PC `000101` |

`daimos-testkit` commit `58150fd` adds the standalone, target-executed
`test-daimos-simh-console-pc-v115` Make target.  The test passes on the
fixed PDP-6 simulator and fails on the pre-fix installed binary.  It does
not rely on a DAIMOS disk image, a compiler, or a syscall-count threshold.

PDP-6, KA10, KI10, KL10, and KS10 builds and their SIMH register-sanity
checks passed against the renamed symbol.  The KA10/KI10 builds required
an explicit `-lm` option on the local Termux toolchain because of a
separate, pre-existing Makefile library dependency; that issue is not
included in the namespace correction.

## Subsequent DAIMOS group-022 acceptance (v120)

The full DAIMOS trap-frequency runtime test subsequently passed under
both KCC and the separately validated GCC PDP-6 compiler on the corrected
SIMH binary.  The observed startup, ECHO, MEMSTAT, LS, PS and CAT128 syscall
counts were KCC 111/16/19/33/23/18 and GCC 90/16/19/33/25/18, below their
unchanged respective ceilings of 128/35/70/70/110/45.

The functional testkit fixture explicitly sets `DSH_IMAGE_LIMIT=037000`
(commit `0f6ad07`) because the current DSH `036021` image does not fit the
strict default `036000` limit.  This remains diagnostic 256K execution,
not 32K size/boot acceptance.  The separately corrected KA10/KI10 default
math-library link is committed as sims `5f6d21b`.

Evidence: `$TMPDIR/daimos-trap-runtime-fixed-pc-v117.log` and
`$TMPDIR/daimos-trap-gcc-fixed-pc-v118.log`.  Group 022 is independently
accepted for both compilers; groups 027-030 and the full atomic non-size
matrix remain open.
