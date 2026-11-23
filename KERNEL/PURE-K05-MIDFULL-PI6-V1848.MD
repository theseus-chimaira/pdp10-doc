# PDP-6 K05: real PI6 interrupt inside pinned FULL restoration

Date: 2026-11-23. **Scoped original physical PDP-6 PASS**.
K05-K09 remain OPEN; optional RAW PURE and mode3 remain OFF by default.

## Verified native hardware sequence

The exact source-built five-child v1845/v1847 48K PDP-6 system keeps
TSFS as the selected root, with immutable compressed PURE executable
backing held on independently mounted secondary DTC3. A real original
RUN/PIPE pressure sequence reaches native bounded FULL restore with
slot3's authoritative PACKED RAW suffix `111200..120000` and a
temporary full PROCESS APR `120000..130000` (octal). Both are actual
separate MM extent-table owners and are pinned exactly once.

The testkit v1848 controller arms a breakpoint at the permanent
KCORE `proc_sched_pi_tick` **after** the FULL loader's entry snapshot,
but **before** any attempted DTC3 removal. An actual physical 60Hz
PCLK PI6 arrives at original KCORE PC `017205`, executing source-
matched `350040024510` (`AOS 1,proc_timer_clock`). The actual PI6
frame saved word is `340000037637`, yielding interrupted executive
PC `037637`. The source-linked load-base calculation from KCORE's
actual patched reclaim JRST gives removable PURE MRES base `032417`.
Thus the interrupted PC is at offset `005220`, bracketed by the
original linked source symbols `vfs_create_op=005201` and
`vfs_symlink=005252`. This is a real interrupt within a module
executing as part of the original FULL transaction.

The original physical clock/cooldown/cursor at PI6 are octal
`057171/0/3`. Both source and target MM pins are STILL one, and the
complete active MM extent table is identical to its original
pre-interrupt FULL snapshot. The service continues in slot 0,
the namespace-selected TSFS root is unchanged, and no unfinished
PROCESS APR has been published as runnable.

Only AFTER this hardware PI6 observation does the host detach the
secondary DTC3. The real original FULL callback returns `-4`,
its checked rollback releases the temporary full PROCESS APR,
and the exact RAW PACKED suffix remains owned and unpinned. The
same private DTC3 reel returns and all five original child canaries,
EXIT/WAIT and HALT PC 0 complete. The complete v1845 physical
source-loss/owner/WAIT oracle is required and also passes on this
original unchanged trace.

The v1848 independent physical verifier rejects three separately
executed corrupted original traces: a missing PI6 frame continuation,
a fabricated continuation outside the linked PURE MRES range,
and a dropped target pin. In every case the exact source-linked
PI6/owner verification fails. All physical images are unmodified
from the previous accepted five-child 48K configuration.

SHA256 provenance:

* Original KCORE: `33bf700218a73fefcb36ebb1f1d835c1817f3a291a642898ccb3cbd5d9039d8b`
* Original PURE MRES: `ad246ce14079a821cd69f4c4f7b7661f5d1a0f58c118bdfa5cfa7adf2b2bfe3f`
* Original v1848 physical trace: `6375e8dc51ef6a9e05edbb2b7c9c5252f8ad8e332e9696a05ffea0495cca195d`

Testkit: `tests/validation/pure-k05-midfull-pi6-v1848/`, including
the separate physical source-linked re-audit target. DAIMOS design:
`design/kcore-k05-midfull-pi6-v1848.md`. No resident KCORE word,
process bit, MM registry, feature default or loader code changed.

## Remaining K05 acceptance

This validates **one** true PCLK interrupt *inside* a pinned FULL
transaction, and its subsequent independent source-loss rollback.
It does not establish all possible PI6 interleavings, permanently
unknown MM pin/free results, allocator failure under each owner
configuration, physical continuous write-watch memory bounds,
KA10, optional mode3, or production PURE acceptance. Preserve the
TRANSITION quarantine rule described in
`KERNEL/PURE-MM-UNCERTAIN-V1658.MD`. The earlier v1847 after-rollback
PI6 witness remains separately documented in
`KERNEL/PURE-K05-DTC3-PI6-V1847.MD` and is not conflated with v1848.
