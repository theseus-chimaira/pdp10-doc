# DAIMOS group 030: EXEC and public multi-TTY runtime (V177)

Date: 2026-10-22

Independent EXEC replacement and child-session functional tests pass
against immutable DAIMOS revision `40bc094` using testkit V174.
The testkit starts a 15-second host-output deadline when the guest
emits its pre-EXEC sentinel, rather than timing SIMH startup and
KINIT together with EXEC. A separate 120-second boot ceiling prevents
indefinite startup. This is not a CPU-cycle-based performance test:
guest-output buffering can affect the observed interval.

Both tests use headless SIMH; attempting to use the graphical `boot.ini`
for the child-session case caused an unrelated host WCNSLS open error.
EXEC replacement preserves process identity, credentials, cwd,
session, process group, terminal and descriptors. The child-session
test verifies replacement and parent reaping.

The public `make boot` V177 regression also passes against revision
`40bc094`: DCS0/GE0 input, display-routed CTY output, native EXEC
and WHICH commands, ASMUTILS native `make install`, and double-TAB
completion. This fixture applies the functional DSH image ceiling
`037000` but does not alter the independent strict `036000` size gate.
TCP clients now start only after the simulator reports Type-340
hardware; connecting during the preceding compilation timed out.

Remaining group 030 work includes redesigning the DTFS fixture to
use a supported DECtape-root configuration, GCC integration and
the full serialized regression matrix. DTFS/TSFS provider residency
remains root-selected; no secondary TSFS mount is supported.

Evidence: `$TMPDIR/daimos-exec-replace-v174.log`,
`$TMPDIR/daimos-exec-child-headless-v174.log`, and
`$TMPDIR/daimos-make-boot-multitty-v177.log`.
