# DAIMOS PID 1: INIT AND SUPERVISOR

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
