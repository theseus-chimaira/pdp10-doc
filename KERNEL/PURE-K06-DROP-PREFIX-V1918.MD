# PDP-6 RAW PURE synchronous DROP/PREFIX ownership (v1918)

The original aligned RAW suffix identity is saved in existing u-area
word 067 before the first pin. A reported MM result alone does not
prove that rebase, claim, release or commit succeeded or failed.

DROP reconciles against the original PROCESS and PACKED shapes.
PREFIX revalidates the exact full-span PACKED owner before commit
and the exact RAW suffix before retiring the transaction pin.
Only a proven committed PROCESS owner may publish a runnable APR.
A shared PI-excluded sole-owner scan rejects invalid extent geometry,
capacity overflow, duplicate user ownership and foreign physical
intersection. Nonoverlapping MODULE and KERNEL_DYNAMIC extents have
independent owner namespaces. No new permanent RAM is required.

The v1918 source-linked PDP-6 gate passes six transaction fault cases
and seven scanner geometry cases, with the actual rebase, claim,
release and commit assembly and controlled pin, allocation, PI and
loader fixtures. Two unsafe production-source variants fail native
assertions at 0101 (reply-dependent DROP) and 0403 (foreign unpin).
The v1612 service regression and its unsafe READY_ONCE control pass.

This is synchronous validation, not K06 completion or physical PI6
acceptance. Deferred DROP/PREFIX recovery, terminal ownership cleanup
and final integrated MM geometry review remain separate. RAW PURE and
mode3 remain disabled by default.
