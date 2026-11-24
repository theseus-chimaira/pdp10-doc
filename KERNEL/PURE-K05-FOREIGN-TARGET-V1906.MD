# PDP-6 RAW PURE K05 v1906: exact failed FULL target identity escrow

Date: 2026-11-24. **Original 48K PDP-6 defect reproduced (v1905),
corrected native and full-system physical acceptance PASS (v1906)**.
K05-K09 remain OPEN and RAW PURE/mode3 stay default OFF.

## Why an MM owner count alone cannot prove completed rollback

A failed FULL restoration can retain two original MM extents:
the authoritative PACKED RAW source and the complete but still
uncommitted PROCESS destination. Both must remain owned through
the original saved private executive continuation while
TRANSITION is held. v1902 introduced bounded idle recovery of
that exact pair, freeing the provisional full target without
destroying the original RAW source or saved stack.

The old recovery counted descriptors under the original child
slot, but did not retain the original full target address. The
native v1902 test had intentionally treated an extent now
assigned to another process as unrelated. In fact, that is
ambiguous: it might be genuinely unrelated, or it might be the
same unfreed scratch target with an incorrect owner field.

The original 48K v1905 physical test demonstrates the latter:
after an actual `mm_free(PACKED)` -5, test-only corruption of
the full target at MM metadata `024725` owner3 -> owner2
left the same physical PROCESS target at `120000..130000`,
MM count18. The original recovery nevertheless cleared slot3
TRANSITION with the saved executive stack still live and the
temporary full allocation physically present. This is an
unsafe premature unlock, not successful target deletion.
Actual v1905 defect trace SHA256:
`660ca0198938bc52cb1a4496f00116d78e390e58905e94062b157c074db70ba1`.

## Corrected v1906 ownership proof

When the optional PURE MRES first detects an exact failed FULL
RAW+PROCESS pair and arms original slot0 idle recovery, it
stores the original provisional PROCESS target physical base
in the 18-bit right half of the **existing** u-area word 067.
The existing high-half bit32 nonresident swap-retry tag remains
armed. That right half ordinarily contains a swap retry clock
epoch, but while a RAW FULL is TRANSITION-held it instead
escrows the full target address; the scheduler does not dispatch
TRANSITION-held owners. No extra process table/u-area or fixed
resident kernel storage is introduced.

The PI-atomic recovery verifies the saved target address,
alignment, full words and authoritative RAW ownership. A
surviving same-slot provisional target is eligible for actual
`mm_free(PROCESS)` only at the escrowed address and exact size,
type, owner and pin0. After MM free, regardless of a lost or
failed return code, the kernel rescans the ENTIRE physical MM
table: even a foreign-owned or wrongly typed extent partially
overlapping the original full target prevents TRANSITION
clearance. The original RAW source remains untouched; no
foreign extent may be freed or retagged by slot0 recovery.

Only when the entire target is provably absent and the sole
authoritative RAW source remains unpinned may the existing
u-area recovery tag be retired and TRANSITION cleared. If
owner metadata is genuinely ambiguous, this intentionally
chooses continued quarantine over fabricated progress.

## Original source-built 48K PDP-6 acceptance

With current source-built KCORE/PURE and the original five-child
compressed TSFS/BRK/PIPE workload, native first source free
returns -5 at `036175`. After release of the external source
pin, the test controller changes only original target MM owner
at `024725` from slot3 to foreign slot2; exact target range,
type1, pin0 and size are unchanged, and original MM count18.

At three independent parent user heartbeats the entire original
MM extent set is identical to the foreign-claimed snapshot:
RAW source `111200..120000` pin0, full PROCESS target
`120000..130000` owner2 pin0, unrelated parent intact.
Slot3 keeps TRANSITION/VMbase0, its saved CTX_PC/FDCTL and
backing. Physical u[067] at `047106` is `040000120000`,
encoding the original full target base `120000`. Hardware
PCLK advances `024625`,`024725`,`025025` octal; shared
bounded backoff stays armed, intervals `0100`,`0200`,`0200`.

The controller next **repairs its own deliberately corrupt
MM owner metadata** by returning the one target at `024725`
from owner2 to original owner3. This is a test-only operator
repair, not automatic MM foreign-owner correction. The real
native target `mm_free(PROCESS)` at `033420` then returns 0,
removing the target (MM18 -> MM17). Original transition clear
at `033456` executes only after the original target is absent;
u[067] escrow is retired. The next original FULL source free
at `036175` returns 0, publishing RESIDENT at physical `120000`.
All five children complete typed EXIT/WAIT, parent Z and real
PDP-6 HALT at PC0; MM8 at Z, MM6 at HALT, no residual child
PROCESS or PACKED allocation, original selected ROOT preserved.

The source-linked PDP-6 KCC native tests PASS genuine normal MM
free, pin refusal, lost successful free replies, wrong/missing
escrow, foreign target and partial overlap safety. The original
v1612 normal service and unsafe READY mutant suite re-PASS.
**16 independently corrupted full original physical transcript
controls** all reject, including premature TRANSITION clearance,
wrong escrow, false successful native free, early APR and missing
typed WAIT/HALT.

Exact evidence:

- Fixed KCORE SHA256:
  `a201a7fdcf481b436b06694e5dd55c9367036ebd9f64c630486f854b1a4f89be`
- Corrected optional PURE MRES SHA256:
  `09f65d0ca272986b462c63580eae3c4006490004cd060529ce40f10a55949351`
- Accepted original fixed physical SHA256:
  `c22a45e4b89602b2edfbe2daa75c4294d7c4ffd828cf5094584e3f5d5832055a`
- Fixed KCORE IMAGE `024167`, BSS `000415`, byte-identical to
  v1903/v1904; optional PURE MRES IMAGE `004615` -> `004761`
  octal (+100 decimal words). No new persistent u-area/owner fields.

Testkit:
`daimos-testkit/tests/validation/pure-k05-foreign-target-v1905/`
and `daimos-testkit/tests/validation/pure-k05-foreign-target-v1906/`.
DAIMOS implementation report:
`DAIMOS/design/kcore-k05-foreign-target-escrow-v1906.md`.
Accepted physical trace:
`$TMPDIR/daimos-k05-foreign-target-v1906-physical-v1/`.

**Remaining K05:** permanently inaccessible external pins and
foreign owners, uncertain allocator metadata, terminal teardown
and additional real PI6 interleavings. The v1906 correction is
safe quarantine and operator-repaired recovery, not a means to
forcefully reclaim unknown foreign allocations. K06-K09 remain
separately OPEN; PURE and mode3 remain OFF by default.
