# P1-E v1651: 48K physical MM occupancy and fragmentation checkpoints

Date: 2026-11-17. Original accepted PDP-6 v1643 physical
secondary-DTFS six-PACKED-child trace, read error and recovery.
The original full v1643 physical source-linked auditor passes
again. v1651 does not create a new PDP-6 boot or guest image
and does not change production kernel/MM/MRES/VFS/RAM.

The one original physical MM arena, octal
`042561..140000`, contains 31,375 decimal managed
36-bit words. The lower 17,777 physical words of a
49,152-word/48K machine are outside that arena and
are not individually classified by this MM table.
Each stage is checked against the original sorted
two-word MM descriptor, type, extent owner, pin count
and exact arena bounds.

| Original stage | Occupied words | Free words | Largest free gap |
| --- | ---: | ---: | ---: |
| B | 27,163 | 4,212 | 883 |
| A | 27,163 | 4,212 | 883 |
| FULL_PINNED | 27,668 | 3,707 | 883 |
| ROLLED_BACK | 23,572 | 7,803 | 4,096 |
| LATER_FULL | 27,668 | 3,707 | 883 |
| Z | 4,958 | 26,417 | 26,417 |

At FULL_PINNED the source contains five PACKED suffix
owners of 3,456 words each and the separate 4,096-word
PROCESS destination (pinned alongside its RAW source),
plus the permanent parent PROCESS. One previous child
had already exited, releasing a 3,456-word suffix.
The MM KERNEL_DYNAMIC total also declined by 135 words.
Therefore occupied RAM rose only
`4096 - 3456 - 135 = 505` words relative to B/A.
That delta must not be called the isolated cost of
FULL restoration.

Despite 3,707 total free words at FULL_PINNED, the
largest contiguous gap is only 883 words. Only 640
words of that gap lie after a valid 02000-octal
(1,024-decimal) APR alignment boundary. A new
minimum APR cannot fit. After the genuine failed
DTFS read and rollback, the 4,096-word temporary
destination disappears and exactly that space is
returned to the free MM arena. Final Z has no child
PROCESS/PACKED owners and only the parent PROCESS
plus seven kernel-dynamic extents.

The new testkit accountant independently rejects
three manipulated physical FULL snapshots: enlarged
arena, shortened kernel-dynamic extent and falsely
unpinned FULL destination. The original full v1643
source/error/rollback/return-and-WAIT auditor also
passes, including its existing falsifications.

DAIMOS report:
`design/kcore-p1e-48k-mm-sampled-peak-v1651.md`.
Testkit:
`tests/diagnostics/pure-peak-ram-v1651/`,
commit `9815f0c` on
`test-kcore-p1e-physical-ram-v1651-20261117`.

**Scope:** exact observed 48K managed-MM snapshots,
NOT an all-time whole-machine transient maximum,
NOT a matched PURE-on/off saving for the larger
v1643 guest, and NOT 32K/64K or KA10 acceptance.
The earlier matched v1639 +1,314-word net advantage
belongs to a different physical guest workload.
P1-E OPEN, PURE OFF by default, strict 32K HOLD.
