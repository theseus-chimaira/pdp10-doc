# PDP-6 K05: physical committed PACKED source free with false error reply

Date: 2026-11-23. **Scoped source-built original PDP-6 48K physical
PASS**, including original five-child EXIT/WAIT and real machine HALT.
K05-K09 OPEN; RAW PURE and mode3 OFF by default.

## Authoritative MM owner semantics

The original v1839 FULL restore fixed a dangerous error branch:
`mm_free(PACKED)` could physically remove the sole authoritative
RAW suffix even if its reported return value was an error. Blind
rollback of the fully loaded successor PROCESS would then destroy
the last valid mutable state. The repair checks the exact unpinned
MM owner set while PI remains excluded. Sole complete PROCESS
means source free **really completed** and RESIDENT must be
published; exact PACKED+PROCESS means a genuine refusal permits
checked destination rollback; everything else is quarantined.

The later v1855 precommit correction requires both original
owners to be exactly singly pinned before any final unpin/free.
The original v1890-v1892 physical pin2 tests exercised failure
BEFORE source deletion. This v1893 test takes the alternate path:
real successful source deletion, then a deliberately incorrect
reported MM error, testing v1839 reconciliation in the actual
full-system PDP-6 runtime rather than only the native fixture.

## Source-linked original full-system injection

The exact original source-built 48K v1855 KCORE, opt-in PURE MRES,
selected TSFS-root, secondary DTC3 compressed PURE executables,
normal five-child RUN/PIPE/BRK/writable sentinel checks and parent
WAIT/HALT guest are completely unchanged. The simulator uses
private DTC0-DTC3 copies, never unmounts or detaches DTC3, and
does NOT patch original kernel instructions.

The original physical KCORE reclaim JRST reveals installed PURE
base `032417` octal. The test reads the ORIGINAL compiled
`proc_pure_full_v1611.dobj` production symbol and relocation table
and verifies the linked MRES instructions before choosing a trap:

* `proc_pure_ram_restore_v1391` begins DOBJ `.text+000504` octal;
* the original PACKED-source `mm_free` call relocates at
  `.text+000674`, while its later target-rollback `mm_free` call
  relocates at `.text+000755`;
* the original linked `PUSHJ 17,mm_free` executes at physical
  `035310`; the following return consumer `MOVE 010,1` is at
  `035311` (the trap PC); the original complete FULL service
  return is at `033067`.

The actual physical simulator first stops at FULL pinned loader
entry `036202`, verifying both original independent slot3
MM extents: PACKED source `111200..120000` pin1 and PROCESS
target `120000..130000` pin1 (18 MM extents). It resumes the
unchanged actual loader, mutable RAW suffix overlay, exact
precommit owner guard, both unpins and genuine original
`mm_free(PACKED)`. At the return PC `035311`, BEFORE any injected
fault, original hardware AC1 is `000000000000` (MM_OK) and the
actual complete MM table shows:

* PACKED `111200..120000` **physically deleted**;
* solely surviving PROCESS `120000..130000`, pin **0**, owner 3;
* original MM extent count 18 -> 17, with all other MM owners,
  pins and selected TSFS root unchanged.

ONLY AFTER that independent physical source-free success proof,
the private test controller deposits AC1
`777777777773` (`MM_ERR_BUSY=-5`) and re-examines the original
hardware register. It resumes the untouched original v1839/v1855
reconciliation, whose exact zero-pin MM scan recognizes the sole
complete successor and returns `MM_OK=0` at source-linked FULL
service PC `033067`. The original slot3 process record at that
point is:

* meta `046031400401` octal, TRANSITION **clear**;
* VM word `010000120000`, physical resident APR base `120000`;
* scheduler word `200024000000`, original valid process state;
* MM table still 17 descriptors, solely original target pin0;
  the RAW source is gone and TSFS root unchanged.

## Real user and terminal acceptance

The actual original user INIT completes its five independent
child PIPE read, mutable sentinel and typed EXIT-status checks.
The first child outputs `w` BEFORE the source-free stop; the four
remaining children output `wwww` after recovery, followed by
parent `Z`, and the real original
`HALT instruction, PC: 000000`. The physical MM Z snapshot
contains eight extents and NO remaining slot3 PROCESS/PACKED
owner: normal recovery, process EXIT and terminal cleanup all
completed for this case.

The first test-only verifier falsely demanded the literal
uninterrupted substring `wwwwwZ`. Because the source-linked
SIMH MM snapshots intentionally interrupted those five original
guest characters (1+4), that was a verifier error. Only the
testkit checker was corrected to count exactly five genuine
`w` outputs between its original source/terminal checkpoints,
with parent `Z` and HALT mandatory. The actual physical machine
trace was NOT modified or rerun. The independent read-only audit
and all SIX separately executed forged transcript cases PASS:
actual free did not succeed, pinned sole target, foreign target,
missing injected failure result, falsely refused recovered FULL,
and still-TRANSITION published process.

## Image hashes and limitations

Original normalized real PDP-6 physical transcript SHA256:
`8d852bc8f40944035bb27a6bc773b8b4b11913b1683ae32ec2e39d41e968f0cf`.
Source-built unchanged fixed KCORE DXR SHA256:
`33bf700218a73fefcb36ebb1f1d835c1817f3a291a642898ccb3cbd5d9039d8b`.
Unchanged optional PURE MRES DXR SHA256:
`a0fb25c51cf1c57f4bc70ca1dc83d8b9ea39b1ac3ddb3bba7b9ffcea17e5b3fd`.

Original private real-machine evidence:
`$TMPDIR/daimos-k05-source-free-reply-v1893-real/`.
DAIMOS full original-MM design and provenance:
`design/kcore-k05-physical-source-free-reply-v1893.md`.
Testkit commands:
`make test-pure-k05-source-free-reply-physical-v1893` and
`make test-pure-k05-source-free-reply-audit-v1893`.
Original native v1839 background:
`design/kcore-k05-source-free-reply-v1839.md`.

This establishes only the **physical successful source-free with
lost-success reply** branch. It does NOT prove a genuinely refused
free preserving both original owners, foreign MM ownership,
failed rollback/unpin, persistent allocator outcome ambiguity,
terminal cleanup under an unresolved quarantined owner, all PI6
timings, 64K/KA10 or K05-K09 release. RAW PURE and mode3 remain
OFF by default. No production source, kernel/MM registry, optional
MRES, KCORE data, static RAM or process field changed in v1893.
