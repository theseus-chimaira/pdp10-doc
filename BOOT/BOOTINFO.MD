# PDP-6 STAGE1 TO KINIT HANDOFF

STATUS: CURRENT IMPLEMENTATION SUMMARY.

## 2026-10-29 v655: native DSK0 loss and KINIT watermark build coherence

The GCC full-profile 96K PDP-6 SIMH controller-loss test detaches
DSK0 immediately after Stage1 loads KINIT. With a fresh, consistently
instrumented KINIT (`KINIT_STACK_WATERMARK=1`, reserve `0400` octal),
root initialization fails safely at the fatal HALT. Stack high-water
is `0172` octal (122 decimal words), leaving 134 words of the
256-word reserve. The original D6FS diskset remains CLEAN; testkit
`daimos-kinit-controller-media-loss-v655.sh` PASS.

This exposed an independent build dependency defect. Reusing a build
directory with different watermark settings could regenerate KINIT C
without regenerating the assembler's matching watermark/reserve
definitions. A preliminary `0577` octal report from such a mixed
image was **not** a valid stack measurement. The boot and kernel
Makefiles now use a versioned mode stamp to invalidate the C and
assembler KINIT outputs together whenever the reserve or watermark
setting changes. Testkit `daimos-kinit-mode-invalidation-v655.sh`
passes mode switches 1 -> 0 -> 1 using the same build directory.
No permanent KCORE data or instruction words were added.

## 2026-10-28 v638: 32K KINIT image boundary

### v639: optional KINIT stack watermark bounds

### v640: fatal-root stack watermark measurement

### v641: native 32K kernel-only initialization acceptance

### v642: KINIT installed-core detection at larger memory sizes

### v643: native GCC full-profile 96K initialization

### v644: native KCC full-profile 96K initialization

### v645: native KCC full-profile 256K initialization

### v646: native GCC full-profile 256K initialization

### v647: interactive full-profile ROOT login and DSH matrix

### v648: explicit disk-selector fatal-path stack measurement

### v649: native D6FS mount-validation failure watermark

### v650: native MM idle-stack allocation failure watermark

### v651: native BLOCKSET read-service I/O failure watermark

### v652: KINIT bootstrap stack reserve reduced to 256 words

### v653: missing/nonregular SYSTEM/INIT root acceptance

Native PDP-6 SIMH full-profile D6FS root acceptance rejects either
a missing `/SYSTEM/INIT` directory entry (zero child reference) or a
nonregular INIT FCB (directory type). In both cases D6FS initializes
but KINIT emits `?RT` and HALTs before starting INIT V1. Testkit
`daimos-kinit-nonregular-init-v653.sh` PASS in `missing` and
`nonregular` modes. It edits only disposable copies of CLEAN disk
media. No production kernel changes or repeated stack measurements.

The KINIT bootstrap stack default is now octal `0400` (256 decimal
words), reduced from `0600` (384 words), with consistent PDP-6 boot,
kernel-build and KINIT-header defaults. The highest previously
measured fatal-path stack watermark was `0172` octal (122 words).
The new reserve leaves 134 words above that measured high-water.

The fresh native PDP-6 32K GCC lowmem instrumented build reaches
`proc_boot_init`; linked image end `077260` plus stack `0400` is
`077660`, strictly below the `0100000` core boundary. Invalid-root
HALT records `0164` octal (116 words). These observations validate
the reduction against measured paths, not untested failure conditions.

Native PDP-6 SIMH 96K GCC full-profile KINIT is interrupted immediately
after Stage1 loading to patch only the linked in-memory
`blockset_boot_read` entry to return a storage error. The physical
disk root remains discoverable; subsequent D6FS mount read fails,
producing `?RT` and fatal HALT. The recorded stack high-water is
`0172` octal (122 decimal words), below the unchanged `0600`-octal
(384-word) reserve. The test resolves entrypoints from the linked
maps, uses no persistent code changes, and verifies CLEAN D6FS media
before and after. Testkit `daimos-kinit-blockset-read-failure-v651.sh`
PASS. Missing/nonregular `/SYSTEM/INIT` remains open.

On a GCC 96K full-profile KINIT built with stack-watermark support,
the native PDP-6 SIMH test patches only the live instruction requesting
the `0100`-word kernel idle-stack allocation. It requests `0777777`
words instead, causing the genuine `mm_alloc` error path to reach
fatal HALT before filesystem mounting or userland startup. The patch
site is discovered from the linked image; neither production source nor
persisted media is altered. Recorded high-water is `0172` octal
(122 decimal words), below the unchanged `0600` reserve (384 words).
Both source-media D6FS checks report CLEAN. Testkit
`daimos-kinit-mm-alloc-failure-v650.sh` PASS.

