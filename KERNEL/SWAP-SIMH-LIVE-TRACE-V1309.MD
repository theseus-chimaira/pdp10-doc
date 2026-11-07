# DAIMOS swap-in liveness: read-only SIMH live tracing v1309

Date: 2026-11-07. Scope: 256K PDP-6, KCC, DSK/D6FS physical
swap/OOM acceptance, with the production requester-only BRK OOM
policy enabled. No production kernel or MM/swap source changed.

## Why the live trace exists

The 24-user F07 guest exercises true process swap-out and swap-in,
but successful ordinary swapping does not prove a no-exchange-partner
allocation failure occurred. A completed 16K-BSS guest recorded 41
successful swap-ins and zero allocation-pressure returns/cooldowns.
A separately observed 24K-BSS guest admitted 18 RUN children and,
before its diagnostic stop, counted 49 swap-outs and 33 swap-ins,
again with zero allocator-pressure returns and zero cooldown entries.
The historical fallback experiment added 16 SWAP MRES words but had
not reproduced a failure on the clean kernel; it remains excluded.

## Diagnostic implementation

The testkit v1306 sampler runs SIMH through a host PTY and sends
Ctrl-E at a bounded periodic interval. It examines PDP-6 PC and
linked test-kernel scheduler, swap, allocator, and backing counters,
then issues GO. All examined symbols use addresses from the
disposable guest's actual KCORE map, not previous version addresses.
It reads the machine, never deposits a memory word or changes the
production kernel. Its brief pauses alter timing; samples must not
be used as a throughput benchmark.

The full raw SIMH monitor transcript is saved separately as
`simh.out.raw-v1306.log`, and the compact periodic summaries are
saved as `simh-sampler.log`. The ordinary guest output `simh.out`
excludes debugger commands/results, preserving F07's exact milestone
sequence and post-HALT MM accounting. At normal HALT the runner
executes the original boot.ini's postflight EXAMINE/EXIT commands,
but never modifies the original boot.ini. The v1309 correction
removes the interactive `sim> ` prefix from the separate post-HALT
guest output, including when the prefix spans PTY reads.

The sampler's PTY command/resume/postflight, missing-symbol failure,
and partial-snapshot negative control PASS against a protocol stub.
The real PDP-6 short two-sample test PASSes with PC/timer/counter
observations. `--stop-after-samples` intentionally terminates the
guest without claiming a full-system pass.

## Running the longer physical guest

```sh
export TMPDIR=${TMPDIR:-$HOME/tmp}
export DAIMOS_REPO=$HOME/git/DAIMOS
export PDP10_PREFIX=${PDP10_PREFIX:-$HOME/cross}
export KEEP_TEST=1 V1306_LIVE_SAMPLER=1
export V1306_INTERVAL=60 V1306_COUNT=15 V1306_TIMEOUT=1810
sh "$HOME/git/daimos-testkit/tests/validation/daimos-swap-no-partner-fullguest-v1304.sh"
```

After the guest's build, obtain its current versioned fixture directory
from `$TMPDIR/daimos-oom-requester-live-v1075.*` and read
`simh-sampler.log` for elapsed live progress, `simh.out` for the
unmodified guest milestones, or `simh.out.raw-v1306.log` for complete
monitor responses. Do not infer a hang merely from one sample with
no swapped-process progress; the kernel may be sleeping or waiting
on an ordinary provider. A genuine no-partner observation requires
actual swap-in MM failure, no eligible pair, cooldown, independent
runnable activity, later restoration, and final zero leaked backing.

The physical v1303 verifier accepts only completed guest evidence
and identifies unexercised no-partner conditions with exit status 3;
`--snapshot` also exits 3 even if it can classify a partial sample.
Its positive/negative synthetic controls do not substitute for a
physical guest. New source/testkit commits: `5e23ff0`, `3fa4792`.

The first v1308 long 24K guest was still in progress at the
documentation checkpoint. It must not be represented as accepted
until the full guest and final memory-accounting checks complete.

## 2026-11-07 v1312: completed traced guest and replay verification

The original v1308 24K-word-BSS, 256K KCC PDP-6 guest finished all
18 additional RUN admissions and 18 TERM/WAIT reaps, requester-only
BRK OOM, witness/provider milestones and the expected HALT. All 15
live read-only snapshots completed. At the final linked after-HALT
snapshot it had 51 successful swap-outs, 51 swap-ins, zero resource
allocation failures, zero exchanges and zero no-partner cooldowns.
The original v1308 runner returned nonzero **after** the requester
OOM milestone because the in-flight earlier sampler revision wrote
interactive `sim> ` prefixes into its final address/examine records.
The v1150 verifier could not recognize its `armed` address. This
was a simulator/trace formatting failure, not a demonstrated kernel
owner or MM failure.

Testkit v1311 `9ddb748` extracts the unchanged original v1075
postflight validation block, makes a separate copy of the original
captured console with only line-leading monitor prompts stripped,
then replays the **same original** F07 checks. The recorded guest
passes provider ownership/retry (v1150), protected processes (v1159),
physical MM (v1203), actual process APR/BRK accounting (v1205),
real swap/stack watermarks (v1207), and release-media/E13 evidence
(v1160), including internal SHA256 manifests. A negative fixture
altering exactly the provider armed counter from -1 to 0 is rejected.
The immutable original and replayed logs are both retained. This is
**offline postflight verification**, not a new direct full-guest run.

Evidence archive on the Termux host:
`$TMPDIR/daimos-swap-no-partner-evidence-v1312-r1.tar.gz`, SHA256
`e5224db0e4266bea68c7e953c22f49c6074443ea930a5b360665725ab8a1b669`.
The outer 28-file and inner 9-file SHA256 manifests both verify.
There is still no observed real swap-in allocator pressure or
no-equal-exchange-partner retry on this current production kernel;
full-system no-partner forward-progress acceptance stays OPEN. Do
not merge the historical v1208/v1211 experimental fallback without
an actual failing workload, independent cleanup and memory costing.
