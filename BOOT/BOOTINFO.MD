# PDP-6 STAGE1 TO KINIT HANDOFF

STATUS: CURRENT IMPLEMENTATION SUMMARY.

## 2026-10-28 v638: 32K KINIT image boundary

### v639: optional KINIT stack watermark bounds

### v640: fatal-root stack watermark measurement

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
