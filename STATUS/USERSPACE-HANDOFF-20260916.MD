# DAIMOS Userspace Bootstrap Handoff

Date: 2026-09-16
Revision: v4
Status: PRE-DSH USERSPACE BOOTSTRAP GATE COMPLETE

## 1. Purpose

The first real DAIMOS userspace bootstrap is implemented and accepted.  Boot now
enters a distinct `/SYSTEM/INIT` as PID 1.  INIT starts independent LOGIN
processes for CTY, DCS0, and GE0; successful LOGIN replaces itself with the
configured shell, initially `/SYSTEM/EXEC/DSH`.

This document records the final implementation, the defects found while reaching
it, and the acceptance state.  Normal DSH feature development may now resume.
Do not reopen the older Stage1, DAS, KCC, VFS, or EXEC hypotheses unless a new
regression supplies evidence for them.

## 2. Toolchain installation order

Install the current `local.tar.gz` into `$PDP10_PREFIX` first.  Then build and
install the current source copies of `pdp10-tools` and DAS into the same prefix;
the copies bundled in `local.tar.gz` are stale for this work.

The current KCC source must include the conditional-zero optimizer correction
described below.  The current DAS source must include both label/address-boundary
and indexed-XCT preservation fixes.

## 3. Defects fixed while reaching userspace

### 3.1 Stage1 stack collision

Stage1 originally placed its pushdown list inside the current KINIT load range.
A KINIT disk load could overwrite Stage1 return addresses and eventually return
into the paper-tape read-in path.  The transient Stage1 stack was moved outside
the KINIT image range.

### 3.2 KCC PDP-6 option handling

Bare KCC `-O` consumed the following `-x=pdp6` option as an output-file argument,
allowing later-CPU instructions such as ADJSP into KINIT.  The DAIMOS build now
uses the optimizer spelling that preserves the PDP-6 target option.

### 3.3 KINIT diagnostic scratch lifetime

KINIT used Stage1's fixed SIXBIT helper in storage that the growing KINIT stack
could overwrite.  KINIT diagnostics are now self-contained.

### 3.4 DAS labeled-instruction optimizer defect

DAS `-O` removed a JRST-to-next-label whose instruction word was intentionally
named by a preceding label-only line and later modified in place.  That corrupted
the DSK270 MRES dispatch word.  Label-only lines now protect the addressable next
instruction from this optimization.

### 3.5 DAS indexed-XCT table optimizer defect

DAS optimized consecutive instructions in a symbolic indexed-XCT table as if
they were ordinary fall-through code.  D6FS type conversion then collapsed to
the last table entry.  DAS now preserves addressable indexed-XCT tables for both
forward and backward symbolic references, using spill-backed assembler state
rather than a permanent fixed-RAM target table.

### 3.6 VFS six-word STAT stack-frame defect

`struct vfs_stat` is six words: type, mode, character size, word size, uid, gid.
The PDP-10 assembly wrappers for `vfs_mount`, `vfs_readchar`, and `vfs_writechar`
reserved only four local words.  A successful provider STAT overwrote saved
stack state, including the MRES return word during the first RAMFS mount.  All
three wrappers now reserve the complete structure.  A provider regression writes
all six words and verifies the return path.

### 3.7 KCC conditional-zero return-value defect

Optimized KCC could eliminate the required transfer to AC1 on a control-flow
edge where another register was known to contain zero.  This miscompiled the
root fast path in filesystem permission checking and made ordinary userspace
lookups such as `STAT /SYSTEM/INIT` fail.  KCC now preserves the edge-specific
zero transfer without disabling the general CSE optimization.

### 3.8 EXEC entry trampoline assembled into BSS

`vm_pdp6_asm.s` switched to `.bss` for `vm_pdp6_apr` and did not switch back to
`.text`.  `proc_exec_enter` was consequently allocated in BSS and zeroed at
boot.  EXEC of a dynamically spawned session leader jumped into a zero word and
looped through LUUO dispatch.  Restoring `.text` before `proc_exec_enter` fixes
the problem.  A child/session-leader EXEC regression covers this lifecycle.

## 4. Host filesystem-image support

`mkd6fs` supports a `:text` source encoding.  It packs host text as four 9-bit
DAIMOS characters per PDP-10 word and records the exact final character count.
This is used for `/CONFIG/INITTAB` and `/CONFIG/PASSWD`; configuration files are
ordinary DAIMOS text files, not generated octal-word fixtures.

## 5. Implemented native filesystem hierarchy

The initial installed tree is:

    /SYSTEM/INIT
    /SYSTEM/EXEC/LOGIN
    /SYSTEM/EXEC/DSH
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

`/SYSTEM` contains mandatory base-system content.  `/OPTION` is the parallel
layout for optional/non-system software.  `/CONFIG` is the system configuration
directory.  DSH's initial external-command search order remains `/SYSTEM/EXEC`
then `/OPTION/EXEC`.

## 6. INIT implementation

The live process chain is:

    KINIT
      -> /SYSTEM/INIT                    PID 1
           -> /SYSTEM/EXEC/LOGIN 0       CTY
           -> /SYSTEM/EXEC/LOGIN 1       DCS0
           -> /SYSTEM/EXEC/LOGIN 17      GE0
                -> EXEC configured shell, initially /SYSTEM/EXEC/DSH

