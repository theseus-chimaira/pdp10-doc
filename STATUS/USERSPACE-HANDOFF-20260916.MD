# DAIMOS Userspace Bootstrap Handoff

Date: 2026-09-16
Revision: v3

## 1. Purpose

DAIMOS has reached the userspace transition, but the first real INIT/LOGIN/DSH
boot is not yet accepted.  Several bootstrap and toolchain defects were found
while trying to run the first EXEC replacement test.  Those defects were fixed
and packaged before continuing userspace work.

The next implementation session should continue from the patch set and blocker
recorded here.  Do not restart the investigation from the older Stage1, KCC, or
DAS hypotheses: those defects are already isolated and fixed.

## 2. Authoritative uploaded bases

Use these uploaded repositories as the authoritative source bases for the
current archive-level patch set:

    DAIMOS              b9abad85f8538db3b5eb665f0256943e0aee2b2b
    daimos-testkit      b111035703727a0e993ac29dccba70dbde530627
    pdp10-doc           6ffcd76bfbeb79b190f3013554486564ddc95e49
    pdp10-tools         40e76f099167fa99656bf46acf03812f5fffe044

The DAIMOS base already contains credentials, filesystem permission enforcement,
generic TTY routing, EXEC replacement, the Stage1 stack fix, and the KCC PDP-6
flag fix.  Do not apply the older credentials, v4, or v5 DAIMOS patches to this
base.

The pdp10-tools base already contains the dlink RUN/WAIT/GETPID/PROCCTL veneer
relaxation commit.  Its archive-level SHAR is therefore validation-only and
must not manufacture a duplicate source change.

## 3. Required toolchain installation order

Install the current local.tar.gz into $PDP10_PREFIX first.

Then build/install the current source copies of pdp10-tools and DAS into the
same prefix.  The packaged copies of DAS/pdp10-tools in local.tar.gz are stale
for this work.

The uploaded pdp10-tools archive already contains the required dlink process-UUO
relaxation.  No additional pdp10-tools source delta is required.

DAS is still a separate prerequisite and is not part of these four uploaded
archives.  The verified DAS label/address-boundary and indexed-XCT fixes must be
present in the DAS installed into $PDP10_PREFIX before DAIMOS runtime testing.
The matching pdp10-testkit DAS regressions should likewise be retained.

## 4. Unified archive-level patch set

Exactly one SHAR now corresponds to each uploaded archive:

1. DAIMOS: `daimos-userspace-bootstrap-unified-20260916-v8.shar`
2. daimos-testkit: `daimos-testkit-userspace-bootstrap-unified-20260916-v7.shar`
3. pdp10-doc: `pdp10-doc-userspace-bootstrap-unified-20260916-v2.shar`
4. pdp10-tools: `pdp10-tools-userspace-bootstrap-unified-20260916-v2.shar`
   (validation-only because the required dlink change is already HEAD)

The DAIMOS unified patch contains only the KINIT diagnostic-lifetime change and
the userspace roadmap update because all earlier userspace/PDP-6 work is already
in the uploaded DAIMOS base.  The daimos-testkit unified patch contains the
userspace/PDP-6 regression set plus the KINIT diagnostic-lifetime regression.
Do not apply the superseded v1/v4/v5/v6/v7 archive patches on top of this set.

## 5. Completed and packaged implementation work

### 5.1 Credentials and permissions

The current uploaded DAIMOS base already includes the credential layer:

- real process UID and GID;
- credential inheritance;
- privileged credential-changing operations;
- D6FS/VFS use of owner/group/other mode fields;
- no expansion of the three-word process descriptor.

The credential word lives in the stable u-area rather than the process table.

### 5.2 Generic logical TTY routing

Unified v5 adds generic CTY/DCS/GE process terminal routing instead of the old
CTY-only process input/output path.

The controlling logical TTY is used by standard input/output/error.  DCS/GE
input uses existing per-TTY state for deferred characters rather than a large
per-terminal permanent queue.

This is needed for independent logins on:

    tty 0   CTY
    tty 1   DCS0
    tty 17  GE0

### 5.3 True EXEC replacement

Unified v5 adds true image replacement.  The new image is staged before the old
one is released.  Successful EXEC keeps the process identity and kernel-owned
state while directly entering the new image instead of returning through the
old userspace stack.

State intended to survive includes PID, u-area, credentials, cwd, descriptors,
session, process group, and controlling TTY.

### 5.4 Stage1 stack collision

