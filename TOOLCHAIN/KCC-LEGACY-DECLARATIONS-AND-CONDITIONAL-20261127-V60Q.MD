# KCC legacy implicit-int declarations and GNU omitted conditional

Date: 2026-11-27. Extends KCC-BITFIELD-AND-ARRAY-DECAY-20261127-V60N.MD.
Installed host compiler: KCC 7efd7b6. Testkit: ac22506.

## v60p: legacy external declarators

KCC 9f08eb2; testkit 083b6a4. In GNU/KCC compatibility mode,
an external declaration without storage-class or type specifiers
uses int as its base type. Each subsequent declarator gets that
base type, not the derived type of the first declarator.
Strict C89 and C99 retain the previous diagnostic for a declaration
without specifiers. Direct mode probes confirmed GNU89 and GNU99
acceptance and strict-mode rejection.

Original 931013-1.c: 22/22 PASS. New semantic regression: 22/22.
Existing frontend guard: 22/22. Total acceptance: 66/66.

## v60q: omitted-middle conditional

KCC 7efd7b6; testkit ac22506. GNU a ?: b evaluates a once and
uses its value when nonzero, otherwise evaluates b. Reuses the
ordinary conditional and assignment nodes. Literal constants
need no temporary; other operands in function scope are saved
in ordinary automatic storage. No compiler type or node
structure is enlarged. Generated functions can require additional
stack words. GNU89 and GNU99 accept; strict C99 rejects.

Original 981001-4.c: 22/22 PASS. New semantic regression: 22/22.
Other conditional and constant-query guards: 48/48.
Total acceptance: 92/92. Tested integer, pointer, double, wide
integer, volatile and nested operands, and evaluation counts.

Nonconstant operands in unevaluated file-scope expressions remain
unsupported by this implementation. Do not claim complete GNU
conditional-expression compatibility. Native self-hosting and
peak memory usage have not been independently measured.

## Cumulative verification

Original KCC compile_fail cohort: 2,196. Explicit unsupported:
1,000. Recovered: 430, including 44 in this series. Remaining: 766.
All 430 recovered cells replayed on the installed v60q compiler:
430 PASS, no failures or cached results. This is not a full
matrix sweep. GCC and DAIMOS sources were not changed.

Evidence under TMPDIR:
kcc-implicit-int-v60p.7twlvv/guards-v1,
kcc-omitted-query-v60q.5pENfx/guards-v1,
kcc-cumulative-v60q.9xeiME/replay-v1.
Exact recovered and remaining cell manifests are in the last
directory's parent.

Final compiler-only install archive:
kcc-omitted-query-v60q.5pENfx/install-v1/local.tar.gz
SHA256 1dd73b3fa7c99478b4a2b466a6dae4a317874ef44a75e70a628cdd73347c2da2
Installed bin/kcc SHA256:
1d602315538f4038a78799a1c30b40b4358d715b49eb9f57ea58a72e82ff01c1
