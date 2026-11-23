# K05 v1901: real PDP-6 foreign TERM must not discard private executive context

Date: 2026-11-23. **Original full-system 48K PDP-6 physical PASS,
19 executed unsafe physical transcript controls correctly rejected.**
This is a completed, scoped event-lifetime safety proof, not
full K05 acceptance. Original fixed KCORE/PURE are unchanged
from accepted v1900 and RAW PURE/mode3 remain OFF by default.

## Observed failure and original event policy

Original FULL restore can fail with one authoritative PACKED RAW
MM owner and one complete provisional PROCESS target allocated
to the same slot. The v1900 corrected removable terminal PURE
MRES can free both only from an authentic finalized process
cleanup context. Before that, `proc_event_apply` in permanent
KCORE refuses events to a slot with `PROC_TRANSITION` set;
`proc_finish_slot` also rejects a transition owner that has not
been legitimately marked FINALIZE. Those checks protect
in-flight private executive continuations, not just MM descriptors.

v1901 executed a REAL user-mode `SYS_PROCCTL_EVENT_PID` /
`SYS_EVENT_TERM` request from the resident parent to child slot3
AFTER its original native `mm_free(PACKED)` had genuinely
refused with `MM_ERR_BUSY=-5`, preserving both original owners.
The original user dispatcher returned failure (`TR` printed
through two original user writechar calls). The process was
not killed; no FINALIZE was invented, no MM extent was freed,
and the resident parent continued normally.

## Why the rejection is necessary here

Actual physically saved original slot3 descriptor and context:

| Word | Physical octal address | Original octal value |
|---|---|---|
| Process table pointer | `024615` | `000000045014` |
| Slot3 metadata | `045025` | `046133600401` |
| Slot3 VM | `045026` | `010000000000` |
| Slot3 saved `CTX_PC` | `046153` | `340000016664` |
| Slot3 FDCTL | `046157` | `000000004010` |
| Slot3 FINALIZE | `046222` | `000000000000` |
| PACKED RAW MM source | `024703` | `000014000003` |
| PROCESS scratch target | `024705` | `000001000003` |

The slot3 saved `CTX_PC` has its original user-return bit
`010000` CLEAR in the left half. It therefore still owns a
private **executive continuation**. A foreign TERM that forcibly
calls VM/u-area release would deallocate memory required by
that continuation and is unsafe. Slot3 has TRANSITION set
and VMbase ZERO. MM extent count stays 18, including original
PACKED `111200..120000` pin1 and temporary complete PROCESS
`120000..130000` pin0. No synthetic mode or allocator result
was used to prove this. The real parent remains current slot1;
original PCLK increases `022015` -> `022115` across two new
post-fault user HBs and TERM.

## Controlled source-built physical method

The test-only five-child original ROOT/secondary DTC3,
RUN/BRK/PIPE parent guest adds one C89-compatible `dsys_procctl`
call. An exploratory initial user fixture mistakenly delivered
TERM BEFORE the original FULL fault (`HBTA`); that run is
REJECTED. The corrected fixture compiles a **test-only user
variable** `volatile kword_t v1901_term_gate`, mapped from the
original linked user DXR at logical octal `005265`. Its actual
resident parent base `130000` yields physical `135265`.

The physical controller first validates the FULL native -5,
the owner table, slot3 TRANSITION and the first independent
resident-parent HB. It then examines test-only USER word
`135265` as zero, deposits only `012345670123` into that
one USER word, and resumes the actual guest. The parent C code
performs the original SYS_PROCCTL native call ONCE and prints
`TR` if the kernel refused it. The controller does not alter
kernel memory, user syscall result, physical MM ownership,
slot3 metadata or native return PCs. This prevents a premature
event from being mistaken for a quarantined-child terminal case.

The original normalized 48K PDP-6 physical transcript SHA256:
`ce5460ee0b43593eeb158b60f11df7dd7336918e9ad0977d9afc7677ae8bc251`.
Production fixed KCORE SHA256:
`33bf700218a73fefcb36ebb1f1d835c1817f3a291a642898ccb3cbd5d9039d8b`.
Optional v1900 corrected PURE MRES SHA256:
`09bcb71e4cbb172222f6389153c27293353fdaf6d3210eeef4630ed06de934f2`.
Test-only modified user INIT DXR SHA256:
`e678526d30901f58308bf3d10bba0df0c8b0c648d75367fc32eaabca60f4ffd9`.

The v1901 read-only oracle reuses complete original v1895/v1900
physical MM/root/quarantine/PCLK checks. NINE independently
mutated physical TERM gate, result, MM owner, context and
finalizer transcript variants and TEN historical v1895 physical
negative controls all FAIL at their intended safety assertions
and are thus correctly rejected. All 19 controls PASS.

## Remaining K05 path

The ordinary PURE single-owner retry verifier
`ppure_retryable_v1612` cannot unlock an already failed FULL
with both same-slot source and scratch target present. Even
after an external pin is released, TRANSITION remains held,
so the original private executive continuation cannot resume.

The next fix should add a **bounded exact-owner rollback and
reconciliation path from a safe slot-0 context**, retaining
the authoritative PACKED source and all private context while
it genuinely frees the temporary PROCESS allocation. Only
after it proves a valid original single-owner state should it
release TRANSITION and allow the normal process-owned return,
fatal event and VM/finalizer paths. Do not bypass the foreign
event TRANSITION guard, forge FINALIZE or free an unverified
private executive stack. This recovery, fatal delivery and
subsequent original parent WAIT/HALT remain OPEN.

Versioned tests:
`daimos-testkit/tests/validation/pure-k05-transition-term-v1901/`.
Production design analysis:
`DAIMOS/design/kcore-k05-transition-term-v1901.md`.
Accepted physical evidence:
`$TMPDIR/daimos-k05-transition-term-v1901-physical-gated/`.
