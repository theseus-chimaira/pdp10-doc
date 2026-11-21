# K03 requester-only OOM physical refusal acceptance v1739

Date: 2026-11-21. This supplements the already accepted F12/F13
production requester-only BRK OOM policy (v1298), which is ON.
It does not introduce a new victim-selection policy.

Three independent 256K PDP-6 KCC guests passed against DAIMOS
`2eb0b2ee` with the installed KCC v1735 and repaired matching host
DAS/DAS2 phase executables:

* RUN-parent: real MM descriptor exhaustion returned cause `-3`,
  retained no failed-RUN child, and preserved the independent parent
  and its WAIT contract. Physical requester OOM recovered MM accounting.
* Held provider: a real in-flight foreign filesystem lease and
  pre-dispatch waiter survived requester OOM. Seven busy owner retries
  preceded release; the independent spinner and MM accounting progressed.
* Permanent refusal: a real wrong-owner MM `-4` preserved the fatal
  owner and u-area, with seven invalid attempts, six bounded reaper
  attempts, unreaped WAIT, and progress by the independent foreign
  filesystem owner and bystander.

The three runs are separate physical guests, not a combined scenario.
Their transcript SHA256 digests, exact counters and repository
provenance are retained in `daimos-testkit/tests/validation/
daimos-k03-physical-accept-v1739.md` (testkit commit `c168c01`).
The source owner contract v1731, transitive release graph v1732,
and production policy matrix v1283 also pass.

The permanent wrong-owner result intentionally does not promise
automatic reclamation of unreleasable provider resources; that is
K04 policy. This acceptance also does not close the 32K RAM budget
or the separate default D6FS-root interactive login issue.

## Post-PI6 allocator fix verification (v1743)

The full three-guest K03 matrix was rerun against DAIMOS `03dcb0e9`
after the PI6-only MM allocation change. All three gates passed again.
The complete K01 native suite passed 33/33 tests, including the new
PI6-only allocator test. The previously blocked full 256K D6FS ROOT
login/DSH/TRUE/HALT/CLEAN guest also passed; see
`MM-PI6-ALLOCATION-DCT-V1742.MD`. Exact post-change physical
transcript SHA256 digests are in daimos-testkit report
`daimos-k03-post-pi6-accept-v1743.md` (commit `ea27f7f`).
The older root-login blocker statement above describes the original
v1739 state, not the current result. K04 and strict 32K fit remain open.
