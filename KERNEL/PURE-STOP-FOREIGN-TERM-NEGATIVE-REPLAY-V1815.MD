# Physical STOP/PACKED foreign-TERM evidence replay, v1815

Date: 2026-11-22. This is a completed test-verifier consolidation,
NOT a new physical PDP-6 execution or PURE production acceptance.

## Source and scope

The old daimos-testkit `test-pure-stop-foreign-v1682-20261119`
worktree retained one changed test oracle and a new physical negative
control. Both were compared with the currently integrated source and
promoted to daimos-testkit main in v1815.

The source-linked v1682 oracle checks the original saved USER PC,
STOPPED scheduler state, absent private WAIT, RAW PACKED owner, and
the original v1673 A/B/C MM/proc/memory conservation. It now also
requires a single observed native `V1682 APR WORDS 3072` growth event,
not merely successful STOP/TERM text in a terminal transcript.

Five negative controls mutate copied original physical evidence:
saved USER-PC flag, scheduler EXEC/wait bit, STOPPED scheduler state,
the foreign TERM acceptance marker, and the APR growth event. The
first three operate on original B-checkpoint word addresses obtained
from the matching KCORE and fixture maps. Each modified original must
be rejected by the unchanged physical and source-context oracles.

## Replayed evidence

The preserved 48K GENERAL and STRICT v1682 original console traces
both passed the v1673 conservation oracle and v1682 STOP/foreign-TERM
source checks. All five deliberately invalid source witnesses failed
for each profile: ten negative rejections in total.

The reproducible script is
`daimos-testkit/tests/validation/pure-stop-foreign-v1682/replay-negatives-v1815.sh`.
It requires `V1815_EVIDENCE_ROOT` to point to the original source-
authentic v1682 console and map artifacts, and does not regenerate or
edit the accepted physical recordings. Test outputs are separate.

## Preservation and limits

Before resetting the old two-file dirty test state, both source files
were byte-compared against main and preserved, with a binary-capable
Git patch and SHA256 manifest, in
`~/backups/daimos-v1682-dirty-promotion-v1815-20261122/`.
Only those two promoted changes were cleaned; the historical branch
and attached worktree remain.

This evidence replay does not close K05-K09: real new PI6 pressure,
terminal MM ownership, root matrices, source failure and production
enablement are separate outstanding gates. RAW PURE stays default OFF.
