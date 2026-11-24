# KCC v32 static-inline reachability

The Unix-hosted cross compiler now folds identical function designator
comparisons and suppresses provably unreachable statement tails while
preserving goto and switch labels. Internal static-inline assembly is
spooled to disk; actual emitted symbol references form the reachability
graph. Only live definitions are replayed. No complete AST is retained.
Native DAIMOS KGEN still requires a verified native spool implementation.

The installed compiler passed 238/238 fresh strict test cells with -j4.
The exact 34 historical KCC link failures were rerun: 30 PASS and four
link_fail, all remaining cases requiring a real hosted puts provider.
The original 20011115-1.c passes all four previously failing cells.
No linker relocation was suppressed and no console stub was introduced.

Installed compiler SHA256:
`aaeb1d409dabc1c5ade71c36f763cca2727cbd86050cc5fc69b986227c160017`.

Detailed evidence and reproduction manifests are in pdp10-testkit:
`history/PDP10-KCC-STATIC-INLINE-REACHABILITY-20261124-v32.md` and
`manifests/PDP10-KCC-STATIC-INLINE-ACCEPTANCE-SOURCES-20261124-v32.txt`.
GNU extended-asm constraints remain unsupported independently of v32.
