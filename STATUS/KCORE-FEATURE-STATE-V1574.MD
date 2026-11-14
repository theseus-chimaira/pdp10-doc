# KCORE v1574: Type-136 reverse-write overrun corrected

Date: 2026-11-14. Source-matched real PDP-6/Type-551 physical
driver acceptance, independent of DTFS operator-trust policy.

## Defect

After the 128th reverse DECtape WRITE DATAO, production
`tape_dct_reverse_advance` unconditionally reset `storage_count`
to octal `0200`. The following Type-136 PI3 write acknowledgments
therefore reentered the reverse DATAO path, which transmitted a
second unintended stream from adjacent memory. A guest-visible
successful write could corrupt tape blocks unrelated to the
requested RBN, not merely the requested block. Repeated DTFS
RBN0 reads in the uncommitted v1572 experiment changed the motion
sequence and hid the corruption; they did not repair the driver.

The v1574 correction adds one conditional skip instruction: restore
the `0200` count after reverse READ, but leave it at zero after
reverse WRITE. The pre-existing two DCT write acknowledgments then
execute in their proper handler. Earlier early-completion attempts
lost leading physical data words. A strict-direction-only change
did not cure the corruption.

## Acceptance

The actual production tape driver and storage router were assembled
and executed on the PDP-6 with real Type-551 simulated physical tape.
The original counted two-block forward stream, original reverse
read/write, explicit repeated block write and exact reverse DATAO
fixture all PASS, with the entire reel compared before/after and
only RBN0200/0201 changed. Both unsafe mutants that reintroduce
unconditional count reset were executed and rejected on unrelated
physical changed blocks; a zero guest exit code alone does not pass
the oracle.

The complete 256K KCC KCORE/DTFS+TSFS MRES image links.
`__kcore_low_end` remains octal `024364`. The optional tape MRES
grows by one TEXT word, with no added data, BSS, process, mount
slab, API or on-media format. KCORE version remains 0.5, strict
32K optimization HOLD. The separately uncommitted v1572 operator-
trust admission changes are not part of this driver correction.

Original DAIMOS design: `design/kcore-type136-reverse-write-v1574.md`.
Executable testkit: `tests/validation/daimos-dtc-reverse-write-count-v1574.sh`
and the source-linked `daimos-dtc-repeat-write-physical-v1573.sh`.

Remaining validation before removing repeated DTFS v1554 RBN0
reverification: a controlled full physical DTVOL2 RW/CLEAN and
cold boot without that call; genuine partial-DATA/DIR failures,
poisoned owner and recovery regressions; then a size/tape-I/O
comparison. Operator trust does not permit unsafe hardware I/O.
