# TSFS file decoder and swapless bitmap MRES: v1434 decision study

**Current v1451 decision:** the general-purpose block file decoder
has been optimized from 164 to **118 TEXT words** (v1441), while
the distinct swapless-only bitmap pair has been reduced from 55
to **30 TEXT words** (v1451), needing a 36-word transient
two-export MRES package. It uses caller-volatile registers and
no untrusted-stream validation. **Neither is production-linked.**
The figures below describe the earlier v1434 baseline, not
the latest component sizes. See
D6CODEC-BLOCK-OPTIMIZATION-V1441.MD and
D6CODEC-BITMAP-TRUSTED-OPTIMIZATION-V1451.MD.

Date: 2026-11-10. Research/test conclusion only; **no production
file format conversion or process compression activation**.

## Recommendation

1. Favor block v1441 as the future **single** general-purpose
   file/executable decoder, subject to the mandatory Stage-1
   size/layout, bounded PURE restore and native 32K/48K tests.
   D6LZ36-BLT stays production until an atomic replacement.
2. Keep the zero-presence bitmap encoder and decoder as a
   **separate optional MRES**, selected only for a verified
   swapless TSFS-root system. It must never replace the
   file/executable codec.
3. Finish PURE-only inactive-process reclamation first. Later,
   compress only genuinely cold memory-pressure victims, not
   every SLEEP or every interactive-terminal wakeup.

## Compiled-image comparison

29 actual built DXRs contain 48,487 initialized 36-bit words.
Their full-image D6LZ36 payloads total 37,310 words versus
37,190 words with block v1424. With independently compressed
256-word restart segments the totals are 37,833 versus 37,557.
The difference is small: 0.32% of the unframed compressed
payload or 0.73% in independently framed output. Some individual
executables prefer D6LZ36; actual DSH is 13,400 versus 13,617.
Host compressors can search more extensively at build time
without burdening the PDP-6 with native LZ encoding.

## Native decoder steps on compiled code

Both codecs decode identical source words. D6LZ36 uses its
v1428 BLT match optimization; v1424 uses its register-resident
block core. Tests exclude comparison loops but include the
direct-buffer frontend.

| Input | Raw | D6LZ36-BLT steps | Block v1424 steps |
|---|---:|---:|---:|
| First up to 128 words of 29 DXRs | 2,477 | 23,603 | **2,490** |
| First 2,048 initialized DSH words | 2,048 | 19,213 | **5,039** |
| First 2,048 initialized KCORE words | 2,048 | 20,181 | **2,961** |

Block uses 74-89.5% fewer retired SIMH instructions on these
samples. One PDP-6 BLT may move many words, so this is **not**
an equivalent measured hardware-time speedup. Decode-time
calculations must additionally charge BLT transfers, VFS
calls, and TSFS media movement/positioning.

| Resident decoder cost | D6LZ36-BLT | Block v1424 |
|---|---:|---:|
| Core TEXT words | 39 | 75 |
| Core plus VFS frontend TEXT | 96 | 164 |
| VFS stack frame | 72 | 40 |

Block adds 68 standalone persistent words (0.21% of a 32K
machine) while reducing the temporary VFS frame by 32 words.
Its substantial decoder instruction-count advantage justifies
considering that cost. A full linked/folded KCORE RAM
comparison is still required.

## Hard Stage-1 gate

The current Stage-1 fixed D6LZ36 core fits 39 words. Block
v1424 uses 75 words. A naive replacement adds at least 36
words before initialization/framing changes. The previously
measured MTC Stage-1 was 105 words after the BLT change; a
naive replacement alone would raise it to at least 141, above
its historical 106-word budget. The old DTC size-cap test
already fails even before any block conversion. Investigate
and fix layout/budget constraints instead of raising limits
blindly.

