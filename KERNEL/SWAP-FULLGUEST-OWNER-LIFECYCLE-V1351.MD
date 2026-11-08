# v1351: full PDP-6 F07 physical swap and owner lifetime accepted

Date: 2026-11-08. Production DAIMOS kernel unchanged.

The *second*, independently booted 256K KCC PDP-6 F07 guest
(testkit `f156746`, v1349 runner) completed the original complete
physical-OOM/held-provider/F07 suite and also passed the separate
source-linked two-phase real-MM owner verifier in its original
after-HALT postflight and SHA256 evidence manifest.

The suite verifies eighteen authentic additional child RUN/ACK and
TERM/WAIT operations, requester-only actual BRK OOM, protected
blocked witness, serialized filesystem holder and foreign waiter,
three genuine fatal-busy retries, one guarded holder return through
production filesystem unlock, and at least five real independent
spinner SLEEP operations. It completed HALT and SIMH EXIT.

| Physical acceptance counter | Final observation |
| --- | ---: |
| Genuine typed swap-in MM_ERR_NOMEM failures | 1 |
| No-eligible-partner cooldowns | 1 |
| Successful swap-ins | 34 |
| Swapped-process exchanges | 0 |
| Swap blocks outstanding at HALT | 0 |
| Backstore blocks outstanding at HALT | 0 |
| Children admitted / reaped | 18 / 18 |
| Genuine held-provider fatal BUSY attempts | 3 |
| Normal provider holder return | 1 |
| Independent spinner progress witness | 8 |

The newly *integrated* v1349 owner-lifetime verifier observes
four actual 8K-word owner-tagged `MM_TYPE_KERNEL_DYNAMIC` extents
and a real resident spinner pin at diagnostic sample 1, before
the production swap-in memory shortage. At sample 13, after real
MM_ERR_NOMEM/no-partner cooldown, the independent spinner has
released all four physical extents using production `mm_free`,
unpinning its own process in the original transaction. The
final link-derived owner counters are `prepared=4`, `freed=4`,
`count=0`, `error=0`; six deliberately invalid owner-life evidence
states fail verification.

The suite also checks the v1207 **fixed mandatory after-HALT
field schema**. Additional linked `f07s_diag_*` read-only live
sampler addresses do not appear in the original final scripted
examine list and are optional; they are not mistaken for missing
required fields. Both missing and duplicate required-field
negative controls fail. No original final MM, backstore, stack,
provider, media, OOM or child lifecycle assertions were removed.

The full accepted original v1348 guest is documented separately
in `SWAP-FULLGUEST-COMPLETE-V1348.MD`. v1351 adds a *fresh full
guest with integrated owner-phase check*, rather than extrapolating
its acceptance from v1348's offline verification.

### Archived evidence

`$TMPDIR/daimos-swap-complete-lifecycle-v1351-evidence-r1.tar.gz`

SHA256:
`4ab99004129151bc475e7a5a577d39ebf336e6c290acb701696c5f1655e7e644`.

The archive holds real guest addresses and linked kernel map,
original and read-only SIMH streams, original F07 postflight log,
owner-lifetime and physical no-partner independent verifiers,
reproduction test sources and a verified per-file hash manifest.

**Acceptance boundary:** This closes booted 256K KCC PDP-6 DSK/D6FS
raw swap no-partner recovery, independent physical MM release and
held-provider recovery in the tested profile. It does not test
every hardware machine or future compressed swap format. No
production KCORE, MRES, PI6, MM, FS or swap source or permanent
memory footprint changed; the speculative old +16-word swap
fallback is not justified by the accepted workload.
