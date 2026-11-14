# KCORE v1567: physically interrupted Type-551 CLEAN and cold-boot rejection

Date: 2026-11-14. **Real controller partial-write and recovery refusal
PASS.** P0-B remains OPEN. Production DTVOL2 READ-ONLY remains mandatory.

## What the production hardware interface proves

DAIMOS's resident `tape_io.s` serializes Type-136/Type-551
transfers, saves each request's physical unit/RBN and checks normal
controller/PI errors and deadlines. It has no persistent physical
TU55 reel-generation identifier or mechanical exchange interlock.
Even a freshly validated 72-bit RBN0 header cannot prove continuous
reel attachment through a subsequent WRITE. A software volatile
generation counter cannot close an external cartridge exchange.

## Actual PDP-6 test results

Eleven cold boots of source-matched, unmodified production 256K
KCC VFS/DTFS MRES on physical Type-551 DTC3 tapes PASS. Ten are
host-generated torn DIRTY/CLEAN RBN0, identity/magic/padding,
and DIRTY with damaged DATA/DIR images. They all refuse both RO
and RW mount without publishing a DTFS VFS slot; whole tape
SHA-256 remains unchanged. The eleventh is an explicit negative
control: a fully valid CLEAN RBN0 with unrelated physically damaged
DATA. It is admitted RO, demonstrating that RBN0 alone cannot
check integrity of arbitrary DATA blocks. These eleven inputs are
offline-created crash images, not physically interrupted writes.

The separate real interrupted physical CLEAN test copies the
production image and, ONLY inside that disposable copy, uses the
real v1551/v1552 Type-551 first-write DIRTY arm. Exact linked VFS
breakpoints stop SIMH immediately before production v1566 CLEAN
and on its real sticky WRITE_UNCERTAIN failure branch. After
DETACH flush, the original 72-bit IDs and complete DIRTY RBN0
are independently verified, with all other tape words unchanged.
The diagnostic Type-551 controller is then armed with
`PARTIALWRITE=0000:1`; real CLEAN writes its first changed 36-bit
word and fails. The second breakpoint and physical DETACH prove
that the previously DIRTY state word became CLEAN but its old DIRTY
salted seal remained, and that the other 578-block tape bytes
were untouched. Real VFS refuses FSINFO, CHDIR and UNMOUNT of the
poisoned owner; unrelated TSFS root operations remain healthy.

A **second independently booted, unmodified production kernel**
mounts the exact physically torn DTC3 tape from that failure.
Both RO/RW fail, the DTFS mount slot stays unpublished and full
media SHA-256 remains identical. Old v1529 real DATA-block
partial-write simulator regression PASSes on the current image.

## Remaining release gates

The v1555 unconditional production V2 READ-ONLY veto is intact;
no new fixed KCORE TEXT or DATA/BSS, MM extent, ABI or module was
added to production. Fixed KCORE still ends at `024364` octal.

**P0-B still OPEN**: Type-551 hardware/operational generation
interlock for removable reels, true partial V2 DATA/DIR writes,
mounted media exchange/replacement during I/O, reinsertion and
ordinary user-write cold-boot recovery are unaccepted. No claim
of comprehensive DATA checksum integrity is made.

P0-A CLOSED; P1-C/D/E unchanged; VERSION 0.5; strict 32K HOLD.

Full evidence: `DAIMOS/design/kcore-dtfs-media-fault-v1567.md` and
`daimos-testkit/tests/validation/daimos-dtfs-media-fault-v1567.md`.
