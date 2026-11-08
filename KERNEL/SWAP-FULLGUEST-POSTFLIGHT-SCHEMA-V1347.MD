# v1347: completed 256K physical no-partner guest; full suite still open

Date: 2026-11-08. Scope: actual PDP-6 256K guest with 24K-word
process images, real DEVICE/DRM storage and test-only physical
MM owner reservations. Production kernel/MM/FS/PI6/SWAP/MRES
sources and resident footprint remain unchanged.

The unbounded v1344 physical guest successfully completed INIT,
all eighteen RUN/ACK admissions and eighteen real TERM/WAIT reaps,
requester/witness `KWP`, HALT and SIMH EXIT. The typed production
MM allocator returned one actual `MM_ERR_NOMEM` for a pending
disk-backed swap-in and slot 0 performed one no-partner cooldown;
the independent resident spinner released four real 8K-word MM
owner extents and its own pin through the original production MM
operations. The guarded independent provider holder completed
and unlocked through the ordinary production FS service path.

The source-linked v1303 postflight physical checker PASSed:

| Observation | Final result |
| --- | ---: |
| Additional children admitted | 18 |
| Additional children reaped | 18 |
| Real swap-in MM allocation failures | 1 |
| No-partner cooldowns | 1 |
| Successful swap-ins | 34 |
| Process exchanges | 0 |
| Outstanding swap blocks | 0 |
| Outstanding backstore blocks | 0 |

The original v1303 decoder incorrectly treated every new
`f07s_diag_*` read-only sampler address as a required after-HALT
postflight field. The v1342 timer frontier, for instance, is
not emitted by the old postflight script. Testkit v1347 `a7454e4`
restricts decoding to v1303's documented fixed required fields.
This is a verifier schema correction, not suppression of a failed
MM/swap assertion. A dedicated regression accepts optional extra
diagnostics while rejecting missing and duplicate required fields.
The v1346 fatal-versus-reclaim refusal and original v1303
regressions remain PASS.

**Full original F07 suite acceptance is still OPEN**: v1150's
strict independent-spinner progress witness requires at least five
real sleeps after held-provider admission, whereas this guest
recorded four. Holder owner, lease, real OOM, release, three
fatal-busy retries and normal return were correct. The test
must synchronize spinner termination after five *actual*
scheduled sleeps rather than relax or forge that progress witness.
The v1348 disposable follow-up tests that precise scheduling
requirement without changing production kernel behavior.

Evidence retained on Termux host:
`$TMPDIR/daimos-swap-fullguest-halt-v1347-evidence-r1.tar.gz`.
SHA256: `dcab588ef04e13280b8417ce017e7e47a012ea89cb99fe4a1246eb3cc4d65045`.
The archive contains the actual linked map, source address manifest,
raw/canonical SIMH output and original postflight checker results,
along with the unmodified v1150 error record and a verified manifest.
