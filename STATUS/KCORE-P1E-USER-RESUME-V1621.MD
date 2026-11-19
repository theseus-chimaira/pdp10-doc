# P1-E v1621: physical restored child executes user code

2026-11-16. Experimental TSFS/no-AUX
PDP-6 PURE MRES; default remains OFF.

The full original 48K guest is traced
through a source-linked v1384 MM commit
and unpin, original READY_ONCE|slot2
publication and fixed KCORE slot-2
selection. Real CPU execution resumes.

The guest then emits a unique user
lowercase 'w' before the first timed
continuation finishes. The exact source
of the /SYSTEM/INIT PURE DXR image in
the root TSFS manifest prints this
only after a successful PIPE read and
checks of two private mutable sentinels.
That source path then requests child
exit status 057 octal. The v1621
testkit refuses a missing user marker
or an unrelated source file.

The independent physical fitting
negative-control guest uses four
children and a smaller BRK:

    parent BRK    010000 -> 016000
    process words 052450 -> 060450

The complete +006000-octal process
growth equals BRK growth; no net
process residency reduction is
visible. The original physical
guest rightly emits !R. This is
NOT a successful parent WAIT/EXIT
test, despite later child output.

Another real 48K guest requesting
six children printed five ready
markers followed by !c, before
any BRK request. This is either
an invalid RUN result or failure
of the bounded child-to-SLEEP
check; the present output cannot
distinguish them. The testkit
records and mutation-tests the
early refusal, not a guessed cause.

The installed GCC PURE MRES itself
occupies 1758 decimal words. Four
omitted 512-word prefixes would
save 2048 words gross at most,
only 290 more than the module before
transient costs. No positive net or
peak physical memory measurement
is established.

Next correctness gate: a genuinely
pressure-inducing yet allocatable
48K/64K workload, real all-child
PIPE/WAIT/EXIT and backing lifetime,
then exact physical RAM accounting.
No production kernel, MRES, process
descriptor or tape driver was changed.

Authoritative DAIMOS record:
design/kcore-p1e-physical-user-resume-v1621.md.
