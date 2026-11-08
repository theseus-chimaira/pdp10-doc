# v1350: staged grouped D6CODEC native PDP-6 baseline

Date: 2026-11-08. Production DAIMOS still uses D6LZ36.

Testkit `239a191` provides a versioned runner that executes the
existing grouped D6CODEC native PDP-6 fixtures in isolated
workdirs, retaining independent logs and a SHA256 manifest.

| Native test stage | Passing cases |
| --- | ---: |
| v045: golden streams, capacity and boundaries | 10 |
| v047: complete application stream corpora | 3 |
| v048: malformed stream / decoder safety | 22 |
| v049: complete DSH stream | 1 |
| v051: complete darc/stack | 1 |
| v055: staged DAIMOS kernel ABI wrapper | 1 |
| **Total** | **38** |

All 38 pass. This establishes a reproducible staging baseline
but is **not** the production codec cutover. The active Stage-1,
KINIT, EXEC, VFS, PURE backing, TSFS, host producer and stored
payloads still use D6LZ36. Full grouped format migration must
be atomic; no compatibility guessing, no mixed-format images,
and no partially linked KCORE change are authorized by this test.

The remaining integration requirements are in
`DAIMOS/system/kernel/core/D6CODEC-INTEGRATION-TODO-v054.md`:
canonical exact stream behavior, native/host parity, Stage-1
placement and boot devices, consumer/producers, error rollback,
linked 36-bit-word code+stack+workspace RAM measurements and full
system acceptance. The two staged v053 assembler sources are still
inactive. No permanent KCORE word-size or performance benefit has
been claimed by this baseline.

Evidence on the Termux host:
`$TMPDIR/daimos-d6codec-native-baseline-v1350-evidence-r1.tar.gz`.

SHA256:
`a1052dfa875e91213faf39da2643c9b87721492d2d365c281f98ddfe13ce5ea3`.
