# Actual PDP-6 DSH writable-RAM codec experiment v1426

Date: 2026-11-10. Status: **completed native read-only capture and
three-codec comparison**. This supersedes the v1425 synthetic DSH
numbers for the three states measured below. No kernel source,
executable, boot media or running guest state was patched.

## Provenance and capture procedure

The experiment booted existing, version-matched DAIMOS DSK-root
media inside the actual PDP-6 simulator, using isolated copies
of all writable disks, drums and tape media. It waited for the
CTY LOGIN prompt, logged in ROOT and entered an actual DSH V1.
The host interrupted the simulator with Ctrl-E and examined
physical memory read-only.

The sampler located the live process descriptor through **linked
KCORE symbols** (proc_table, proc_high_slot, proc_current_slot),
then verified five actual DSH entry-point instruction opcodes
against the executable DXR and linker map. There was exactly
one matching resident process: slot 2. All addresses are 18-bit
physical addresses; process relocation was not inferred from the
historical layout or assumed constant. The captured words are
36-bit values stored as 64-bit little-endian host slots.

Captured states:

1. Idle after logging in as ROOT, before explicit shell commands.
2. After six completed SET assignments and a verified ECHO.
3. After defining FOO with DEF FOO DO ECHO FUNCTIONOK DONE,
   then successfully executing the function. This grew and moved
   the process's physical extent. The sampler found and verified
   the DSH instruction signature again at the new base.

| Real shell state | Process slot | Physical base (octal) | VM words | Raw sector suffix | Nonzero words |
|---|---:|---:|---:|---:|---:|
| Idle prompt | 2 | 662000 | 18,432 | 3,200 | 225 |
| Six variables | 2 | 662000 | 18,432 | 3,200 | 435 |
| Function defined and executed | 2 | 656000 | 20,480 | 5,248 | 531 |

Between the first two captures, **214 physical suffix words
changed**. The function definition causes the VM to grow by
2,048 words, and the physical base relocates by 2,048 words.

The real DSH DXR2 has 15,330 TEXT words, one initialized writable
word and 1,011 BSS words. The prospective PURE sector boundary
is logical 15,232, retaining 114 trailing TEXT words alongside
mutable DATA, BSS, stack, heap and allocation padding. The current
DSH DXR is **not marked PURE**, so no such process release actually
occurred: the raw resident extent remained 18,432/20,480 words
during this experiment.

## Compression of actual physical suffix (36-bit words)

| Real DSH | Raw | Grouped D6CODEC v1420 | Block v1424 | RAM RLE v1376 |
|---|---:|---:|---:|---:|
| Idle | 3,200 | 246 (7.69%) | 280 (8.75%) | 263 (8.22%) |
| Six variables | 3,200 | 439 (13.72%) | 532 (16.62%) | 537 (16.78%) |
| Function | 5,248 | 524 (9.98%) | 656 (12.50%) | 677 (12.90%) |

Relative to retaining the raw physical suffix, payload-only
reductions would be:

| State | Grouped saved | Block saved | RAM RLE saved |
|---|---:|---:|---:|
| Idle | 2,954 | 2,920 | 2,937 |
| Six variables | 2,761 | 2,668 | 2,663 |
| Function | 4,724 | 4,592 | 4,571 |

No future ownership descriptor, process metadata, codec resident
TEXT, intermediate compressed buffer, fault-path reserve, or
transaction memory is included in those theoretical payload
savings. They are **not** measured production free-RAM increases.

## Strictly writable-only subset

Excluding the 114-word trailing immutable text fragment:

| State | Raw writable | Grouped | Block | RAM RLE |
|---|---:|---:|---:|---:|
| Idle | 3,086 | 128 | 166 | 149 |
| Six variables | 3,086 | 324 | 418 | 423 |
| Function | 5,134 | 408 | 542 | 563 |

This is useful for assessing a future policy that can recover the
last text sector directly from immutable backing. It does not
describe the current sector-granular drop contract.

## Compression implementation and native acceptance

- The grouped host encoder uses the established v1420 grouped
  D6CODEC format and an unmodified v1420 native PDP-6 decoder.
- The block host encoder uses the proposed v1424 36-bit LZ/RLE
  format and its compact native PDP-6 decoder.
- For a production-relevant RAM-specific baseline, the test
  calls the **actual** staged kpack_ram_rle_v1376.c implementation,
  compiled on the host with no edits. This code already provides
  bounded native measuring, encoding and decoding.
- All 18 combinations of three captured states, two suffix
  geometries, and three codec implementations roundtrip exactly
  on the host.
- Nine native PDP-6 tests PASS: for each full physical suffix,
  v1420 and v1424 decode and compare every word, while v1376
  performs its real target-side measure, encode and decode,
  compares the native compressed stream with the host output,
  and compares every decompressed word.

The v1376 encoder+decoder implementation was previously measured
at 449 standalone TEXT words. The v1424 and grouped native
implementations currently have **no target encoder**; their
164/364-word standalone totals represent only core decoders and
VFS adapters. It would be incorrect to rank net process-RAM
footprints by simply summing these unequal code scopes.

The MM allocator supports exact word-count packed extents with
alignment 1 in the inactive v1 PACKED_PROCESS reservation
prototype. Thus a 246- or 280-word compressed payload need
not consume an entire APR-sized 1,024-word physical allocation.
That legacy prototype is not a production admission or
atomic restore implementation.

## Evidence and safety boundary

Scripts and test instructions:
daimos-testkit/tests/validation/d6codec-shell-live-v1426/.
The capture writes three actual 36-bit word arrays, exact
linked-symbol and geometry metadata, raw console transcript,
isolated disk copies and SHA256 hashes to versioned TMPDIR
files. Memory dumps are **not included in Git** because real
user-process data may be private. The host rerun reproduces them
from the actual current boot build, and failed or ambiguous
process identification aborts rather than substituting synthetic
memory.

This experiment validates real memory compressibility, and
demonstrates the modeled v1425 result was conservative for
these three samples. It does not establish a safe process
compression transaction, bounded peak encoder memory, native
block/grouped encoder size or speed, process scheduler
continuation, immutable PURE backing, rollback after memory
pressure, or full 32K/48K workload acceptance.

**Next engineering gate:** decide whether a modest native block
encoder can be added with sufficient RAM savings versus the
already-tested v1376 RLE path. Measure complete encoder + decoder
TEXT and scratch cost, actual compression CPU time, in-place
owner-safe admission, and restore high-water before enabling
swapless mutable-process compression.
