# PDP-6 PURE MM ownership: v1658 fail-closed commit and quarantine

Date: 2026-11-17. Source base: DAIMOS v1657 2370e0b1.
PURE RAM is opt-in and OFF by default. Strict 32K support remains
deferred until a later KCORE size/boot campaign. This note describes
the actual PDP-6 MM and process service, not a new generic VM layer.

## Problem and correction

The v1654 bounded FULL restore already unpinned its provisional APR
before releasing the authoritative PACKED RAW suffix and publishing
RESIDENT. The fast in-place PREFIX restore still retyped the MM
descriptor, published a RESIDENT runnable APR, and then called the
fallible `mm_unpin(base)`. An unpin refusal after publication left
the process TRANSITION-held with a pinned PROCESS descriptor. There
was no remaining PACKED owner for ordinary retry or safe rescheduling.

In v1658, `mm_pure_commit_prefix_v1384` requires an exactly matching
PACKED_PROCESS owner, full APR size, base/geometry and exactly ONE pin.
Only after validation, one metadata store both retypes the extent to
PROCESS and consumes the verified transaction pin. The caller publishes
VM base, clears saved backing, switches storage mode to RESIDENT and
clears TRANSITION under the same short PI-disabled interval.

No filesystem I/O, allocation or blocking service runs with PI off.
The immutable prefix loader executes while the complete PACKED
reservation and saved mutable suffix remain pinned, nonrunnable and
protected. Failure before commit still releases the prefix reservation
and suffix pin with checked operations. There is no second MM owner
registry or new persistent process bit.

## Permanent refused-owner policy

The process TRANSITION bit is the quarantine marker, not a request for
blind MM repair. The scheduler/RT path, ordinary swap and reclaimer
exclude transitional processes. The PURE service cannot set READY_ONCE
without a completed RESIDENT APR. Ordinary HUP/TERM must not release
an active PURE transaction. The idle finalizer resumes only separately
marked PREVM/post-VM terminal stages; an arbitrary uncertain PURE owner
cannot be mistaken for a finalizable zombie.

On a reported DROP cleanup error, the v1658 MRES checks the existing
MM table. Either an exact singly owned unpinned original RESIDENT APR
or an exact singly owned unpinned PACKED RAW suffix can be unlocked.
In the latter case it makes at most one further compaction check.
Pinned, duplicate, missing, foreign, wrong-sized or wrong-type owners
remain transition-held. The same fail-closed policy applies to
refused FULL source/target unpins, source free and failed rollback:
there is no unsafe descriptor reuse or synthesized process completion.

The original terminal PACKED detach verifies one exact unpinned owner
before MM free. Refused free retains the slot/u-area and retry state;
an impossible post-free storage publication mismatch uses the existing
kernel invariant fail-stop. Permanently corrupt pins cannot be
forcibly recovered safely from the observed MM descriptor alone. An
irrecoverable quarantine requires operator restart, not a scheduler
busy-loop or unrelated-process corruption.

## Native and physical evidence

The new daimos-testkit `tests/validation/pure-mm-owner-v1658/` links
the real PDP-6 assembly and production C transactions and executes:
original v1654 FULL source/target/rollback negatives, original v1612
service, actual successful PREFIX retype with zero remaining pins,
and DROP errors where unpin either completed but returned an error or
genuinely refused. The latter remains pinned and TRANSITION-held.

An independently linked and executed mutant omitting the fast-commit
pin decrement is rejected by the original native descriptor oracle:
the reported native test exit is octal 002002, not the required zero.
Compilation success alone is not counted as negative acceptance.

The new 48K source-built GCC TSFS-root/secondary DTC3 multi-process
guest also completed the original six-child PIPE/WAIT/EXIT sequence
and halted at PC 000000. The complete fixed KCORE stream SHA256 is
`88970ee633dc22094565172348bc1dcb9f49ba0417e104fb5608b71a87668d01`,
identical to the previously accepted v1654 fixed-core stream.

The v1658 physical map gives `__kcore_low_end=024253` octal,
`proc_pure_mres_service_v1612=000363`, `proc_pure_ram_restore_v1391=002216`,
`exec_image_full_bounded_v1391=003243`, and KINIT image end
`113641` octal. The removable PURE DXR image has 003355 octal
loadable words with zero BSS; its record also contains relocation
information. Relative to v1654's 113631 KINIT end this is eight
additional decimal transient boot words, NOT fixed KCORE growth.
Do not interpret KINIT staging as an accepted 32K footprint.

The independent source-linked physical 48K FULL read-loss/recovery
replay now also PASSes with exact v1658 MRES machine words: real DTC3
detachment produces -4, preserves authoritative RAW, releases the
provisional FULL target, then recovers after reattaching the SAME
physical reel. Six WAIT/HALT complete. The original source auditor
rejects both forged retained-pin and forged successful-read witnesses;
the complete v1658 runner exits zero. This does not physically inject
permanently corrupt MM descriptor metadata.

Continuous physical high-water and matched RAM
comparison remain open. v1657's 19,770 words of compressed RAW
potential are data-only and do not establish a mode3 release saving.

## Follow-on work

Finish current-source native/physical terminal ownership fault matrix,
continuous machine RAM accounting, optional bounded disjoint v1451
bitmap encode/decode transaction with real net RAM evaluation, 64K
matched regression and the full PDP-6 functional freeze before any
32K/whole-KCORE minimization campaign.
