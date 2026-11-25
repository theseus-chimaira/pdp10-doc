# Destructive low-memory EXEC live identity, v1935

The live process descriptor cannot temporarily carry an executable
entry in its META left half: that half identifies the stable u-area.
Nor can its scheduler word be overwritten with staged SIDL state.
The common loader publication leaf now distinguishes the current live
descriptor from the detached RUN/EXEC descriptor. Staged publication
keeps the existing entry/SIDL format; live publication leaves META
and SCHED intact. VM rollback clears only a detached descriptor's
entry. The register ABI returns the logical entry in AC2 on success,
including the certified XIP origin. The C status return remains AC1.

The destructive retry holds the existing user-mapping flag across
replacement loading and startup, protecting the new VM against the
existing reclaim admission checks. It releases the hold on success
or before returning a fatal continuation. The old image destruction
and precommit mapping rollback contracts are unchanged. No new
permanent data; the saved-META local is no longer necessary.

Implementation 27f6be53, testkit 902f1ed. Seven native destructive EXEC
paths pass with actual loader, KCC-compiled backing attachment,
reservation, gateway, owner scans and mapping leaves. Controlled
MM/I/O/startup boundaries exercise successful PURE DXR2 loading and
lookup, STAT, header, allocation, payload and startup failures.
Four unsafe variants fail native assertions. Earlier v1720, v1933,
v1934, v1563, v200 and v1719 gates pass. The historical v1720 test
models the new entry ABI and tests fatal mapping release.

Folded lowmem KCC tape KCORE low end 025446 -> 025444 octal (-2 words);
DXR image 024746, BSS 000416 and PURE MRES 3879. Lowmem frame 012 -> 011
octal words. No process-table expansion. This is not complete-guest
physical I/O or PI6 acceptance; compressed/XIP execution is not
covered by the new fixture. K07 remains open for namespace admission,
fatal/private-stack lifetime, MEMFS mutation, poisoned retirement
and integrated physical validation. RAW PURE/mode3 remain default OFF.
