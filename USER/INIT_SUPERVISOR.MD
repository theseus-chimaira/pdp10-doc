## LOGIN minimal-libc v2 progress (2026-10-13)

LOGIN's initial minimal-libc split removes generic S6REC output,
unused monitor-call wrappers and its full 128-word static LOGSTORE
record. The existing SYS_LOGCTL_EVENT interface sends a four-word
event to LOGSTORE, which builds the persistent record outside LOGIN.
The fresh-build measured DXR image is 004375 octal words with zero BSS.
With 000020 origin and 001000 reserved stack, this requires 005415
octal words, rounded to 006000 (three APR blocks), instead of 010000
in v1. LOGIN authentication, PASSWD file parsing and EXEC environment
setup are unchanged. This is a tested intermediate result; the
002000-octal-word single-APR goal is not yet achieved. See DAIMOS
design/LOGIN-SHRINK-20261013-v2.MD.

# DAIMOS PID 1: INIT AND SUPERVISOR

## LOGIN one-block resident reduction (2026-10-13 v1)

LOGIN is a separate process from PID 1. Its uncompressed DXR2 executable
now carries an ST2 stack reservation of 01000 octal words. The unchanged
006133 image and 000255 BSS, plus 000020 origin, require 007430 words;
PDP-6 APR rounding now allocates 010000 instead of 012000 words. This
saves one 02000-word (1024-decimal-word) process extent while LOGIN is
active. Other executables retain their original stack reservations.

LOGIN retains a larger 01000 stack than SUPERVISOR's 00400 because it
constructs EXEC environment records on its stack and parses account
files. Its actual high-water mark under worst-case login input has not
yet been measured. The disk root installs the ordinary, uncompressed
LOGIN DXR so the D6LZ format cannot discard the ST2 extension.

The fresh PDP-6 disk build, ROOT LOGIN -> DSH, 115/115 shell conformance,
and physical AUXSTORE persisted LOGIN event all passed. The separate
32K low-memory LOGIN-to-DSH acceptance is not established. See
DAIMOS design/LOGIN-ONE-APR-BLOCK-20261013-v1.MD and daimos-testkit
daimos-login-one-apr-size-20261013-v1.sh.

## One PDP-6 APR block for resident SUPERVISOR (2026-10-12 v11)

The resident SUPERVISOR now requires exactly one **02000-octal-word
hardware allocation block** (1024 decimal words), instead of two.
The verified PDP-6 DXR contains **001177 image words** and **000051
BSS words**, with origin 000020 and a per-executable stack reservation
of 000400; the resulting 001670-word requirement rounds to 002000.

To support this safely, plain non-PURE, non-compressed DXR2 images may
carry an optional third-word `ST2` extension instead of `TX2`:
`stack_words,,SIXBIT/ST2/`. Stack reservations are validated between
0400 and 02000 words. The linker supports `--stack-words` (on-target
`--STACK-WORDS`) with an octal argument. Existing DXR1/DXR2 images
and programs not requesting ST2 retain their 02000-word stack default.
The loader records this setting in protected logical word 017 of the
new user image and checks the startup argument records and remaining
stack capacity before committing execution.

SUPERVISOR uses compact root-privileged `SYS_LOGCTL_EVENT` (suboperation
011) instead of building a full 0200-word logging record on its user
stack. The kernel's LOGSTORE MRES temporarily allocates its own 0200
word record, constructs the existing on-disk format, commits it, and
frees that scratch. The old general `SYS_LOGCTL_APPEND` remains usable
by other programs and the on-disk log format is unchanged.

Measured kernel growth is 0072 octal KCORE words and 37 decimal words
in the LOGSTORE MRES. Compared with the 1024-decimal-word process
reduction, net standing resident core saving is approximately **929
decimal words**, excluding transient event scratch and any MM descriptor
effects. The actual stack depth depends on the handoff arguments.

The production PDP-6 build and 115/115 DSH conformance pass. The
LOGDUMP test confirms persistent INIT_READY and LOGIN records; a
separate eight-entry INITTAB test with seven 102-character pathnames
reaches LOGIN on TTY0 and records all seven expected launch failures.
Both results were checked against the actual AUXSTORE drum sectors,
not merely against the in-memory LOGSTORE producer sequence.

The v11 fix also corrects a shared LOGSTORE backend registration defect:
AUXSTORE MINIT had connected the direct LOGSTORE I/O path to the raw
relocated DSK/DRM service entries (hardware address, buffer), although
the caller supplied unit, physical block, and buffer. The system now
installs the matching KCORE validation/conversion bridges. This repairs
both the original APPEND and the compact EVENT operations without
changing the established record format.

Maximum observed user-stack depth, forced repeated LOGIN respawns, and
the unrelated complete 32K LOGIN-to-DSH acceptance are not yet certified.
See DAIMOS design/SUPERVISOR-ONE-APR-BLOCK-20261012-v11.MD for the
kernel, linker, and loader contracts.

## Reuse INIT's immutable EXEC argument records (2026-10-12 v7)

The SUPERVISOR receives up to eight pathnames as counted SIXBIT EXEC
arguments. These records already remain in the process's startup
argument area below the initial stack pointer. SUPERVISOR now retains
validated pointers directly to these records rather than duplicating
the path payloads into its permanent eight-entry BSS table. They remain
readable for later service respawns, including after VM relocation.

| PDP-6 property (octal) | v6 | v7 |
| --- | ---: | ---: |
| Image | 001236 | **001210** |
| BSS | 000261 | **000051** |
| Image+BSS+origin | 001537 | **001301** |
| Initial VM, standard 02000 stack | 004000 | **004000** |