An instrumented GCC 96K full-profile disk-root PDP-6 SIMH boot with
both D6FS superblock magic words invalidated in isolated disk copies
selects the disk root successfully, then rejects the invalid D6FS
superblocks with `?RT` and fatal HALT. Its KINIT stack high-water is
`0172` octal (122 decimal words) versus the unchanged `0600` octal
(384-word) reserve. Testkit `daimos-kinit-d6fs-mount-failure-v649.sh`
PASS. Independent allocation, I/O, and missing-INIT failures remain
to be measured. Original media is not modified.

The instrumented native PDP-6 48K tape-root boot with console switch
selector `1` (explicit disk root rather than tape selector `2`)
reaches the fatal HALT and records KINIT high-water `0164` octal
(116 decimal words), below the `0600`-octal reserve. Testkit
`daimos-kinit-disk-selector-failure-v648.sh` PASS. This adds
selector-failure coverage, **not** independent D6FS mount or MM
allocation failure-path stack measurements. Production code is unchanged.

The four GCC/KCC x 96K/256K full-profile disk-root configurations
pass native PDP-6 SIMH ROOT login, DSH startup, `ECHO`, external
`TRUE` with `STATUS:0`, and orderly `HALT`. The final D6FS filesystem
check reports CLEAN media in every case. Testkit
`daimos-dsh-full-profile-v647.sh` provides the repeatable regression.

**Writable root media must be CLEAN before the next boot.** The KINIT
tests that break at `proc_boot_init` run *after* D6FS mounts the root
and publishes a DIRTY superblock. Such an interrupted test does not
perform orderly shutdown, and reusing its disks makes the next boot
fail with `?RT`. This is expected DIRTY-generation rejection, not a
root discovery or superblock parser failure. The v647 test checks
initial CLEAN state, isolates writable disk/drum copies, and verifies
CLEAN status after successful shutdown.

The independent GCC 256K full-profile disk-root build and SIMH boot
PASS, with KINIT image end `107421`, `0600` octal stack reserve,
correct `MEM 256 K`, DSK270/DRM236 and BSET/LOGSTR/MEMFS/D6FS
initialized and a breakpoint at `proc_boot_init` (`063067`). Testkit
`daimos-kinit-full256-gcc-v646.sh` PASS. Both 96K and 256K full-profile
GCC/KCC kernel-only gates are covered; interactive DSH remains open.

The KCC full-profile disk-root image builds with 256K configured core
and reaches the linked `proc_boot_init` boundary on native PDP-6 SIMH.
KINIT ends at octal `113573`, preserving the `0600` reserve. SIMH
reports `MEM 256 K`, DSK270 and DRM236 OK and BSET/LOGSTR/MEMFS/D6FS
loaded. Testkit `daimos-kinit-full256-kcc-v645.sh` PASS. This covers
kernel initialization only, not interactive userland or DSH.

The KCC variant of the 96K disk-root full-profile image independently
builds and boots in PDP-6 SIMH. Its KINIT ends at octal `113573`, with
the unchanged octal `0600` stack reserve intact. DSK270, DRM236,
BLOCKSET, MEMFS and D6FS initialize, and the simulator stops at the
linked `proc_boot_init` entry (`066510`) before userland execution.
Testkit `daimos-kinit-full96-kcc-v644.sh` PASS. This is not an
interactive DSH or full 256K acceptance result.

The 96K GCC full-profile disk-root image builds and boots on native
PDP-6 SIMH to the map-derived `proc_boot_init` breakpoint. Linked
KINIT ends at octal `107421`, preserving the `0600`-octal reserve;
DSK270, DRM236, BLOCKSET, D6FS, and MEMFS initialize successfully.
The regression `daimos-kinit-full96-gcc-v643.sh` covers full-profile
kernel initialization, not userspace/DSH or the KCC compiler variant.

The native testkit v642 kernel-only matrix runs one instrumented GCC
lowmem TSFS-root image with 48K, 96K and 256K installed core. All
three SIMH runs report the expected `MEM` value, initialize DTC and
DSK270 and stop at the linked `proc_boot_init` entry before userspace.
This verifies lowmem-image detection, not full-profile or DSH behavior.

The normal GCC 32K tape-root image can be tested without entering
userspace by stopping SIMH at the final-map `proc_boot_init` symbol.
Testkit `daimos-kinit-kernel-only-v641.sh` validates the unchanged
`0600` octal stack-reserve boundary, hardware initialization and
successful DTC/DSK270/DTFS discovery before userland execution.
Native PDP-6 SIMH PASS: `MEM 32 K`, expected device markers, and
breakpoint at `proc_boot_init` octal `061423`, without `INIT V1`.
No unsupported `PROC_BOOT_USERS=0` configuration is introduced.

