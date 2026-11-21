# KCC pure outgoing stack-argument optimization v1735

Date: 2026-11-21.  Applies to the native KCC GCC-compatible PDP-10
calling convention, not to the DAIMOS kernel or its process ABI.

## Motivation and measured result

The correct v1410 call generator reserves outgoing stack-only argument
slots before evaluating arguments.  This prevents nested calls and
temporary register spills from overwriting arguments.  However, the
reservation and subsequent indexed MOVEM sequence cost additional
instructions even when all arguments are pure and the stack suffix
can be pushed directly in reverse order.

The v1735 change in KCC `ccgen2.c` selects direct PDP-10 PUSH only
when all arguments are free of side effects and none of the stack-only
arguments is an incoming register identifier.  Calls with nested
evaluation, side effects, or stack-passed incoming register identifiers
retain the v1410 reserved-slot path and its lifetime guarantees.
Both paths preserve the exact stack-word count and caller cleanup.
There is no heap allocation or new static data.

DAIMOS DSH was rebuilt using the installed v1735 compiler, with the
relocation-safe `p10fold -m 2` minimum already used by KCORE.

| Configuration | DSH DXR image (octal) | BSS (octal) |
| --- | ---: | ---: |
| Previous KCC, fold 4 | `036210` | `001763` |
| Previous KCC, fold 2 | `036154` | `001763` |
| KCC v1735, fold 4 | `036005` | `001763` |
| KCC v1735, fold 2 | `035751` | `001763` |
| Strict EXEC image limit | `036000` | -- |

The final image is 23 decimal words under the EXEC limit and 159
decimal words smaller than the previous KCC/fold-4 baseline.  The
image improvement is a toolchain and userland-link change, not a
KCORE size reduction.  The previous compiler fails the same strict
image test, providing a negative control.

## Acceptance and limits

`pdp10-testkit/misc/compat/test-kcc-stack-push-v1735.sh` exercises
five- and six-argument calls, stack-passed values, nested-call
fallback and stack-pointer balance on the simulated PDP-10.  The
versioned mixed-call-depth and direct KCC/GCC ABI runners also pass.
`daimos-testkit/tests/validation/daimos-dsh-kcc-image-v1735.sh`
builds the actual KCC DSH image and checks the two-word fold floor,
the 036000 text limit and the unchanged 001763 BSS ceiling.
The DSH resource contract and command-length checks also pass.

The separate full 256K PDP-6 D6FS-root login/DSH run was attempted
with the installed compiler and correctly sized DSH image.  It did
not reach DSH: the guest printed INIT V1 and LOGIN, then halted at
PC 004270.  The KCORE map and `proc_storage_runtime_v30.s` identify
this PC as `kcore_critical_media_root_halt_v1515`, the explicit
fatal root-I/O failure path.  A second run using the DCS0 terminal
instead of the older CTY-only harness reached LOGIN and halted at
the same PC.  Preflight D6FS was CLEAN; the halted writable copy
was DIRTY afterward.  This is an unresolved root-I/O acceptance
blocker, not evidence of a DSH instruction-size or ABI failure.
The tests above establish only the stated ABI and image gates; they
do not establish a complete 256K login/DSH boot or 32K low-memory
acceptance.

KCC commit: `1dca382`.  DAIMOS commit: `2eb0b2ee`.
PDP10 testkit: `5111ba4`, `0ce92f1`.  DAIMOS testkit: `58b9e7e`.
