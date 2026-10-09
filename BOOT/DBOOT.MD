# PDP-6 STAGE1 COMPRESSED BOOT AND DSK/DRM DBOOT V1

STATUS: CURRENT IMPLEMENTATION SUMMARY.  SOURCE OF TRUTH:
`SYSTEM/STAND/PDP6/DSK/STAGE1.S`, `SYSTEM/STAND/PDP6/DRM/STAGE1.S`,
`SYSTEM/STAND/PDP6/DTC/STAGE1.S`, `SYSTEM/STAND/PDP6/MTC/STAGE1.S`,
`SYSTEM/STAND/PDP6/PTR/STAGE1.S`,
`SYSTEM/KERNEL/STORAGE/BLOCKSET_LAYOUT.H`, AND THE CURRENT DAIMOS `TOOLS/HOST`
BOOTSET IMAGE BUILDER.  THE OLD BOOTINFO HEADER IS NOT PART OF THE CURRENT ABI.

## BOOT PATH

The current production compressed-image contract is shared by all implemented
PDP-6 Stage1 transports.  DSK/DRM add DBOOT member discovery around it:

'''TEXT
STAGE0 PAPER TAPE/RIM PROXY
  -> DEVICE STAGE1 (DSK270, DRM236, DTC551, MTC516, OR PTR)
  -> DEVICE-SPECIFIC STREAM DISCOVERY/TRANSFER
  -> READ THE THREE-WORD COMPRESSED DAIMON STREAM HEADER
  -> STAGE THE D6LZ36 PAYLOAD ABOVE THE FINAL UNCOMPRESSED IMAGE
  -> INSTALL THE COMMON 40-WORD RESUMABLE D6LZ36 CORE AT 000060
  -> EXPAND THE IMAGE DIRECTLY TO 030000
  -> ENTER KINIT AT 030000 + ENTRY_OFFSET
'''

`000020..000057` remains architectural PDP-6 low memory.  Address `000060` is the fixed D6LZ36 resumable-core entry and is part of the resident contract.
KCORE links the exact same core source first, so `d6lz36_decode_core` is also
physically `000060`; no trampoline or second resident token engine exists.
Stage1 carries those exact 40 core words, whose internal transfers are encoded
against `000060+offset`, copies them to low core, initializes the decoder state,
and calls `000060` directly.  The generic memory-wrapper ABI is therefore a
boot/test convenience only and consumes no permanent KCORE words.  At runtime
the VFS frontend feeds eight-word windows to the same core and resumes only
when a window empties, without a per-word input call.

Stage1 still treats the expanded KINIT/KCORE image as opaque.  It does not
parse KINIT, MINIT, MRES, D6FS, or module metadata.  KINIT continues to own
those responsibilities after decompression.

### Compressed stream header

The normal full-profile Stage1 stream is:

'''TEXT
0  SIXBIT DAIMON
1  UNCOMPRESSED_WORDS18 ! KINIT_ENTRY_OFFSET18
2  COMPRESSED_WORDS18   ! 0
3+ RAW D6LZ36 PAYLOAD
'''

The compressed source is staged at `030000 + UNCOMPRESSED_WORDS`.  The image
builder rejects a stream when source plus destination cannot fit the configured
boot core.  The forward decoder therefore cannot overwrite unread source.
Stage1 requires both decoder success and exact consumption of the declared
compressed-word count.  The staging range is dead after expansion and becomes
the KINIT stack area.

DTC and MTC consume this stream sequentially from one medium.  PTR preserves
its operator-friendly two-tape handoff: Tape 1 carries a count followed by the
three-word DAIMON header and the first compressed segment; after `TAPE2`, Tape
2 carries a count followed by the remaining compressed segment.  The two PTR
segments are staged contiguously before the same decoder is invoked.  Type-516
MTC boot media is split into records no larger than the simulator/controller
32-KiB record buffer.  Stage1 treats EOR as a record boundary, restarts READ on
the next record, and continues until the declared compressed-word count is
satisfied; the count, not EOR, remains the image-termination oracle.

The sequential Stage1 loaders use a temporary pushdown list below the KINIT
load window while reading and expanding the stream.  Immediately before KINIT
entry they move AC17 to the first word after the expanded image, matching the
DSK/DRM handoff contract.  This is required because both the expanded KINIT
image and the subsequent KCORE low-memory copy can overwrite a fixed Stage1
stack.  Post-decode fatal paths likewise remain above the installed
`000060..000127` decoder image.

The current full-profile measurement is 23202 uncompressed payload words and
21756 D6LZ36 words, about 6.2 percent smaller.  This is only a dated image
measurement; the compression ratio is not part of the boot ABI.

### 32K KINIT profile

