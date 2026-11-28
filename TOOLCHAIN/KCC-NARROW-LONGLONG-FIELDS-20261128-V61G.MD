# KCC single-word long long bitfields and wide constant conversion (v61g)

Date: 2026-11-28. KCC 017b3a8, testkit 3f038c0c.

GNU/KCC mode accepts signed and unsigned long long bitfields of
width at most 36. They reuse the existing signed/unsigned bitfield
types and native layout. Promotion uses the actual field width:
signed fields through 36 bits and unsigned fields through 35 bits
promote to int; unsigned 36-bit fields promote to unsigned int.
Strict-mode declaration rejection is retained. Fields exceeding one
word still require a separate implementation and remain diagnosed.

The initial declaration-only experiment passed the existing GNU
promotion oracle but failed the new static initializer regression.
The constant-folding CAST_IT_IT path had discarded payload bit 35
when narrowing a normalized 71-bit arithmetic constant. Reconstruct
the physical low word from high-limb bit zero and the 35 low payload
bits before target-width truncation, and clear NF_WIDE. The existing
integer-to-pointer constant conversion shares the same helper. No
change to target runtime representation or calling convention.

## Verification

The v61f baseline and successive candidates used separate prefixes
and fresh checkpoints on charybdis. The initial 198-cell guard run
gave 132 PASS and 66 compile_fail at baseline. The declaration-only
candidate gave 154 PASS and 44 run_fail, including the new narrow
field test. The constant repair recovered all 22 new field cells.
The independent wide-to-narrow constant regression (positive and
negative values, static initializers and volatile runtime conversion)
failed 22/22 at baseline and passed 22/22 after the repair.

The final shared-helper build passed 308/330 selected bitfield,
negative, pointer constant and wide arithmetic guards; the other
22 were the unchanged historical bf-sign-2.c runtime failures.
The additional ordinary 36-bit promotion oracle passed 22/22.
Both existing target-correct promotion companions are registered
for all 22 applicable KCC cells. Explicit strict C99 opt/noopt
declarations reject normally without ICE. The original source and
its registrations are unchanged: it expects unsigned 32/35-bit
fields not to promote to signed int, contrary to native 36-bit int.
See testkit history/PDP10-P1-BATCH2-FINAL-20261119-v1.md.

Fresh 1042-cell active historical replay: 760 PASS, 260 compile_fail,
22 run_fail. The entire previous failure-key set is unchanged;
exactly the 22 bf-sign-2.c cells transitioned from compilation
failure to runtime failure. No previous pass regressed. No new
scope exclusions, no new historical passes claimed. The original
2196 cells partition into 1154 documented unsupported, 760 PASS,
260 compile_fail and 22 run_fail. Remaining compilation failures
span 17 sources. The runtime transition remains separately tracked.

Testkit manifests (under manifests/):

- PDP10-KCC-COMPILE-REMAINING-20261128-v61g.tsv
- PDP10-KCC-COMPILE-RUNTIME-TRANSITIONS-20261128-v61g.tsv

## Native footprint and installation

Native PDP-6 -Pgnu99 -O -m=gas, HOST_DAIMOS=1, HOST_UNIX=0;
same v61f compiler and DAS -F -C. DOBJ sections in words:

| Source | Text before | Text after | Data before/after | BSS before/after |
| --- | ---: | ---: | ---: | ---: |
| ccdecl | 13920 | 13922 | 1 | 13 |
| cceval | 7138 | 7178 | 0 | 2 |

Total +42 text words, unchanged data/BSS. Duplicated reconstruction
initially increased cceval by 89 words; sharing saved 49 words.
Linked native phase size, native execution and timing not measured.
Existing cross-word one-bit load/store generation is unchanged;
efficient multiword fields and native physical/arithmetic limb
conversion require separate work. The three historical sources
920501-3.c, bf64-1.c and 991118-1.c still fail compilation.

Evidence relative to TMPDIR: kcc-narrow-longlong-field-v61g.WIauFo
(baseline-results, candidate-results, narrow-v2, cast-baseline,
cast-candidate-v2, shared-guards, target-oracle-v3, shared-replay,
native, strict mode logs, install-v1). Compiler-only local.tar.gz
installed into PDP10_PREFIX; bin/kcc verified byte-identical to
candidate-v3. Previous compiler retained in install-v1.

Installed bin/kcc SHA256:
484c4c1e56d5af9dd7265b63491a9b54beb55ba8628e80501f75182e94b59fb8

Compiler-only local.tar.gz SHA256:
575f42e7a1169bc51671379c23246fd5ebfeeadb5151edbab6eee671cd2c616d
