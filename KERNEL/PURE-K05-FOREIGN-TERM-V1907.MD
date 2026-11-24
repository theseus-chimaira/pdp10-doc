# PDP-6 K05 v1907: native TERM refused during foreign FULL ownership

Date: 2026-11-24. **Scoped original 48K PDP-6 physical PASS.**

The v1906 exact-target escrow prevents a failed FULL owner from
becoming runnable when its still-live temporary PROCESS target
is incorrectly claimed by a foreign process. The separate
v1901 original physical test proved user TERM rejection on a
TRANSITION-held saved executive stack. v1907 directly tests
this combined condition, avoiding a redundant broad test matrix.

With unchanged production KCORE and optional PURE MRES, the
source-built original compressed TSFS five-child workload
first executes real `mm_free(PACKED)` returning -5. The test
controller releases the original source pin, then temporarily
changes only the full PROCESS target metadata `024725` from
owner3 to owner2. Three parent HBs preserve the MM18 RAW source,
foreign-claimed target, TRANSITION/VMbase0, original saved
CTX_PC/FDCTL and escrow `040000120000` in child u-area word 067.

Immediately after the first foreign-owner heartbeat, the
controller opens ONE test-only parent user-space gate at
physical `135361`. The original parent issues a genuine
`SYS_PROCCTL_EVENT_PID` with `SYS_EVENT_TERM` for slot3, and
prints **`TR`** (refused). The exact physical snapshot at this
point confirms no MM free, FINALIZE, FILE change, escrow loss,
TRANSITION unlock, unexpected APR or premature parent WAIT.
The original parent WAIT release gate at `135357` remains zero.

After two further safe parent heartbeats, the test controller
repairs only its own injected MM target owner. The original
slot-0 native target free, subsequent FULL source free and
RESIDENT publication all succeed. Original five child EXITs,
typed parent WAITs, Z and actual PDP-6 HALT PC0 PASS. Final
MM count is 6, no residual child owner; selected ROOT unchanged.

The full v1906 physical validator re-PASSes. Exactly **four**
targeted additional corruptions (false TERM success, early
TERM gate, missing foreign target, and lost escrow) are
independently rejected. The existing 16 v1906 MM mutants
need not be repeated for this unchanged production binary.

No production source, KCORE image, BSS, per-process field,
PURE image size, or default policy changed:

- KCORE SHA256 `a201a7fdcf481b436b06694e5dd55c9367036ebd9f64c630486f854b1a4f89be`
- PURE SHA256 `09f65d0ca272986b462c63580eae3c4006490004cd060529ce40f10a55949351`
- Accepted original v1907 physical SHA256 `da5ae4d1920279f78965080297985b5e698f1ae6ea3390b573a5edb34b26c7dc`
- KCORE IMAGE `024167` / BSS `000415`; PURE IMAGE `004761` octal

Testkit:
`daimos-testkit/tests/validation/pure-k05-foreign-term-v1907/`.
Implementation analysis:
`DAIMOS/design/kcore-k05-foreign-term-v1907.md`.
Accepted physical evidence:
`$TMPDIR/daimos-k05-foreign-term-v1907-physical-v1/`.

**K05 remains OPEN**: this proves that ordinary remote TERM
cannot discard an original live executive continuation while
the temporary target is foreign-claimed. It does NOT prove a
way to terminate an irrecoverably pinned or foreign MM owner.
Never bypass TRANSITION or invent FINALIZE to force cleanup.
Other PI6/allocator cases, K06-K09 remain open; RAW PURE and
mode3 remain default OFF.
