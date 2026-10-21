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

The full DAIMOS trap-frequency runtime acceptance remains a separate
regression gate.  Neither its syscall-frequency thresholds nor any
32K/size acceptance criterion was changed by this work.
