# DAIMOS swapped MonitorFS native acceptance — v562 (2026-10-28)

## Scope

The existing MonitorFS process-file reader supports NAME, CMDLINE and
ENVIRONMENT from a swapped process's BACKSTORE snapshot. This acceptance
covers a *real* process swap, readback, wake, user continuation, child exit,
and parent WAIT. It does not infer correctness from a stubbed reader.

## Test

- Repository: `daimos-testkit`
- Test: `tests/validation/daimos-monitorfs-swapped-resume-v562.{c,sh}`
- Source provenance: recovered the original v74 fixture from testkit Git
  commit `844e497`, with versioned fixture names; the same metadata and
  wake/reap checks are retained.
- Target: fresh 48K PDP-6 SIMH lowmem disk-root image, GCC C99 guest.
- Outcome: PASS, full guest trace `rapbppSMXVC2GABDWP`, normal HALT.
- Testkit commit: `8d95961`.

## Interpretation

The v74 result (2026-10-20) previously demonstrated correct swapped
NAME/CMDLINE/ENVIRONMENT readback, but the resumed reader's WAIT did not
complete before timeout. Against the current kernel, v562 successfully
completes both phases. The historical stall is no longer reproducible
with this fixture. This result does **not** establish which intervening
change eliminated the earlier failure. No new kernel code, permanent
per-process metadata, or RAM overhead was added for v562.

## Remaining gates

The independent GCC multiprocess scheduler/preemption checkpoint failure
and GCC/KCC grouped acceptance remain open. This fixture does not replace
those tests, nor does it close the full P0 release checklist.
