# KCORE K01-K04 closed: final policy and acceptance v1803

Date: 2026-11-22. Status: **K01, K02, K03 and K04 CLOSED**
for their documented implementation and release scopes.

K01 v1718 closes permanent MM failure ownership and checked
retry. K02 v1730 closes process-owned resource cleanup across
EXIT, EXEC and failed RUN. K04 v1801 closes teardown recovery
under its bounded physical acceptance; see
`KERNEL/KCORE-K04-CLOSURE-V1801.MD`.

K03 v1803 closes **by explicit architectural policy**. The
requester-only BRK physical-NOMEM termination already enabled
in production (`PROC_OOM_BRK_ENABLED_V1000,1`) remains ON.
F12/F13 v1298 accepted this behavior for the defined 256K
PDP-6 profile with 64/64 mandatory gates and independent
Android/Linux media evidence; see
`KERNEL/OOM-F13-PRODUCTION-ACCEPTANCE-V1298.MD`. The v1739/
v1743 physical refusal checks and v1802 source guard further
cover owner retention and cause discrimination. Existing
swapping is preferred over killing another process to recover
resident memory.

**No cross-process OOM killer will be implemented** under K03.
There is no outstanding victim-selection task: it is rejected,
not deferred. Killing a different process cannot guarantee
reclaim when a resource is pinned or a provider permanently
refuses cleanup. The process swap-out selector is not an OOM
victim killer. A future policy reversal requires a separately
approved feature proposal and its own safety acceptance.

K04 continues to quarantine unreleasable owners rather than
freeing unproven memory. Bounded retry *rate* is not bounded
time to recover. Untested two-owner clock wrap, published
pinned session-HUP members and organic media/controller errors
remain explicit residual coverage risks, not claimed passes.

The v1803 work changes documentation and backlog status only.
Production KCORE text, DATA, BSS, ABI and permanent RAM have
**zero delta**. Earlier dated K03/K04 OPEN statements are
historical evidence and are superseded by this closure.
DAIMOS `design/kcore-k01-k04-closure-v1803.md` is the primary
cross-item closure record. The remaining KCORE feature backlog
starts with K05; the strict 32K size target is separate.
