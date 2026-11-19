# DAIMOS KINIT PDP-6 stack-codegen build invariant v1

Date: 2026-11-19. Completed build dependency and ownership correction.

The production KINIT Makefile previously declared its
`.kinit-stack-v655-*` target before defining the default value of
`KINIT_STACK_WATERMARK`. The target was expanded with an empty
watermark, while the actual KINIT dependency later requested the
`-0` suffix. GNU make could not construct that prerequisite.

DAIMOS `01e58f2` defines the watermark default before constructing
the stamp target. The actual kernel-source compilation and its two
original PDP-6 output assertions pass: no unsupported `ADJSP`, and
a paired-halfword simulated stack adjustment. This is not an assertion
that every KINIT compilation mode or boot succeeds.

The test inspects the production DAIMOS kernel source and thus belongs
in `daimos-testkit`, reached by `make test-daimos-kcc-pdp6-codegen`.
DAIMOS-testkit `d6e864f` owns that check. `pdp10-testkit` retains
standalone compiler/regression source tests, not kernel-source
integration checks. Historical test reports are preserved as dated
evidence rather than rewritten.
