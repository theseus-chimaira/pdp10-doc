# KCORE v1529: actual partially written DECtape block verified

Date: 2026-11-13. Feature completion only, not
optimization. The installed 256K KCC PDP-6 tape-root
guest runs the unchanged v1527 KCORE and optional
DTFS/TSFS MRES (native DTC0-2 TSFS root).

## Physical partial-write acceptance

The guest mounted stamped native DTC3 read/write,
created an actual FILE on that mount, allocated
its first data block, and wrote two distinct words.
The host DETACHed DTC3 at a controlled CTY stop,
flushed and verified both words in the actual
36-bit DECtape RBN0145, and saved a full tape
preimage. Reattaching exactly the same 72-bit-ID
tape, a diagnostic-only Type-551 simulator modifier
armed a single-block interrupted-write fault:

    SET DTC3 PARTIALWRITE=0145:1

The genuine kernel FILE/VFS/DTFS + Type-551 write
changed the first data word and returned an
error before completing the second changed word.
The diagnostic persists through retries rather
than allowing an eventual silent completion.
The host DETACHed again, flushed the actual
partly written medium, and compared all 128
words with its saved preimage. Independent
Python verification required exactly one
requested new data word and one requested old
word, with no unrelated or header modifications.
It PASSed (changed only slot 1).

After the same partly damaged DTC3 returned,
the original KCORE mount still refused FSINFO,
STAT, two FILE CLOSE attempts and ordinary
UNMOUNT. The TSFS root remained accessible.
Actual target prints:

    DTFSPART V1529 WRITE_FAILED
    DTFSPART V1529 STICKY_CONFIRMED
    DTFSPART V1529 PASS

The full physical guest host runner exits zero.
This is stronger than v1528's already-detached
medium failure: it proves **mixed old/new
physical data on a partially committed block**.

## Build and independent controls

The diagnostic fault is compiled into the SIMH
Type-551 controller only with the v1529
preprocessor macro. The accepted simulator
was built using ALL standard SIMH compiler/
platform flags plus this macro, installed
under PDP10_PREFIX through a versioned
local.tar.gz. Early experimental simulators
which dropped those platform flags failed
the known v1528 baseline and were rejected;
no production KCORE bug was inferred.

The final v1529 acceptance ran in an isolated
versioned copy of the real production KCC
guest image. Earlier concurrent attempts
sharing a writable DTC3 image were discarded.

Independent controls PASS: real Type-551
DTC3 RBN0145 write/read; native source-linked
empty-file resize (one/two words); the
previously accepted v1528 one-word failed
write under the corrected simulator; and
v1527 changed-volume rejection and safe
original-volume return.

Source and protocol:
DAIMOS/design/kcore-dtfs-partial-write-physical-v1529.md
daimos-testkit/tests/validation/
daimos-dtfs-partial-write-v1529.md

No KCORE/MRES production code changes, no
on-media format or new persistent RAM.
Optional DTFS/TSFS MRES remains octal
003747 text and 000002 BSS; VERSION 0.5,
strict 32K gate HOLD.

## Outstanding features

P0-B actual physical read-error return,
failed write, and **partial write with sticky
WRITE_UNCERTAIN containment are accepted**.
P0-B **checked privileged administrative
repair/discard and safe FILE/cwd owner
retirement remain OPEN**. An exact 72-bit
volume ID match cannot certify previously
damaged directory or file data.

P0-A permanently lost critical root/BACKSTORE,
P1-C mixed-provider faults, P1-D dynamic
FSINFO inventory/capacity, P1-E swapless
PURE/MM/APR, and final feature freeze remain
OPEN. No optimization before feature freeze.