The low-memory profile remains governed by `BOOT_PROFILE=lowmem`,
`BOOT_CORE_WORDS=0100000`, and the configured KINIT stack reserve.  It writes a
zero `compressed_words` header and streams the raw KINIT image directly to
030000; this avoids simultaneous final-image plus compressed-staging storage on
a 32K machine.  The full profile retains D6LZ36 compressed boot.

## BOOT FLOW

The DSK Stage1 probes DSK0..DSK3 and the DRM Stage1 probes DRM0..DRM3.  Both
validate the same member descriptors, construct the same logical member map,
and read the selected bootstream in logical round-robin order while skipping
recorded bad runs.  The metadata and stream machinery are intentionally shared
by design; only raw media address formation and controller I/O are device
specific.  DRM maps one 128-word DAIMOS block to the Type-236 controller as
`unit<<16 | block<<3` and performs a polled Type-167/236 DMA transfer.

After reading and validating the stream header, a nonzero `compressed_words`
field selects the normal compressed path: Stage1 stages exactly that payload,
installs the shared decoder at 000060, expands to 030000, and jumps to KINIT.
A zero field selects the LOWMEM raw path and streams the declared uncompressed
image directly to 030000 before entering KINIT.
KINIT then reopens the selected boot medium and performs normal DAIMOS boot
initialization.

DTC, MTC, and PTR currently implement the normal compressed stream only; the
explicit 32K raw LOWMEM path remains a DSK-oriented diagnostic/acceptance
profile.

Stage1 is not a filesystem reader, module linker, or rescue OS.

## MEDIA RECORDS

THE SECTOR SIZE IS 128 PDP-10 WORDS.  STAGE1 SCANS UP TO 128 SECTORS FOR A
VALID DESCRIPTOR.

- 'DBC': COMPACT ONE-SECTOR DESCRIPTOR FOR CLEAN/EMULATOR MEDIA.
- 'DB0': BAD-RUN DIRECTORY FOR MEDIA WITH BAD SECTORS.
- 'DB1': OPTIONAL HEADERLESS BAD-RUN CONTINUATION SECTOR.
- 'DBX': DESCRIPTOR FOLLOWING THE LAST BAD-RUN SECTOR AT THE NEXT GOOD SECTOR.

DB0 WORD 000 HAS SIXBIT 'DB0' IN THE LEFT HALF.  THE CURRENT DB0 VERSION IS 1.
ITS RIGHT HALF IS:

'''TEXT
17..15  VERSION3
14..09  DB0_RUN_COUNT6
08..00  DB1_RUN_COUNT9
'''

DB0 WORDS 001..020 CONTAIN UP TO 32 BAD-RUN HALFWORDS, LEFT HALF FIRST.
WORD 021 CONTAINS THE PHYSICAL DB1 SECTOR WHEN DB1_RUN_COUNT IS NONZERO.
WORDS 022..177 ARE RESERVED.  DB1_RUN_COUNT=0 MEANS THAT DB1 IS ABSENT; THERE
IS NO SEPARATE HAS-DB1 FLAG.

DB1 HAS NO HEADER, MAGIC, VERSION, COUNT, OR NEXT-SECTOR POINTER.  ITS
128 WORDS ARE ENTIRELY BAD-RUN STORAGE, PROVIDING UP TO 256 ADDITIONAL
BAD-RUN HALFWORDS.  DB1 DOES NOT CHAIN.  DB0 PLUS DB1 THEREFORE REPRESENT
UP TO 288 BAD RUNS PER MEMBER.

EACH 18-BIT BAD-RUN HALFWORD IS:

'''TEXT
17..08  START_SECTOR10
07..00  RUN_LENGTH_MINUS_ONE8
'''

START_SECTOR COVERS THE CURRENT 02000-SECTOR BOOTSET IMAGE PROFILE.  THIS
10-BIT FIELD DOES NOT COVER THE FULL 45056-SECTOR RAW DSK270 GEOMETRY; A
FULL-SIZE REAL-MEDIA PROFILE REQUIRES A WIDER COMPATIBLE BAD-RUN DESCRIPTOR
BEFORE THAT PROFILE IS FROZEN.  RUN LENGTH IS 1..0400
SECTORS; A LONGER CONTIGUOUS DEFECT IS SPLIT INTO MULTIPLE RUNS.  PACKED
HALFWORDS ARE STORED LEFT HALF FIRST, THEN RIGHT HALF.

DBC AND DBX USE DESCRIPTOR VERSION 2.  THE CURRENT DBX/DBC SECTOR CONTAINS:

