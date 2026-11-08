# v1348: complete 256K PDP-6 no-partner and held-provider F07 acceptance

Date: 2026-11-08. The original strict F07 full-system test on a
256K KCC PDP-6 has now **PASSed**, including its previously open
independent spinner progress requirement. Production kernel, KCORE,
MM, scheduler, filesystem, swap and MRES source remain unchanged.

The disposable v1348 guest adds one INIT read-only barrier before
terminating the independent spinner. It waits for the existing
source-linked count of at least five **real scheduled spinner SLEEP
operations** after the foreign filesystem provider acquires its
lease. The original threshold and fatal busy/rollback conditions
are not relaxed, and no synthetic process, MM or swap result is
invented. The original F07 suite verifies 18 additional child
RUN/ACK and TERM/WAIT operations, requester-only genuine BRK OOM,
the protected witness, serialized filesystem provider admission,
holder return and normal unlock, complete guest HALT and SIMH EXIT.

The original strict test runner reports its physical F07 and
held-provider acceptance as PASS. Independent verification of the
same completed, linked guest gives:

| Observation | Observed |
| --- | ---: |
| Genuine typed swap-in MM allocation failures | 1 |
| No eligible partner / 64-tick cooldown | 1 |
| Successful disk swap-ins | 34 |
| Swapped-process exchanges | 0 |
| Outstanding swap blocks at HALT | 0 |
| Outstanding backstore blocks at HALT | 0 |
| Additional child process RUN/ACK and TERM/WAIT | 18 / 18 |
| Held-provider fatal BUSY retries | 3 |
| Guarded holder return / normal provider unlock | 1 / yes |
| Independent spinner's genuine sleep witness | 8 |

The additional v1349 two-phase source-linked MM-owner verifier
was run **against this completed v1348 guest**, rather than being
mistaken for a separate v1349 boot. It verifies four genuine 8K
owner-tagged MM extents and a real spinner pin at diagnostic sample
1, exactly one independent owner release after real swap-in pressure
at sample 9, and final `prepared=4`, `freed=4`, `count=0`, `error=0`.
Six executable negative controls, including missing pin, missing
owner, fabricated pressure and leaked owner, are all rejected.
The separate strict v1303 physical no-partner postflight verifier
also PASSed. Neither checker replaces the original F07 suite.

The accepted v1348 guest evidence, original testkit programs, raw
and canonical SIMH transcripts, linked map, both independent
validation reports and SHA256 manifest are retained on Termux:

`$TMPDIR/daimos-swap-original-suite-v1348-evidence-r1.tar.gz`

SHA256:
`d97d0d05ef0c0cf6bd047d99e8c81ca64c5496b8b81690b50f588dea409e9951`.

The planned v1349 repeat adds the phase-correct MM-owner validator
inside the strict guest's own postflight/evidence manifest. That
separate repeat remains unverified until it independently reaches
HALT and passes all original and additional gates; it is not needed
to reinterpret the already accepted v1348 F07 guest.

The booted 256K no-partner recovery acceptance that was previously
OPEN is now closed by v1348. This does not qualify arbitrary real
PDP-6 hardware, other machine configurations, optional compressed
swap or future storage modes. There is no reason to incorporate the
historical speculative 16-word v1208/v1211 swap fallback.
