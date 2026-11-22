# DAIMOS dirty-worktree preservation audit v1810

Date: 2026-11-22. Scope: preservation of unreconciled code,
tests and documentation. No source promotion or new KCORE
physical acceptance is claimed by this audit.

Seventeen dirty DAIMOS, daimos-testkit, toolchain, pdp10-doc
and simulator worktrees were inventoried. Each has a versioned
archive containing modified/untracked files, source hashes,
status and binary diff. All archive SHA256 values passed
verification. Local device archive:

    ~/backups/daimos-dirty-worktrees-v1810-20261122/

The archived pre-v1809 mainline dirt and its original
annotated Git stash tags are retained separately inside
the same backup directory. No dirty worktree was reset.

The highest-value source candidates are the uncommitted
pdp10-tools linker fix for declarations with no relocation,
the p10run SIMH timeout/step-expiry false-PASS guard, and
DAIMOS's CAT S6REC streaming and MAKE string-arena goal
storage. The old MAKE v1414 experiment reports a reduction
of the main frame from 01113 to 0172 octal words; this
measurement is historical and needs current-source retesting.
Relevant native KCC/DAS tests, K01 F07 failure probes,
v1682 physical mutation controls, and DTFS/BCACHE tests
also remain in archived dirty daimos-testkit worktrees.

The older dirty D6CODEC v1458 fixed decoder is superseded
by the 53-word production decoder. The KCC and PDP-6 dirty
binary outputs are build artifacts, not source patches.

Separate, previously unreachable implementation/reference
commits were protected by explicit annotated origin tags:
DAIMOS offline MKSWAP v1073 and odd-even MM plan, plus
testkit odd-even MM reference, PTR/TSFS image evidence and
TTY PI6 v1147. Their preservation does not imply integration.

The source and test triage, original worktree identifiers,
all five archive tags and outstanding negative-test gates
are documented in DAIMOS
`design/daimos-dirty-worktree-audit-v1810.md`.

K01-K04 remain CLOSED under v1803 contracts. K05-K09,
RAW PURE production acceptance, and bitmap mode3 remain
OPEN and disabled by default. Toolchain/assembler correctness
regressions precede further KCORE RAM optimization.