When watermark instrumentation is enabled, the KINIT fatal HALT entry
points now record the current high-water mark before stopping. The
native PDP-6 48K tape-root test forces an invalid root selector and
reads the saved value after HALT: octal `0164`, or 116 decimal stack
words, below the unchanged octal `0600` (384-word) reserve. Testkit
`daimos-kinit-failure-watermark-v640.sh` verifies the actual measured
value and bounds. Normal uninstrumented KINIT is unchanged. This does
not establish every failure-path high-water or the 32K kernel-only gate.

The KINIT stack reserve has an exclusive end address. On the v638 32K
image, `077200 + 0600 = 0100000`, which is beyond installed 32K core.
The optional sentinel-fill and high-water scan routines now stop at the
last valid word, `end - 1`. This changes only transient KINIT diagnostics,
not normal stack allocation or permanent KCORE residency.
The instrumented lowmem GCC TSFS image links and boots on native PDP-6
SIMH with 48K core (MEM 48 K, INIT V1; testkit v639). Actual 32K
failure-path high-water acceptance remains a separate open gate.

The lowmem GCC KINIT image now ends at octal `077200`, reduced from
`077236` by 30 decimal words using PDP-6-native probe and diagnostic
routines. At 32K memory the unchanged octal `0600` stack reserve also
starts at `077200`. The image passes `mkimage` and native PDP-6 SIMH
boots print `MEM 32 K` and reach `INIT V1` on a TSFS+AUXSTORE setup.
No permanent KCORE words moved or grew. The limit is met exactly,
without spare linked-image margin: failure-path stack watermark and
additional memory/boot-profile acceptance remain open.

SOURCE OF TRUTH IS THE DAIMOS SOURCE TREE:

```TEXT
SYSTEM/STAND/PDP6/DSK/STAGE1.S
SYSTEM/STAND/PDP6/DRM/STAGE1.S
SYSTEM/KERNEL/BOOT/KINIT.H
SYSTEM/KERNEL/BOOT/KINIT.C
SYSTEM/KERNEL/BOOT/ROOT_SELECT.C
```

THE OLD 035-WORD `BOOTINFO` RECORD AT 073000, AC1 BOOTINFO POINTER, BI2
HEADER, KRES FIELDS, AND OPAQUE INIT-STREAM FIELDS ARE NOT PART OF THE CURRENT
BOOT ABI.  THEY BELONG TO AN ABANDONED EARLIER DESIGN.

## CURRENT LOW-CORE HANDOFF

DSK AND DRM STAGE1 USE TWO WORDS AT 000040 AND 000041.  TOGETHER THEY HOLD UP
TO FOUR 18-BIT MEMBER LOCATORS, ONE HALFWORD PER LOGICAL MEMBER.  EACH VALID
HALFWORD PACKS THE PHYSICAL UNIT IN THE HIGH PART AND THE MEMBER DESCRIPTOR
LOCATION IN THE LOW PART.  AN UNUSED HALFWORD IS ALL ONES.

STAGE1 BUILDS THIS HANDOFF WHILE SELECTING A COMPLETE BOOT MEMBER SET.  IT THEN
READS THE THREE-WORD DAIMON STREAM HEADER, LOADS OR DECOMPRESSES THE COMPLETE
KINIT/KCORE IMAGE AT 030000, AND ENTERS KINIT AT 030000 PLUS THE DECLARED ENTRY
OFFSET.  STAGE1 DOES NOT PASS A POINTER TO A SEPARATE BOOTINFO STRUCTURE.

KINIT SAVES 000040 AND 000041 IMMEDIATELY.  ROOT SELECTION MAY later replace
that saved pair after scanning the root medium selected by the console-switch
policy.  D6FS root discovery uses the same compact locator representation.

## ROOT SELECTION

THE LOW FIVE CONSOLE-SWITCH BITS ARE THE V0.9 ROOT SELECTOR.  THE LOW THREE
BITS SELECT AUTO, DSK, DTC, DRM, OR THE RESERVED RAM CLASS; THE NEXT TWO BITS
SELECT AN ORDINAL.  AUTO TRIES THE IMPLEMENTED ROOT CLASSES IN KINIT POLICY
ORDER.  THE SELECTOR IS NOT AN EXTRA BOOTINFO WORD.

TSFS ROOT BOOT USES ITS OWN KINIT DISCOVERY data but publishes the selected
root through the same KINIT root-selection interface.  DSK/D6FS AND DRM/D6FS
use the two-word member locator handoff.

## COMPATIBILITY

ON-MEDIA DBC/DB0/DB1/DBX RECORDS REMAIN PART OF THE DBOOT MEDIA FORMAT.  DO
NOT CONFUSE THOSE RECORDS WITH THE REMOVED HIGH-MEMORY BOOTINFO STRUCTURE.
