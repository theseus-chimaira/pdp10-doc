# DAIMOS KCORE v1517: DECtape selection error is not tape-loss detection

Date: 2026-11-12. New executed physical-controller-simulation evidence;
no production KCORE change and no claim of real Type-551 media
removal detection. Supersedes earlier unsupported suggestions that
`DTB_ILL` from PDP-6 SIMH establishes permanent root-tape removal.

## Executed PDP-6 SIMH Type-551 status experiment

Testkit `daimos-dtc-select-status-v1517.sh` deposits and executes
real PDP-6 `MOVEI`, `CONO 0210`, `CONI 0214`, `MOVEM` and HALT
instructions, rather than mock controller functions. A legal
SEARCH/SELECT instruction targets DTC0 before attachment, after
attachment, and again after `DETACH DTC0`. The same instruction
is also sent to unrelated unattached DTC7.

| DTC transport | Raw CONI status (octal) | Observed |
| --- | --- | --- |
| 0, unattached | `000004` | Select/illegal error |
| 0, attached | `001000` | Request, no select error |
| 0, detached again | `000004` | Select/illegal error |
| 7, unattached and unrelated | `000004` | Identical error |

All four real CPU HALTs at PC `001004` and all four stored
controller status words are verified. Both local source-built
SIMH and the installed simulator PASS. Testkit revision `3536d98`.

SIMH's implementation `sims/PDP10/pdp6_dtc.c` explicitly sets
`DTB_ILL` when a selected unit has no `UNIT_ATT` backing file.
That is a simulator-specific behavior, not proof that original
Type-551 hardware detects physical reel removal in the same way.
The Type-551/TU55 uses a shared Type-136 data channel; normal
read errors and bounded searches/transfer watchdogs remain
different fault categories. Historical TU55/TU56 transport manuals
describe OFF/LOCAL/REMOTE switching and warn that the later
analog SELECT ECHO transport detector is not used with 550/551/552
controls. `DTB_ILL` is a **select/illegal condition**, not a
dedicated readable *tape-changed* or *reel-unmounted* register.

## Production policy and remaining P0

Do **not** turn a single `DTB_ILL`, negative Type-136 I/O result,
or bounded tape watchdog expiry directly into a fatal KCORE root
HALT. Even an authoritative offline event must be related to an
actual root member through the accepted v1516 physical-unit
membership mask. Optional non-root DTFS loss must be nonfatal;
it retains mounts and open ownership until safe revalidation.

The minimal v1515 critical-device HALT leaf is already implemented
and native-tested. The v1516 exact TSFS root-set predicate is
already implemented and native-tested. What remains OPEN is a
trustworthy **hardware-compatible loss event**, its connection
to those two helpers, and actual guest root/auxiliary-media
removal acceptance. A transient timeout cannot stand in for
device-loss evidence, and no retry threshold is added.

Source references:

- `DAIMOS/design/kcore-dtc-select-status-v1517.md`
- `daimos-testkit/tests/validation/daimos-dtc-select-status-v1517.md`
- `https://bitsavers.trailing-edge.com/pdf/dec/dectape/tu55/H-TU55_MaintMan_Nov66.pdf`
- `https://bitsavers.org/pdf/dec/dectape/tu56/DEC-00-HRTC-D_TU56_Mantenance_Manual_Jan73.pdf`

The 32K strict KCORE size issue from v1516 remains a separate
unresolved integration gate; v1517 adds zero instructions and
zero permanent data words.
