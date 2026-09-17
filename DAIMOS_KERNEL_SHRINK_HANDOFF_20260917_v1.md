# DAIMOS Kernel Shrink and Storage Work Handoff

Date: 2026-09-17
Revision: v1
Status: freeze after kernel-size investigation; accepted and WIP patches packaged separately

## 1. Purpose

This handoff freezes the work completed during the 2026-09-17 kernel-size
sweep and records work that was investigated but is not present in the final
source trees available to this session.

The primary optimization goal remains permanent RAM consumption.  Execution
speed is the secondary goal.  Disk size is secondary to both.

The kernel/runtime acceptance boundary for this work is the `LOGIN:` prompt.
DSH reachability is intentionally not part of the kernel/storage smoke gate.

## 2. Canonical input commits

The uploaded archives used to reconstruct and verify this freeze contain:

```
DAIMOS          b14cf165aa9bdd87cdfbe03165458d95c9a9f8af
                WIP: integrate PDP-6 Type 167/236 drum

daimos-testkit  f57aa900c79d407580727ca9fddd6d96854908f2
                WIP: add DAIMOS DRM236 integration tests

KCC             67d983f...
                optimizer: preserve conditional zero return copy

pdp10-doc       bffae6af717268b927134d1da4a7c5d55cce9432
                docs: record completed userspace bootstrap gate
```

## 3. Accepted KCC fix

Bare `-O` had acquired conflicting driver semantics: it consumed the following
argument as an output filename.  In a command such as:

```
kcc -O -x=pdp6 ...
```

`-x=pdp6` was consumed as the output filename.  Code generation then proceeded
with the default later PDP-10 target and could emit `ADJSP`, which is illegal on
the PDP-6.  During the shrink sweep this manifested as KINIT entering an
illegal instruction and eventually returning to the low-core paper-tape
read-in loop.

The accepted KCC patch restores the documented semantics:

- bare `-O` means enable all KCC optimizations;
- `-O0`, `-O1`, `-O2`, and `-Os` keep their driver mappings;
- `-o FILE` remains the compiler-driver output-name option;
- `-O` no longer consumes the following command-line argument.

Verified with a PDP-6 compilation using `-O -x=pdp6 -m=gas`: no `ADJSP` is
emitted, and `-o FILE` still names the assembler output correctly.

## 4. Accepted DAIMOS shrink/correctness patch

The accepted DAIMOS patch is based directly on `b14cf16` and contains four
independent corrections/optimizations.

### 4.1 DRM236-exposed fixed-capacity defects

DRM236 made the full boot configuration exceed two permanent fixed tables.
The old limits were not sufficient for the configured module set:

```
PDP10_PI_HANDLER_CAPACITY  11 -> 12
MODULE_RUNTIME_MAX         19 -> 20
```

Without these corrections the boot reached D6FS and then reported `SLV FAIL`.
The corrected configuration reaches `SLV OK`.

### 4.2 Correct permanent-memory accounting

`report-permanent.sh` did not include the new DRM MRES.  The accepted patch
adds the `drm236_io` image to the permanent-memory report.  All numbers below
therefore include DRM236.

### 4.3 Use KCC `-O` for the RAM-first kernel build

For this kernel, KCC `-O2` was measured larger than the normal KCC optimized
mode.  Once the KCC option-parser bug above was fixed, the DAIMOS kernel build
was changed from:

```
-O2 -x=pdp6
```

to:

```
-O -x=pdp6
```

This is a size choice, not a workaround for PDP-6 code generation.

### 4.4 Compact multi-terminal process helpers

Five TTY data-path helpers were moved from KCC-generated PDP-10 code to compact
PDP-10 assembly while their C implementations remain available to host tests:

```
proc_tty_read_enter
proc_tty_input
proc_tty_output
proc_tty_pending_take
proc_tty_pending_store
```

The assembly preserves the existing process/u-area and TTY-record formats and
does not add permanent data.

The reconstructed accepted image reports:

```
KCORE+MRES      14579 words
PERMANENT_LAST  14626 decimal
```

The production PDP-6 boot was verified through:

```
DRM236  OK
D6FS    LOADED
SLV     OK
INIT V1
LOGIN:
```

The exact count differs slightly from earlier transient measurements because
this freeze was rebuilt from the uploaded archives and corrected toolchain;
the important property is that the report now includes DRM236 and the image
reaches the LOGIN acceptance boundary.

## 5. Accepted daimos-testkit patch

The testkit patch changes the production kernel smoke test to require:

```
DAIMON ...
LOGIN:
```

rather than `DSH V1`.  DSH execution is a userspace test and must not make a
kernel/storage optimization patch fail after the kernel has successfully
booted INIT/LOGIN.

The patch also adds a small DRM236 capacity regression that requires at least:

```
PI handlers   12
MRES owners   20
```

Both this capacity test and the production `kernel-runtime` LOGIN smoke test
pass against the accepted DAIMOS patch.

## 6. WIP permission shrink snapshot

