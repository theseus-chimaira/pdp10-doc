# DAIMOS kernel implementation ownership (v979)

The PDP-6 DAIMOS kernel has one implementation of each operation. When an
operation is implemented in PDP-6 assembly, do not retain or introduce a
secondary C implementation behind a compile-time macro. Implement a new
operation once; improve or repair its authoritative implementation directly.

## Source cleanup

The v979 cleanup deleted redundant guarded C bodies from:

- `system/kernel/proc/proc.c`: u-area release, TTY session release,
  process slot discard/finalization, child adoption, event handling,
  WAIT status, scheduler selection, and swap victim selection.
- `system/kernel/proc/vm_pdp6_swap.c`: redundant swap transaction,
  transfer, service, and reclamation implementations.
- `system/kernel/fs/file.c`: redundant FILE OPEN implementation.

The corresponding `*_PDP6_ASM` switches in `system/kernel/Makefile` were
removed. Other C functions remain where they provide distinct services,
not parallel versions of assembly routines; in particular,
`proc_exit_finish` invokes the assembler owner finalizer.

## Validation and effect

The full 256K PDP-6 image built and linked. The headless boot smoke test
reached DAIMON V0.5 with MEMFS and D6FS loaded. Source-linked fatal event
precedence and self-delivery tests v971/v974 passed. A broader booted
regression remains a separate validation gate.

This is a source-maintenance cleanup, not a kernel size optimization:
the removed C bodies were already excluded from the production assembly
build by preprocessor defines. Do not attribute linked KCORE size savings
to deletion without a measured before-and-after binary comparison.

New test cases belong in `daimos-testkit`, while this document records
long-lived kernel policy rather than a temporary work handoff.