The unchanged normal stack allocation remains the barrier to the one
02000-word process target. A future per-image smaller stack reservation
must leave space for all initial EXEC argument records *and* worst-case
stack use, including the 0200-word temporary LOGSTORE record.
The complete PDP-6 build and 115/115 DSH conformance probes pass with
v7. Maximum-length eight-service and repeated-respawn stress tests
are still separate requirements.

## PDP-6 executable and alignment audit (2026-10-12 v6)

A line-by-line source review and generated-KCC comparison identified
unused code in the shared syscall object. SUPERVISOR now links only
eight ABI-identical PDP-6 UUO veneers from its own assembler file:
EXIT, WRITECHAR, RUN, WAIT, GETPID, GETTIME, LOGCTL and TTYCTL.
Its image fell from 001442 to **001236 octal words** (0204 words saved),
while BSS remains 000261 and rounded initial VM remains 004000.
The complete PDP-6 build and 115/115 DSH conformance checks pass.

The hardware minimum process relocation/protection granularity is
**02000 octal words**, not an arbitrary size in the generic physical
allocator. The PDP-6 APR relocates the upper eight logical address
bits, while ten low bits pass unchanged. Process VM creation, BRK,
compaction, and swapping therefore align process extents to 02000.
The generic MM can already allocate smaller non-process extents, but
individual process protection and relocation cannot be reduced to a
smaller unit by editing an allocator constant. User-space sharing of
one hardware page would be a different, significantly less isolated
process model and is not supported by the current DAIMOS architecture.

Further source-specific optimization and stack constraints are
documented in DAIMOS design/SUPERVISOR-LINE-APR-AUDIT-20261012-v6.MD.

## Resident RAM optimization: transient LOGSTORE record (2026-10-12 v4)

The PID 1 SUPERVISOR uses a private compile-time setting to allocate
the LOGSTORE event record on the existing reserved user stack instead
of keeping the 0200-octal-word block permanently in BSS. Log calls
are synchronous and serialized by PID 1; other programs keep their
static-record implementation.

| Measurement (octal) | v3 | v4 |
| --- | ---: | ---: |
| SUPERVISOR image | 001447 | **001442** |
| BSS | 000461 | **000261** |
| Stack reservation | 002000 | 002000 |
| Rounded resident VM | 006000 | **004000** |

The reduced BSS crosses the physical allocation boundary, saving
02000 octal (1024 decimal) words of resident core. The extra temporary
stack use during logging is 0200 octal words. A full PDP-6 boot and
115/115 shell conformance pass with the new layout. A separate 32K
LOGIN/DSH EXEC verification remains outstanding.

## Resident SUPERVISOR text cleanup (2026-10-12 v3)

The PDP-6 SUPERVISOR image has shrunk from 001530 to **001447** octal
words, with BSS unchanged at 000461. The change removes a redundant
RUN record-clearing loop, a temporary TTY string, repeated SIXBIT field
extraction, and a constant two-word copy loop. All service validation,
ONCE/RESPAWN behavior, and eight-entry capacity remain unchanged.

The normal 02000-octal-word user stack reservation is unchanged, so
the initial rounded SUPERVISOR VM remains 006000 octal words. The
full PDP-6 boot and DSH conformance pass 115/115 on the changed image.
The separate 32K PDP-6 DSH EXEC constraint remains open.

Date: 2026-10-12. Implementation: two-stage initialization for the PDP-6.

The kernel starts `/SYSTEM/INIT` as **PID 1**. INIT reads
`/CONFIG/SYSTEM`, activates storage policy, starts and waits for MOUNTALL,
and validates up to eight records from `/CONFIG/INITTAB`. It then calls
`EXEC` for `/SYSTEM/EXEC/SUPERVISOR`.

**EXEC does not create a new process: SUPERVISOR remains PID 1.** It
receives the validated INITTAB configuration through the existing bounded
EXEC argument ABI, starts the configured child processes, waits for their
termination, reaps them, and respawns services marked `RESPAWN`.
Entries marked `ONCE` are not restarted. The original eight-entry,
pathname, output-sink, and TTY capacities are preserved.

The handoff sends two counted SIXBIT arguments for every configured
entry. The first is a four-character record (two decimal TTY digits,
`R` or `O` action, `D` or `N` output sink); the second is the full
counted SIXBIT executable path. Eight entries use all sixteen supported
EXEC arguments. The resident supervisor validates the transferred data
and requires PID 1. There is no temporary configuration file and no
kernel ABI extension.

## PDP-6 memory usage

All sizes are octal 36-bit words:

| Program | Executable | BSS | Allocated initial VM |
| --- | ---: | ---: | ---: |
| Startup INIT | 004210 | 000461 | 010000 |
| Resident SUPERVISOR | **001530** | **000461** | **006000** |

The resident executable is under the `02000` octal-word text target and
saves `02000` octal words of resident VM compared with startup INIT.
INIT and SUPERVISOR each reserve the standard 02000-word stack and
020-word origin. Startup-only parsing and mount code is discarded on
the EXEC transition; it is not shared with SUPERVISOR.

The PDP-6 disk build and 115/115 DSH conformance probes pass with this
handoff. The dedicated 32K-core test still fails at DSH login EXEC;
the new PID 1 arrangement is necessary progress but is not a complete
solution to physical-core constraints.

## Additional overlays

The existing EXEC replacement supplies the most useful overlay-like
optimization: startup code is completely discarded after initialization.
An in-process overlay loader is not implemented. Any future design must
justify its net resident-text savings against relocation, loading code,
I/O latency, and safe return addresses on a PDP-6 without shared text.
The current resident supervisor is already small enough that overlays
are unlikely to be cost-effective for it.
