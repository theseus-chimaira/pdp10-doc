# KCORE feature state v1527: physical DTFS media return and isolation

Checked: 2026-11-13. DAIMOS 0.5,
PDP-6 256K full KCC tape-root guest.
This is a **feature correctness** increment,
not KCORE optimization or a module
installation-policy change.

## Physical acceptance: PASS

The real KINIT installs optional DTFS/TSFS
MRES on a DTC-selected root. A disposable
KCC PID1 test running through real KCORE,
VFS, FILE, MM mount-table and Type-551/136
I/O mounts **three distinct stamped**
volumes in sequence: original DTC3,
independent DTC4, and a DTC3 replacement.
The host verifies complete DTVOL1 stamps,
both nonzero 36-bit halves, checksum
and independent identities.

- Detaching only DTC3 produces an actual
  physical read failure. Original mounted
  FILE and FSINFO refuse stale access,
  while DTC4 remains accessible.
- Reattaching a **different** stamped
  DTC3 volume does not authenticate the
  original mount. Old FSINFO and FILE
  remain refused; DTC4 FSINFO works
  before and after the mismatch.
- Detaching that replacement and
  returning the **original** DTC3
  performs real RBN0 identity validation
  against both saved 36-bit halves.
  The old open FILE becomes usable
  again, then closes. Both mounts
  unmount cleanly and FSINFO slots
  disappear.
- The guest prints `DTFSMEDIA V1527
  RETURN_VERIFIED`, then `PASS`,
  and the full-guest runner exits 0.
  The v1526 two-process cwd/FILE
  physical regression independently
  PASSed on the new v1527 image.

## Correctness defect and precise repair

BCACHE slot 0 shares the single
128-word `fs_block_workspace`.
The prior v1522 volume-identity
revalidation read DTC3 physical
RBN0 **directly** into that buffer,
bypassing BCACHE's normal tag
tracking. A different volume was
properly rejected, but the old slot-0
tag remained valid. Subsequent TSFS
parent/path metadata lookup could
return that DTVOL1 block as if it
were the prior TSFS file record,
making even healthy DTC4 FSINFO
and STAT fail.

Actual SIMH memory snapshots
excluded the common false
diagnoses: provider lock FREE
(`777777777777`), original DTC3
still published with READ_SUSPECT
(`000000200003`), independent DTC4
still published at unit 4 with
**no** suspect/uncertain flags
(`000000000004`), seven
MM mount slots intact.

The fix adds one instruction:
`PUSHJ` to the existing
`bcache_workspace_invalidate`
**before** physical RBN0 read.
No new state, no duplicate buffer,
no second cache, no change to
BCACHE's independent clean slot 1.
The v1522 native source-linked
fixture was also corrected to use
the **actual** BCACHE state BSS,
not duplicated fixture variables.
Its old seven negative mutants and
new executable stale-tag mutation
all fail while baseline PASSes.

The optional DTFS/TSFS MRES text
increased precisely **one octal word
of instruction count**: `003746`
to `003747` (i.e. one 36-bit word);
BSS stays `000002`. Fixed KCORE
is unchanged, and no persistent
RAM words were added. Full KCC
256K image build and physical
guest PASS. Strict 32K remains
on HOLD, not an optimization target
during feature completion.

## Remaining KCORE feature tasks

**P0-B:** Actual failed/partial
physical DTFS *write*, sticky
WRITE_UNCERTAIN, and checked
administrative fail-atomic recovery;
identity equality must not
automatically clear possible
on-media partial damage.

**P0-A:** Independent authoritative
proof of permanent loss of
the selected TSFS root or BACKSTORE
before any v1515 fatal HALT;
generic select/PI/timeout evidence
is not sufficient.

**P1:** Mixed storage-provider
fault matrix, dynamic capacity-aware
FSINFO/MOUNTS/DF acceptance
including 1/4/8/26 slot cases,
uncompressed swapless PURE with
MM/APR lifecycle and full feature
freeze. No KCORE optimization yet.

Primary source report:
`DAIMOS/design/kcore-dtfs-physical-return-v1527.md`.
Actual guest and native regression:
`daimos-testkit/tests/validation/
daimos-dtfs-media-return-fullguest-v1527.md`,
`daimos-dtfs-mounted-id-v1522.md`.