INIT is deliberately small.  It:

- requires PID 1;
- parses `/CONFIG/INITTAB`;
- supports `RESPAWN` and `ONCE` only;
- rejects malformed records without losing valid following records;
- passes the logical TTY number to LOGIN in AC1;
- waits for any child and therefore reaps adopted orphans;
- distinguishes exit from STOP/CONT notifications;
- respawns only a dead `RESPAWN` entry;
- never remains involved in a successful LOGIN -> shell replacement.

The initial INITTAB is:

    0:RESPAWN:/SYSTEM/EXEC/LOGIN
    1:RESPAWN:/SYSTEM/EXEC/LOGIN
    17:RESPAWN:/SYSTEM/EXEC/LOGIN

There is no separate long-lived logind.

## 7. LOGIN implementation

LOGIN receives the logical TTY number from INIT and:

- creates a new login session/process group;
- claims the configured controlling TTY;
- makes its process group foreground on that TTY;
- uses fd 0/1/2 on the configured logical TTY;
- reads `/CONFIG/PASSWD`;
- authenticates with the temporary version-0.9 password policy;
- sets GID before UID;
- changes to the configured home directory;
- EXECs the configured shell.

The first PASSWD record is:

    ROOT::0:0:/:/SYSTEM/EXEC/DSH

The format is:

    name:password:uid:gid:home:shell

An empty password field logs in without a password prompt.  A nonempty password
is read without echo and compared directly.  Authentication failure is reported
generically.  A stronger verifier remains deliberately deferred.

## 8. SIMH serial transport

DAIMOS networking remains deferred.  The remote-login acceptance uses simulator
serial transport only:

    DCS0 -> host TCP/telnet port 1101
    GE0  -> host TCP/telnet port 1201

The default boot configuration attaches both listeners before LOGIN output is
used.  A client that connects after an initial prompt has already been emitted
can send an empty line to obtain the next LOGIN prompt; this is a simulator
carrier/prompt timing property, not a DAIMOS networking protocol.

## 9. Accepted userspace behavior

The target tests verify:

1. `/SYSTEM/INIT`, `/SYSTEM/EXEC/LOGIN`, and `/SYSTEM/EXEC/DSH` are distinct
   images;
2. boot enters `/SYSTEM/INIT` as PID 1;
3. valid INITTAB records parse and malformed records are rejected safely;
4. `RESPAWN` and `ONCE` have distinct correct behavior;
5. CTY, DCS0, and GE0 can all reach independent LOGIN and DSH sessions;
6. fd 0/1/2 output remains routed to the owning logical TTY;
7. empty-password login skips password input;
8. nonempty-password login does not echo input and reports failure generically;
9. LOGIN changes to the configured UID/GID and home directory;
10. LOGIN EXECs `/SYSTEM/EXEC/DSH`;
11. DSH exit is reaped and only the matching `RESPAWN` LOGIN is restarted;
12. STOP on one foreground terminal does not make INIT create a duplicate LOGIN;
13. CTY and GE0 remain usable while the DCS0 foreground process is stopped;
14. repeated DCS0 login/logout leaves process words, process slots, and file
    slots at the baseline values;
15. existing process-hierarchy tests cover orphan adoption/reaping;
16. existing TTY/job-control tests cover controlling TTY, foreground pgrp,
    background read stop, Ctrl-Z/CONT, and Ctrl-C;
17. the child/session-leader EXEC regression reaches the replacement image and
    is reaped by its parent.

The multi-terminal acceptance deliberately runs CTY, DCS0, and GE0 concurrently,
performs repeated DCS0 logout/login cycles, checks cross-terminal output
isolation, exercises a DCS0 foreground STOP, and verifies independent GE0
respawn.

## 10. Current resident-size observation

The accepted PDP-6 build remains within the existing non-hard resident-size
policy.  The latest measurement during this gate was:

    KCORE           021322   8914 words
    KCORE+MRES      034317  14543 words
    PERMANENT_LAST  034376  14590 words

INIT, LOGIN, and DSH are userspace images and are not permanent kernel-resident
storage.

## 11. Patch ordering

Keep patch boundaries bisectable.  Relative to the userspace handoff base, the
important later changes are ordered as follows:

1. DAIMOS VFS six-word STAT-frame correction, commit `2c504be`;
2. KCC conditional-zero optimizer correction, current commit `67d983f`;
3. DAIMOS EXEC `.text` trampoline correction, commit `a1fefc9`, plus testkit
   regression commit `ad4f331`;
4. `pdp10-tools` `mkd6fs :text` support, commit `894c6e7`;
5. DAIMOS INIT/LOGIN/configuration/multi-terminal bootstrap, commit `5519837`,
   plus testkit acceptance commit `a0eff46`;
6. this documentation update.

Do not manufacture duplicate source changes if an updated archive already
contains an earlier item in this sequence.  Apply each SHAR only to the exact
base it declares.

## 12. Next work

The pre-DSH userspace gate is complete.  Resume normal DSH implementation work:

- pipeline execution and redirection;
- foreground/background process-group operation and builtins;
- base command lookup using `/SYSTEM/EXEC` then `/OPTION/EXEC`;
- getopt help/manual integration and the deferred structured completion model;
- self-hosting utilities in the established roadmap order.

Networking remains deferred for version 0.9.  A stronger password verifier is
also deferred.