Do not permanently ship both complete file decoders in KCORE
as a fallback. D6LZ36 is scheduled for **replacement and deletion**,
not permanent parallel support. A real block-format decision requires one
coordinated Stage-1, KINIT/KCORE, DXR/TSFS producer, EXEC and
bounded PURE-prefix conversion, with all device and low-memory
tests. **If this cannot fit, retain D6LZ36-BLT**.

## Bitmap MRES package proof and gating

The historical v1432 standalone PDP-6 bitmap encoder+decoder assembled
to 55 TEXT words, zero static DATA/BSS. DLINK successfully
produces a normal two-export MRES package:

- Installed resident image: 55 words.
- Temporary boot package: 63 words (3 header, 1 export
  table, 55 image, 4 relocation-map).
- No history buffer, dynamic allocation, or filesystem I/O
  in the codec itself.

**Latest v1451:** 30 resident image words, 36 transient package
words, zero DATA/BSS, and no saved-register stack frame.
Nine native tests pass, including callee-saved AC preservation.
These replace the baseline v1432 sizes above. This is still
a **test-only package**, not KINIT integration or
complete ABI acceptance. The existing MRES architecture
installs permanent packed modules at boot; it cannot unload
and reload this decoder on demand. That is advantageous
for OOM recovery because the decoder remains callable.

Selection should be explicit and **absent by default** until
the real swapless lifecycle is tested. At build time include
the package only for a deliberately swapless TSFS-root
profile (ROOT=tape, TAPE_AUX=none, no BACKSTORE, and no
RAM-only TMPFS). Some TAPE_AUX/TAPE_TMPFS configuration
controls remain planned rather than fully implemented;
do not claim they already enforce the gate.

At boot verify the selected root is DTC/TSFS and that the
actual swap/backstore facility is absent. Root detection
happens before the MINIT table finishes, but enablement may
require a later check after the root has mounted. Only
then publish the stable resident encode/decode entrypoints.
Never select the bitmap for an ordinary disk/drum-root or
TSFS+AUX swap-enabled configuration.

Bitmap candidate results on actual, *hypothetically
PURE-trimmed* DSH RAM snapshots:

| State | Raw suffix | Bitmap | Native encode steps | Decode steps |
|---|---:|---:|---:|---:|
| Idle | 3,200 | 314 | 24,658 | 30,565 |
| Six assignments | 3,200 | 524 | 25,918 | 31,615 |
| Function | 5,248 | 677 | 41,400 | 50,926 |

Each direction is at most about 0.21 seconds at the user's
nominal 250k instructions/sec, not measured hardware time.
The bitmap saves another 2,676-4,571 words beyond PURE-only
reclaim on these samples, before MM ownership and code cost.

No compression on every SLEEP: a terminal shell might wake
per keystroke, and rereading immutable PURE from DECtape
could dwarf the codec time. Only a cold, quiescent
memory-pressure victim may be eligible, after proving
unfragmented APR restore headroom, precise packed length,
rollback-safe ownership, protected user buffers and
immutable executable backing. The v1451 codec already
preserves AC10..AC15. It consumes only trusted MM-produced
packed RAM, so malformed-stream validation is intentionally
absent; exact-length preflight, ownership and admission
remain the MM caller's responsibility outside 30 codec words.

The bitmap must remain RAM-only. Excluding atypically
zero-rich KINIT, it expanded the other 28 compiled DXRs
from 19,390 to 19,914 words (+2.70%). This is why the
general-purpose TSFS file codec must remain LZ-based.

## Outstanding acceptance

- Full Stage-1 redesign/limits for block v1441, all five
  boot devices, real cold TSFS reads and unified bounded
  PURE-prefix restoration.
- Linked KCORE/MRES/stack, 32K/48K RAM high-water and MM
  fragmentation after actual resident/PURE drop transactions.
- Only then promote the test-only bitmap package into an
  opt-in boot MINIT/MRES, with safe runtime gating, exports,
  preflight and process restore.

Evidence: daimos-testkit tests/validation/d6codec-files-v1433/
and tests/validation/d6codec-compressor-v1432/.
