# PDP-6 K05 v1912: late foreign-owner repair and genuine last-user HALT

Date: 2026-11-24. **Scoped original 48K PDP-6 physical PASS.**

## Lifecycle boundary

The v1906-v1911 K05 tests show why an uncertain FULL target
must retain a private executive continuation under TRANSITION,
including after parent EXIT and ordinary sibling cleanup. The
last-user v1911 result was intentionally operator-stopped with
the quarantined orphan live after three real slot0 retries
and a genuine PCLK PI6. The outstanding question tested in
v1912 is whether the ORIGINAL kernel can then safely reach
normal terminal shutdown if the exact original foreign-owner
uncertainty is eventually resolved, with no forced FINALIZE,
synthetic syscall return, or discarded live executive stack.

v1912 reuses byte-identical v1910/v1911 five-child compressed
TSFS guest, original 48K KCORE and optional PURE MRES. Initial
native PACKED source `mm_free` genuinely returns -5. The
original provisional PROCESS target is test-retagged owner3
-> foreign owner2, original TERM(3) is genuinely refused, and
the parent/other healthy children really EXIT. Only slot3
remains live; MM10, three capped recovery retries each `01000`
octal PCLK ticks apart, and a real idle-loop PI6 are observed.
The entire pre-repair portion PASSes the independent v1911
source-linked positive physical oracle.

## Physical MM metadata compaction

The original test fault was injected at MM owner word
`024725`. By the time the normal children EXIT, the original
MM extent array has compacted. The exact still-live original
temporary full PROCESS target `120000..130000` owner2, pin0,
is now active at **MM owner word `024707`**, with authoritative
PACKED RAW `111200..120000` still owned by slot3. The controller
derives this new index by exact geometry/type/owner/pin from
the independently accepted original MM10 v1911 physical
table, then confirms owner2 immediately before the correction.

The valid v1912/v3 controller performs only one test-only
deposit `024707` owner2 -> owner3. This merely reverses the
artificial fault, and is NOT an automatic kernel repair path.
An initial v1912/v2 controller erroneously reused old
`024725` after compaction; that word was then an inactive
owner4 field. The invalid simulation was terminated and its
evidence rejected. The addressing error was fixed in the
test controller, with no production kernel source change.

## Original recovery and final halt

| Genuine PDP-6 event | Observed physical result |
| --- | --- |
| Original native `mm_free(PROCESS)` | Call `033420`, return `033421` **AC1=0**, exact provisional target freed, MM10 -> MM9 |
| Original TRANSITION clear | Code `033456` runs after free; escrow u[067] `040000120000` -> zero, RAW source retained |
| Second original FULL source free | Native PACKED source `mm_free` return at `036175` **AC1=0**; RAW `111200..120000` freed |
| Second FULL completed | Original service return `033737` **AC1=0**, slot3 `047017400001` RESIDENT, VMbase `130000`, TRANSITION clear, MM9 |
| Genuine final user EXIT | Child emits `w` following its pipe read, enters `proc_exit_current=013655` with current slot3 |
| Original last-user HALT | Actual hardware HALT PC `000000`, final MM6, high slot1, slot3 descriptor FREE, no residual slot3 PROCESS/PACKED MM owner |

At every observed stage, ROOT remains original and no
unrelated memory is reclaimed. Original finalization is
performed by KCORE on the permanent slot0 stack after the
restored child invokes its own genuine user-mode EXIT.
This is the original PDP-6 HALT instruction, not an operator
EXIT or fabricated monitor message.

The strict physical audit checks original KCORE/PURE hashes,
the unique live target after MM compaction, exact MM/ROOT
ownership across target free and second FULL, actual native
AC1 results, escrow retirement only AFTER lawful target free,
the final child EXIT and true last-user HALT. FOUR narrowly
targeted negative physical mutants all reject: a failed
owner repair disguised as successful, a forged native
target-free reply, premature TRANSITION clearance and an
incorrect final MM count. The entire pre-repair v1911
positive oracle also re-PASSes on the same physical run.

Accepted physical normalized SHA256:
`d4e4c884b00eb61f455587e60cd8d19033dfbcb220160bf21f4584a51045de96`.
Source-linked simulator controller SHA256:
`0ab39813f75552eb623171899ca56b64dbe10d979ce12d40d407da01fec1b26e`.
Fixed original KCORE SHA256:
`a201a7fdcf481b436b06694e5dd55c9367036ebd9f64c630486f854b1a4f89be`.
Optional PURE SHA256:
`09f65d0ca272986b462c63580eae3c4006490004cd060529ce40f10a55949351`.

Testkit:
`daimos-testkit/tests/validation/pure-k05-terminal-reconcile-v1912/`.
DAIMOS design: `DAIMOS/design/kcore-k05-terminal-reconcile-v1912.md`.
Original accepted physical evidence:
`$TMPDIR/daimos-k05-terminal-reconcile-v1912-physical-v3/`.

**No production C/ASM, fixed KCORE/PURE IMAGE, BSS, RAM, process
state, or feature defaults changed.** K05 remains OPEN for
permanently unknown-owner cleanup policy, other MM/PI6 and
allocator failures, and broader platform release. This proof
requires a trustworthy later ownership reconciliation; it
does NOT authorize speculative owner rewriting or destruction
of an irrecoverable child's private executive continuation.
K06-K09 OPEN; RAW PURE and mode3 remain default OFF.
