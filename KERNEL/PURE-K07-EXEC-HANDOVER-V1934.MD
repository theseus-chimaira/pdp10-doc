# Transactional EXEC backing handover, v1934

The normal staged EXEC transaction preserves both live PURE backing
identities when the shared process record and credential extension
are replaced. Before loading, the original packed PURE record is
decoded against its stable u-area and published mount. After successful
load, the replacement record is decoded before startup. Each dependency
is held in a two-word stack record using the v1933 reservation list.
No extra provider wait is necessary for extending an already published
owner into a transaction hold. Non-PURE images need no hold. Invalid
old backing refuses the transaction; an invalid successful new
attachment is an invariant failure.

The file mutation and mount-owner scanners see both dependencies
through startup, old-image teardown, staged rollback and record
replacement. The common return releases the transient holds after
the surviving record has been committed/restored. The NOMEM branch
releases its outer hold before calling the destructive retry while
the original permanent owner is still published. This adds four
transient stack words and no permanent process/mount storage.

Implementation 81a04511, testkit 51b56bf. Nine native staged EXEC
paths pass using real backing decode, reservation and owner scans
at controlled loader/startup/teardown boundaries. Missing old hold,
missing new hold and missing release unsafe variants are rejected.
Earlier v20/v200/v1719/v1720/v1933 native regressions pass. The
v1720 fixture does not establish complete destructive EXEC backing
publication correctness; it controls VM/loader completion.

Folded KCC lowmem tape fixed KCORE low end 025411 -> 025446 octal
(+29 words); DXR image 024750 and BSS 000416. PURE MRES 3879 resident
words unchanged. No new permanent data.

K07 remains open for destructive low-memory EXEC direct publication,
lookup-to-reservation namespace and fatal/private-stack lifetime,
MEMFS mutation, compressed/raw-XIP integration, poisoned retirement
and physical PI6 acceptance. RAW PURE and mode3 remain default OFF.