Stage1 previously used a pushdown list at 076400 while current KINIT could load
through 076575.  Disk loading overwrote Stage1 return addresses and eventually
returned into the old paper-tape read-in loop.

Unified v5 moves the transient Stage1 stack out of the KINIT load range.

### 5.5 KCC PDP-6 flag integration

DAIMOS used bare KCC -O.  Current KCC treats that spelling as an output-file
option, so it consumed the following -x=pdp6 argument and generated later-CPU
instructions such as ADJSP.

Unified v5 uses the optimizer spelling that preserves -x=pdp6.  The regression
checks that unsupported ADJSP instructions do not enter the PDP-6 KINIT image.

### 5.6 KINIT SIXBIT diagnostic lifetime

KINIT used the Stage1 fixed SIXBIT helper at 077760.  The growing disposable
KINIT stack can legitimately overwrite that Stage1 scratch area before late
boot diagnostics run.

v6 makes KINIT diagnostics self-contained.  Stage1 may still use its own fixed
helper during Stage1-only failures.

### 5.7 dlink DAIMOS process UUOs

The dlink DAIMOS UUO relax table was missing RUN, WAIT, GETPID, and PROCCTL
(UUOs 074-077).  That forced small userspace programs to retain unnecessary
syscall veneers.

The pdp10-tools patch adds those process UUOs to normal relaxation.

### 5.8 DAS labeled-instruction correctness

DAS -O deleted a JRST-to-next-label even when the JRST word itself was named by
a preceding label-only line and later used as a self-modification slot.

That corrupted the DSK270 MRES dispatch word: later HRRM modified the following
MOVE instruction instead of the intended slot.

DAS v1 treats a label-only line as an address/optimization boundary for the
next emitted instruction.

### 5.9 DAS indexed-XCT table correctness

A D6FS type-conversion table was entered by indexed XCT.  DAS optimized the
consecutive table instructions as ordinary fall-through code, collapsing the
entries and causing a correctly decoded directory type to appear as FIFO.

DAS v2 identifies symbolic indexed-XCT target tables and preserves addressable
entries through the next label, including both forward and backward table
references.  It uses spill-backed assembler state rather than adding a large
fixed optimizer allocation.

## 6. Verified boot progress after the fixes

With the corrected DAS v2 and a freshly rebuilt/restored D6FS image for each
SIMH probe:

- Stage0/read-in reaches Stage1;
- Stage1 reaches KINIT;
- KINIT is PDP-6 clean with respect to the ADJSP regression;
- DSK270 MRES no longer corrupts its storage-state load;
- D6FS root discovery and root mount complete;
- the earlier root-directory-as-FIFO failure is gone;
- KINIT continues into kfs_boot_mount_ramfs().

Do not reuse a writable D6FS image between diagnostic probes unless the test
explicitly requires it.  Earlier repeated probes changed superblock DIRTY state
and produced misleading MNTERR failures.

## 7. Current first-boot blocker

The latest clean trace is past D6FS root mounting.

The hang is in the first RAMFS mount during kfs_boot_mount_ramfs():

    d6fs_boot_mount_root()          returns successfully
    kfs_boot_mount_ramfs()
      lookup RAMFS mountpoint       succeeds
      lookup /TEMP                  succeeds
      mm_alloc()                    succeeds
      MEMFS initialization          succeeds
      vfs_mount(RAMFS0, ...)
        vfs_stat(mountpoint, ...)
          provider dispatch         reaches D6FS FS_MRES_OP_STAT
          <current hang/failure area>

At the last trace the mountpoint vnode was a normal D6FS vnode and the request
was the ordinary STAT provider operation.  No production fix for this remaining
hang has been committed or packaged yet.

Continue by tracing the relocated D6FS provider-stat path for this mountpoint:
FCB lookup/read, decode, provider return, and the KCORE/MRES return bridge.
Do not change VFS mount policy until the exact non-returning operation is known.

## 8. Userspace bootstrap design to implement after boot is clean

The required process chain is:

    KINIT
      -> /SYSTEM/INIT                    PID 1
           -> /SYSTEM/EXEC/LOGIN 0       CTY
           -> /SYSTEM/EXEC/LOGIN 1       DCS0
           -> /SYSTEM/EXEC/LOGIN 17      GE0
                -> exec configured shell, initially /SYSTEM/EXEC/DSH

There is no separate long-lived logind.  INIT owns respawning.  LOGIN replaces
itself with the shell after authentication.