'''TEXT
000  HEADER: MAGIC ! VERSION=2 ! RESERVED COUNT ! FLAGS
001  CHECKSUM
002  GENERATION
003  BLOCKSET UUID WORD 0
004  BLOCKSET UUID WORD 1
005  MEMBER WORD: MASK16 ! INDEX4 ! DISK_COUNT4 ! FLAGS12
006  KICFG0: BOOT MODE, CONSOLE MODULE/UNIT, ROOT POLICY, INITCONF, FLAGS
012  MEMBER-LOCAL BOOTSTREAM PHYSICAL START/COUNT
013..067  KERNEL/MODULE CONFIGURATION RESERVED FOR KINIT POLICY
070..177  RESERVED
'''

THERE IS NO S1CONF IN THE CURRENT BOOT PATH.  COMPATIBILITY OPTIONS THAT TRY TO
EMIT S1CONF OR THE OLD INITFS0 FALLBACK BUNDLE ARE REJECTED BY THE IMAGE
GENERATOR.

## DAIMON HEADER

The current Stage1-facing DAIMON stream header is the three-word compressed
header documented under **Compressed stream header** above.  Older four-word
KRES/init-stream header descriptions are historical and are not the current
Stage1 ABI.

## BLOCKSETS

EVERY MEMBER RECORDS A BLOCKSET UUID, GENERATION, MEMBER MASK, MEMBER INDEX, AND
DISK COUNT.  LOGICAL MEMBER INDICES MAY BE SPARSE; MEMBER MASK IS AUTHORITATIVE
AND DISK COUNT IS THE NUMBER OF SET BITS IN THAT MASK.  FOR EXAMPLE, MASK 0013
WITH DISK COUNT 3 NAMES LOGICAL MEMBERS 0, 1, AND 3.  STAGE1 ACCEPTS A BLOCKSET
ONLY WHEN ALL REQUIRED MEMBERS WITH THE SAME UUID, GENERATION, AND MASK ARE
PRESENT.

THE BOOTSTREAM ROUND-ROBIN VISITS PRESENT LOGICAL MEMBER INDICES IN ASCENDING
ORDER AND WRAPS FROM THE HIGHEST PRESENT INDEX TO THE LOWEST.

BOOTSTREAM DATA IS READ FROM CONTIGUOUS MEMBER-LOCAL CONTRIBUTIONS RECORDED IN
DBC OR DBX. THE CURRENT BOOTSET READER INTERLEAVES MEMBERS WITH ONE 128-WORD
SECTOR PER TURN, BUT THIS IS BOOT MEDIA LAYOUT POLICY, NOT GENERAL STRIPING OR
RAW DSK270 DRIVER SCOPE. THE INSTALLER MUST ALLOCATE A CONTIGUOUS PHYSICAL
CONTRIBUTION ON EVERY MEMBER AND RECORD THAT MEMBER-LOCAL START/COUNT IN DBC OR
DBX.

BAD PHYSICAL SECTORS ARE NOT D6FS OBJECTS.  DB0/DB1 BAD RUNS ARE BOOT-TIME
MEMBER/MEDIA METADATA USED BY STAGE1/KINIT TO REACH DESCRIPTORS AND BOOTSTREAM
DATA AROUND KNOWN DEFECTS.  THEY ARE NOT THE GENERAL RUNTIME BAD-MEDIA ABI.
THE NORMAL STORAGE-SET LAYER USES STABLE SOURCE->SPARE EXCEPTION REMAPS SO A
NEW DEFECT NEVER SHIFTS EXISTING LOGICAL BLOCK NUMBERS.  D6FS NEVER CREATES
HIDDEN BAD-BLOCK FCBS.  SEE 'KERNEL/BLOCKSET.MD'.

## STAGE1 DIAGNOSTICS

The size-reduced DSK and DRM Stage1 loaders no longer carry packed SIXBIT
fatal strings.
Failure is reported as an AC1 numeric reason followed by HALT:

'''TEXT
1  no readable selected device (DSK or DRM)
2  no complete selected set
3  invalid compressed-image header
4  bootstream read failure
5  D6LZ36 decompression or exact-consumption failure
'''

This keeps failure causes machine-readable while avoiding duplicate text-output
code in the paper-tape loader.

## KINIT DSK ROOT DISCOVERY

KINIT MUST EVALUATE D6FS ROOT SETS INCREMENTALLY WHILE SCANNING PHYSICAL DSK
UNITS.  AS SOON AS ALL MEMBERS OF THE REQUESTED ROOT ORDINAL HAVE BEEN FOUND,
DISCOVERY STOPS.  THIS MATCHES STAGE1 AND AVOIDS ISSUING MEDIA TRANSFERS TO
UNNEEDED TRAILING UNITS.  IN PARTICULAR, THE NORMAL SINGLE-MEMBER DSK0 ROOT
MUST NOT PROBE UNATTACHED DSK1-3 BEFORE MOUNTING D6FS.  MULTI-MEMBER ROOTS AND
LATER ORDINALS CONTINUE SCANNING ONLY UNTIL THEIR REQUIRED MEMBERS ARE COMPLETE.
