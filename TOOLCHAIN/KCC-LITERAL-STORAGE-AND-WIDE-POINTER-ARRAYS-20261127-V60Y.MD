# KCC literal storage and wide pointer array repairs (v60w-v60y)

Date: 2026-11-27. Extends KCC-REINSTATED-EXTENSIONS-20261127-V60V.MD.
Installed host compiler: e75fe2b. Testkit: 89cb832.

## v60w: literal pool sharing

KCC 423492f, testkit a811d3e. Share equal immutable encoded literal
byte sequences in the existing lexer pool, reclaiming the just-written
duplicate. Length and encoding remain on the consuming node.
No new index, buffer or metadata is required. Native pool capacity
remains 4096 characters; host capacity remains unchanged.
The search is linear in committed pool bytes per literal, with
additional comparisons on matching prefixes. Compilation speed has
not been benchmarked. A new literal must still fit before sharing.
Distinct literals can still exhaust the fixed pool. This shares
compiler storage, not necessarily generated program literal addresses.

Original 931004-1.c: 22/22 PASS. New semantic regression: 22/22
PASS (512 repeated literals, embedded nulls, ninth-bit characters,
concatenation and wide character arrays). Existing string-literals
coverage: 18 compile_only. No failed acceptance cells.
Initial native compile exposed conflicting prototypes when including
the self string header. The accepted comparison uses a direct loop.
The first semantic guard also exposed a pre-existing wide pointer
array initializer bug, confirmed by identical old/new assembly on
the smaller reproducer. That defect was repaired separately below.

## v60x: static wide pointer arrays

KCC 20637df, testkit 73a35ad. Restrict the wide string contents
initializer path in ccgen.c to wide character arrays (TS_INT elements).
Pointer arrays instead emit relocatable literal addresses through the
existing scalar initializer machinery. Tested static and automatic
constant arrays, nested arrays, null padding and wide character
arrays. 44/44 PASS. Packed static pointer relocation support is
not added by this repair.

## v60y: runtime wide pointer arrays

KCC e75fe2b, testkit 89cb832. Apply the same element-type distinction
to genautoiz in ccgen1.c. The new mixed literal/function-call pointer
array regression failed all 22 cells before the patch and passed all
22 after. Total acceptance: 106/106 PASS, including 84 other guards.

## Native object footprint

Cross-compiled before/after source copies with the same installed
host KCC for each patch, -Pgnu99 -O -x=pdp6 -m=gas,
HOST_DAIMOS=1, HOST_UNIX=0, self headers. For code-generation objects
also checked the KCC_PHASE_GEN variant. DAS -F -C assembled each
object. Counts are 36-bit words from DOBJ headers.

Patch | Text before | Text after | Data delta | BSS delta
---|---:|---:|---:|---:
v60w cclex | 3452 | 3584 | 0 | 0
v60x ccgen | 5122 | 5122 | 0 | 0
v60y ccgen1 | 3421 | 3440 | 0 | 0

Total text increase 151 words, no permanent data increase.
Actual linked native phase size, native self-host execution, peak
RAM and compiler execution-time effects remain unmeasured.

## Cumulative status

Original 2196 KCC compile_fail cells: 1154 explicitly unsupported,
540 recovered, 502 remaining. All 540 recovered cells replayed on
installed v60y: 540 PASS, no cached results or failures. The new
wide pointer array bugs were not members of this original cohort.
Full selection dry run: 40886 KCC cells, 85799 GCC cells, 126685 total.
No new exclusions, GCC changes or DAIMOS source changes.
This is not a full matrix sweep.

Evidence under TMPDIR:
kcc-string-pool-v60w.9HSRnY/guards-v2,
kcc-wide-ptr-init-v60x.1ebtPG/guards-v1,
kcc-wide-runtime-v60y.K8V7VR/baseline-v1 and guards-v1,
kcc-cumulative-v60y.KcIrJX/replay-v1.
Native before/after DOBJ files are in each patch directory.
Exact recovered/remaining manifests are in the cumulative parent.

Final compiler-only install archive:
kcc-wide-runtime-v60y.K8V7VR/install-v1/local.tar.gz
SHA256 ec52d5dd1fdaae2f23631bb30671c87a4741de41b648e7d9c6f7c37f7517d847
Installed bin/kcc SHA256:
9846159c1dba1b3594358cd0892367d92af17258414ade5371d9da4179edb814