INIT responsibilities are deliberately small:

- parse /CONFIG/INITTAB;
- support only respawn and once initially;
- start configured LOGIN processes;
- wait for any child;
- reap adopted orphans;
- distinguish STOP/CONT from exit;
- respawn only dead respawn entries.

LOGIN must:

- receive the logical TTY number;
- establish the login session/process group;
- claim the controlling TTY and foreground pgrp;
- use stdin/stdout/stderr on that TTY;
- read /CONFIG/PASSWD;
- authenticate;
- set GID then UID;
- chdir to the account home;
- EXEC the configured shell.

## 9. Password policy for the first version

Do not implement a password hash yet.

The first /CONFIG/PASSWD format stores the password field in cleartext.

- empty password field: login immediately, with no password prompt;
- nonempty password field: prompt without echo and compare directly;
- authentication failure must remain generic.

A better verifier algorithm is deferred deliberately.

## 10. Native filesystem hierarchy

The agreed initial hierarchy is:

    /SYSTEM/INIT
    /SYSTEM/EXEC/
    /SYSTEM/LIB/
    /SYSTEM/INCLUDE/
    /SYSTEM/MANUAL/
    /SYSTEM/SOURCE/

    /OPTION/EXEC/
    /OPTION/LIB/
    /OPTION/INCLUDE/
    /OPTION/MANUAL/
    /OPTION/SOURCE/

    /CONFIG/INITTAB
    /CONFIG/PASSWD

/SYSTEM contains mandatory base-system content.  /OPTION mirrors the same
layout for optional/non-system software.  /CONFIG is the real system
configuration directory, analogous in purpose to Unix /etc.

The first DSH executable search order should be:

    /SYSTEM/EXEC
    /OPTION/EXEC

## 11. SIMH remote-login acceptance configuration

DAIMOS networking remains deferred.  The remote-login test is simulator serial
transport only.

Configure:

    DCS0  -> TCP/telnet port 1101
    GE0   -> TCP/telnet port 1201

Known SIMH forms used during investigation were:

    ATTACH DCS Line=0,1101
    ATTACH GE0 1201

The acceptance test should prove that both remote terminals can independently
reach LOGIN and DSH while CTY remains usable.

## 12. Required acceptance tests before resuming normal DSH work

At minimum:

1. boot enters distinct /SYSTEM/INIT as PID 1;
2. valid INITTAB records parse and malformed records are rejected safely;
3. INIT starts LOGIN on CTY, DCS0, and GE0;
4. LOGIN session, pgrp, controlling TTY, and foreground pgrp are correct;
5. fd 0/1/2 follow the configured logical TTY;
6. empty-password login skips password input;
7. nonempty cleartext password succeeds/fails correctly without echo;
8. LOGIN drops to configured UID/GID and cwd;
9. LOGIN execs /SYSTEM/EXEC/DSH;
10. DSH exit is reaped and LOGIN respawns;
11. STOP/CONT does not create duplicate LOGIN instances;
12. adopted orphan zombies are reaped by PID 1;
13. repeated login/logout leaks no process slot, fd, u-area, or swap backing;
14. CTY, DCS0:1101, and GE0:1201 operate independently;
15. foreground job control on one terminal does not affect INIT or another TTY.

Only after this gate passes should normal DSH feature development resume.

## 13. Important obsolete/intermediate patches

Do not use these on the current uploaded DAIMOS/testkit bases:

- daimos-userspace-credentials-20260915-v1.shar
- daimos-testkit-userspace-credentials-20260915-v1.shar
- daimos-userspace-kcc-pdp6-flags-20260915-v4.shar
- daimos-testkit-kcc-pdp6-flags-20260915-v4.shar

Their needed content is either already in the uploaded base or folded into the
unified v5 chain.

## 14. Immediate next work

1. reproduce the RAMFS mount/provider-stat hang on a freshly restored disk;
2. trace the exact D6FS provider-stat non-return/failure and add a failing test;
3. fix the root cause, preferring compiler/assembler/tool defects over kernel
   workarounds if another code-generation defect is found;
4. rerun EXEC replacement through first userspace entry;
5. implement distinct INIT/LOGIN/config files and the agreed filesystem tree;
6. configure SIMH DCS0:1101 and GE0:1201;
7. run the full multi-terminal login/respawn/job-control acceptance set;
8. rerun permanent-size measurements and regressions;
9. package each further change as the next sequential SHAR with a final git
   commit using Theseus <theseus@chimaira.net>.