A second DAIMOS patch is packaged explicitly as WIP and is based on the
accepted DAIMOS shrink patch.

It replaces only the PDP-10 `file_check_access()` wrapper with compact assembly.
The permission-selection policy remains in C in `file_access_stat()`.  A
matching testkit WIP patch adjusts the standalone FILE target fixture so the
new target wrapper can link.

Current reconstructed WIP measurements:

```
accepted image:        14579 words
permission WIP image:  14575 words
change:                   -4 words
```

The WIP state:

- assembles and links;
- passes the standalone PDP-6 FILE target test;
- passes the existing credential/credential-filesystem checks;
- reaches `LOGIN:`;
- does not yet convert `file_check_owner()`.

This WIP should not be treated as the completed permission optimization.  An
earlier transient implementation had measured a substantially larger saving,
but that exact worktree was lost and is not reproducible from the surviving
artifacts.  The packaged WIP is therefore only a safe restart point.

## 7. Work deliberately omitted from patches

### 7.1 Earlier DRUMSET / second-D6FS implementation

A transient worktree earlier in the session implemented and tested substantial
storage work, but that worktree was lost when the execution environment was
reset.  No trustworthy byte-for-byte diff survived, so no patch is fabricated
for it.

The work that had been demonstrated before the reset included:

- a small linear four-drum DRUMSET block provider;
- reuse of D6FS rather than inventing a drum-specific filesystem;
- a second D6FS instance mounted from a configured DRUMSET region;
- correction of SIMH drum attachment so preformatted media were not recreated
  with `attach -n`;
- investigation of two bounded D6FS instance states;
- initial provider-neutral swap/log interfaces;
- explicit administrator configuration rather than automatic "prefer drum"
  policy.

The agreed storage direction remains:

```
DISKSET
    `-- D6FS instance (may have no swap/log reservation)

DRUMSET
    |-- optional administrator-selected raw swap region
    |-- optional administrator-selected raw log region
    `-- D6FS instance on the remaining configured region
```

There is no planned L2 drum cache, no special TEMP arena, and no complicated
memory-management policy associated with the drums.  Swap/log placement is an
administrator decision; the kernel must not automatically prefer the drum.

This work must be reimplemented from the design documents and current accepted
kernel base rather than reconstructed from memory.

### 7.2 Provider-neutral swap/log completion

The transient tree had reached focused provider tests, but production swap and
LOGSTORE conversion was not preserved.  Reimplement this only after DRUMSET
and the second D6FS instance are restored and accepted.

### 7.3 Permission owner check

The next shrink step was conversion of `file_check_owner()`.  Work had begun,
but the exact implementation was not preserved.  The first standalone harness
failure encountered there was a duplicate test stub, not evidence that the
production idea was invalid.  Restart from the packaged `file_check_access`
WIP patch.

### 7.4 Remaining history-driven kernel shrink

Using `6f12dda` (`kernel: compact KCC common paths`) as the last deliberate
shrink checkpoint, the remaining growth was concentrated in recent
credentials/file paths, process/userspace bootstrap additions, and newer
terminal/device support.  Further optimization should continue from linked
word counts, not source-line counts.

Do not reintroduce the rejected direct permission rewrite that caused a mount
regression during the earlier sweep.  Preserve permission semantics in C or
prove any target implementation with filesystem credential tests before
acceptance.

## 8. Known test issue outside these patches

During the earlier sweep the standalone `tty-session-state` harness failed in
both the shrink tree and its control tree.  It was therefore not attributed to
the accepted TTY shrink.  This should be rechecked independently rather than
used to reject the accepted patch without comparison to its base.

## 9. Patch application order

Each SHAR performs its Git commit as its final action.

Apply repository-local patches in this order:

```
KCC:
  kcc-pdp6-O-driver-fix-20260917-v1.shar

DAIMOS:
  daimos-kernel-shrink-20260917-v1.shar
  daimos-file-access-shrink-wip-20260917-v1.shar       # optional WIP

daimos-testkit:
  daimos-testkit-login-kernel-gate-20260917-v1.shar
  daimos-testkit-file-access-wip-20260917-v1.shar      # with DAIMOS WIP

pdp10-doc:
  pdp10-doc-kernel-shrink-handoff-20260917-v1.shar
```

The WIP patches are based on the accepted patch state in their respective
repositories and should not be applied directly to the original archive base.

## 10. Recommended next steps

1. Keep the accepted KCC, DAIMOS, and testkit patches as the new base.
2. Decide whether the small reconstructed file-access WIP is worth retaining;
   it currently saves only four words.
3. Resume history-driven shrink work on the largest recent KCORE objects.
4. Reimplement DRUMSET as a deliberately small block provider.
5. Reintroduce the second D6FS instance without changing the D6FS disk format.
6. Add explicitly configured raw block-provider bindings for swap and log.
7. Recheck the 32K/low-memory boot acceptance after every storage phase.
8. Update storage architecture documentation when the reconstructed
   implementation again reaches acceptance.
