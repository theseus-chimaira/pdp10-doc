# PDP-6 K05 v1904: pinned FULL target, idle backoff and final typed WAIT

Date: 2026-11-24. **Scoped source-built original 48K PDP-6 physical
PASS.** This extends v1903's actual FULL commit and five-child
EXIT/WAIT/HALT acceptance with three real parent heartbeats while
the provisional PROCESS target is externally pinned.

## Exact physical MM owners and retry invariants

The original secondary TSFS-backed five-child RUN/PIPE/BRK
workload enters the original FULL and returns genuine native
MM_ERR_BUSY=-5 from `mm_free(PACKED)` while RAW source is pinned.
The source remains authoritative; the provisional full PROCESS
owner is not runnable and slot3 is TRANSITION-held. After the
initial external source pin is released, the physical controller
adds ONE temporary pin to the actual slot3 full PROCESS extent.

Original physical MM metadata entries (two words per extent):

- `024723` RAW source: owner3, PACKED, `111200..120000`;
- `024725` full provisional PROCESS target: owner3, `120000..130000`;
- `024727` unrelated original resident parent: owner1, `130000..140000`.

Only target metadata changes `000001000003` to `000011000003`
while held. The original MM extent count remains 18 at three
successive real resident parent user heartbeats. RAW source pin0,
PROCESS target pin1, slot3 TRANSITION and VMbase0, original backing,
saved private executive CTX_PC/FDCTL, root and unrelated MM owners
remain exactly preserved. Independent PCLK advances `025256`,
`025356`, `025456` octal.

The original single shared post-VM retry word at `024532` retains
the pending bit. Its bounded retry interval progresses `0100`,
`0200`, `0200` octal. The native target-free is not executed with
the target pinned; no uncommitted APR is published. An exploratory
attempt to stop at a folded-code service return did NOT hit an
authoritative return PC, so no claim is made of observing a
particular C return code while the target is pinned.

After removing only that external target pin, the original
native slot0 `mm_free(PROCESS)` executes and returns MM_OK=0 after
`000060` octal additional clock ticks. The temporary full target
is removed, sole unpinned PACKED RAW owner remains and original
TRANSITION clears. Another original FULL then frees the RAW source,
publishes a real RESIDENT VMbase `120000` and returns the original
children to execution. All five original typed EXIT/WAIT results,
parent Z and real PDP-6 HALT PC0 PASS; MM occupancy is 8 at Z,
6 after HALT, no residual slot3 PACKED/PROCESS owner.

## Test/controller provenance

The first two exploratory runs incorrectly selected the unrelated
parent owner at `source_meta+4` rather than the true target at
`source_meta+2`. These are REJECTED, not kernel bug evidence.
An independent original timer diagnostic preserved the valid
experiment idea, proving pending original retry state and exact
owner conservation during three actual user heartbeats.
The final v4 controller uses original current linked addresses,
original MM/pin metadata, and strict source-linked native target
free and eventual FULL/WAIT/HALT checkpoints. All **12** separately
mutated genuine physical transcript controls reject, including a
wrong-parent-owner target, premature release/publication, changed
saved context, lost timer pending bit or stale clock.

The accepted production images are byte-identical to v1903.
KCORE DXR IMAGE `024167` octal, BSS `000415`;
optional PURE MRES IMAGE `004615`. No production source, field,
fixed RAM, or default policy has changed.

- KCORE SHA256 `a201a7fdcf481b436b06694e5dd55c9367036ebd9f64c630486f854b1a4f89be`
- PURE SHA256 `55be5113d4d15fe5c4a44409dca22afd463c75f05f661b3f8f71451a057c577f`
- Accepted original physical SHA256 `f912b6cae1c470a876a404cd2e49af708b727f9bb423fbc2eaefe1d0777f16de`

Testkit:
`daimos-testkit/tests/validation/pure-k05-target-pin-idle-v1904/`.
DAIMOS implementation report:
`DAIMOS/design/kcore-k05-target-pin-idle-v1904.md`.
Accepted local physical evidence:
`$TMPDIR/daimos-k05-target-pin-v1904-physical-v4/`.

**Remaining K05:** continuously inaccessible/foreign MM owners,
allocator/free ambiguity, terminal failed-FULL teardown and other
real PI6 interleavings. The successfully released temporary target
pin does not establish permanent ownership recovery. K05-K09 and
optional mode3 stay OPEN. RAW PURE remains default OFF.
