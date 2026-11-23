# K05 v1902: bounded dual-owner RAW FULL recovery on original 48K PDP-6

Date: 2026-11-23. **Scoped physical PASS.** The original
PDP-6 source-built 48K guest recovers from a genuine native
PACKED source `mm_free` -5 refusal without discarding its
authoritative RAW source or live private executive continuation.
After the external pin is released, original slot-0 idle
maintenance frees the provisional PROCESS target, clears
TRANSITION and permits a new FULL restore. The final FULL
commit and original parent WAIT/HALT are **not yet proved**.
K05-K09 remain OPEN; RAW PURE/mode3 default OFF.

## Lifecycle rule

An original FULL transaction can leave two same-slot MM owners
when freeing the PACKED source fails after the temporary full
PROCESS target has been populated. The old single-owner retry
guard cannot unlock this state. The original scheduler cannot
run a TRANSITION-held child, and v1901 proved a foreign TERM
cannot safely discard its still-owned private kernel stack.

The v1902 optional PURE MRES therefore checks the actual
source and target descriptors, types, owner, sizes, alignment,
nonoverlap, MM capacity, pin state, storage mode, backing
identity and unchanged saved u-area. From safe slot-0 idle
context, after both pins are absent, it invokes original
native `mm_free(PROCESS)` **only on the provisional target**.
It then rescans the actual MM table, so a lost successful
free reply is reconciled without double-freeing. It clears
TRANSITION only after exactly one authoritative, unpinned
PACKED RAW owner remains. It never frees the u-area, FILE,
backing, foreign MM extent or saved executive continuation.

The permanent KCORE idle-finalizer reuses the existing
backoff clock and non-FINALIZE swap-retry bit in u-area word
067, with a tagged AC1 request through the already installed
PURE service trampoline. The scheduler cursor and original
ordinary swap service behavior are preserved. No new process
field or BSS allocation is needed. The fixed KCORE IMAGE
grows from `024147` to `024165` octal (**14 words**), with
BSS unchanged at `000415`; optional PURE IMAGE grows from
`004131` to `004615` octal.

## Source-linked physical evidence

Original KCORE SHA256:
`aa80ea2d98778298c13ae96de76c52815cfd40ad5486030e5056b436f85072f5`.
Optional PURE MRES SHA256:
`2dd7b7f7340d5b1b9542a97d0e8aca8478ac32afc3c555f0ae1ab38d0cd39a6b`.
Installed PURE base `032435` octal, original bounded FULL
loader `036704`, original PACKED free call `036012`, return
`036013`, original service return `033556`. The test-only
controller changes only the external source MM pin at
`024721` from `000004000003` to `000014000003` immediately
before native source free, observes the genuine return
`777777777773` (-5), and releases that same pin only AFTER
the first independent parent HB. It does not fake MM free,
kernel flags, native return results or a FINALIZE marker.

The second original physical trace independently examines
the newly relocated process table at `045516` octal and
slot3 u-area `046635`. The actual observations are:

| Original stage | MM extents | Slot3 metadata | Source pin |
|---|---:|---|---|
| Native refusal and first parent HB | 18 | `046635600401` | 1 |
| Next parent HB after external pin release | 17 | `046635400401` | 0 |
| Following parent HB, new FULL restore | 18 | `046635600401` | 0 |

The original MM count decreases only when the real provisional
PROCESS target is released. The original TRANSITION bit clears
at that point; the same u-area and VMbase0 remain. A subsequent
original FULL attempt acquires a new temporary PROCESS target
and TRANSITION again. This is positive evidence for autonomous
rollback and reentry, **not** for the final successful FULL
commit or terminal process cleanup.

Full original physical trace SHA256:
`3bb07f4c9a3429601dd63a3dce4832913e80fac0168fbf8a5d1fd786cb404ede`.
Independent native KCC PDP-6 tests cover source/target pins,
native target-free -5, lost-successful-free reply, malformed
and foreign MM ownership, MM capacity, FINALIZE and backing
identity, absent retry marker and saved PC/FDCTL conservation.
The read-only full-system audit executes **12 independently
corrupted physical transcript controls**; all correctly fail
at the intended safety assertion. Both native and physical
audits PASS.

### Exact native target-free and saved executive continuation

An additional source-linked original 48K PDP-6 physical
controller stops at the real KCC `mm_free(PROCESS)` call
`033245`, return `033246` (native `MM_OK=0`), and actual
compiled TRANSITION-clear store at `033275`. The slot-0
service has genuinely removed the temporary target (MM18 ->
MM17), without changing the original saved executive CTX_PC,
FDCTL, u-area, backing or authoritative PACKED source.
Original parent user HB and real PCLK advance afterward.
Source-linked exact native target-free trace SHA256:
`88e640e3eef12834a2fb6c97d3f0694280092844019c5372e35748a26fab25d5`.
An independent rebuilt identical production binary passes
the same original native controller, physical trace SHA256:
`4fa67d715017fc4e566781c624c3041cf706129a1097fa35216856e1ce5f19fb`.
SIX additional independently corrupted native-target physical
controls all reject fake MM success, retained target,
premature/stuck TRANSITION, altered saved PC and stale PCLK.
Together with the first twelve this makes **18 executed
v1902 physical negative controls**. The actual original
FINALIZE+PREVM cleanup path remains separate.

## Next K05 acceptance gate

The current resident parent is intentionally an independent
heartbeat fixture; it does not execute its final WAIT/Z path.
The next full-system fixture must verify that the newly
reentered FULL eventually commits RESIDENT, resumes the
original private executive continuation, permits all five
children to EXIT with correct typed parent WAIT results,
and reaches Z/real HALT. Continue testing permanent pins,
foreign MM ownership and genuine terminal cleanup without
bypassing the foreign-event TRANSITION safety guard.

Production design:
`DAIMOS/design/kcore-k05-dual-owner-reconcile-v1902.md`.
Tests:
`daimos-testkit/tests/validation/pure-k05-reconcile-v1902/`.
Accepted private physical evidence:
`$TMPDIR/daimos-k05-reconcile-v1902-physical-v2/`.
