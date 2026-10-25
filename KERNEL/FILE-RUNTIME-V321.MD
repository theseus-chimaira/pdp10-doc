# File-runtime lock mount identity (v321)

The resident DAIMOS file runtime checks FLOCK conflicts using the
canonical vnode identity, descriptor owner, regular-file lock state,
and the two high mount-identity bits stored in the u-area extension.
The high mount bits must participate in both the conflict scan and
state-update scan. Otherwise distinct mounts with matching low vnode
fields could interfere with one another.

As of DAIMOS commit ac8e5ab (2026-10-25), the two scanning paths use
one private assembly leaf, `file_lock_mount_ext_v319`. It takes the
current descriptor-pair address in AC2, returns the two-bit mount
extension in AC0, and preserves AC4, which holds the packed descriptor
word. The public descriptor layout, vnode representation, lock ABI,
provider dispatch and lock semantics do not change. No new resident
storage or provider interface is introduced.

Full GCC lowmem disk-root links comparing committed pre-change source
with the new source: KCORE 024764 -> 024756 octal (10740 -> 10734
decimal words); KCORE+MRES 036520 -> 036512 octal (15696 -> 15690
decimal words); PERMANENT_LAST 036577 -> 036571 octal. Total saved:
six permanent words. MRES component sizes are unchanged.

The daimos-testkit GCC fixtures `target-file-pdp6`,
`target-file-walk-pdp6`, `test-daimos-vfs-stat-frame`, and
`test-daimos-pre-dsh-fs-closure` passed. Lock fixture coverage includes
shared/exclusive conflicts and unlock operations. A dedicated test
for two distinct mounts whose low vnode fields collide is a further
regression opportunity. Strict 32K acceptance remains deferred.

## VFS CREATE request-mode staging (v322, 2026-10-25)

The shared CREATE/MKDIR/MKFIFO staging routine now moves the masked
creation mode straight from AC6 to request AC4 rather than taking an
unnecessary step through AC3. The mode's umask processing, existing
stack-passed node-result argument and provider ABI are unchanged.
The GCC lowmem full link saves one additional permanent word:
KCORE 024756 -> 024755 octal (10734 -> 10733 decimal),
KCORE+MRES 036512 -> 036511 octal. The four targeted file and VFS
regressions passed; strict 32K acceptance remains deferred.

## Provider service index normalization (v324, 2026-10-25)

The KCORE register gateway normalizes public provider numbers 4..7 to
service-table indices 0..3 immediately upon entry. MEMFS index zero
continues directly without taking the physical-provider lock. For DTFS,
D6FS and TSFS, the seven saved request registers now contain the table
index; the lock acquisition, sleeping wait, complete provider call,
release and wakeup sequence remain unchanged. There is no second
provider-index subtraction after restoring registers. Invalid IDs still
select the no-service return. This is private implementation staging,
not a change to the register provider ABI or MRES service jump table.

Measured full GCC lowmem disk-root link versus v322: KCORE 024755 ->
024752 octal, KCORE+MRES 036511 -> 036506 octal (three decimal words
saved), PERMANENT_LAST 036570 -> 036565 octal. Existing file and VFS
regressions passed. Additional multi-process contention tests remain
appropriate; strict 32K acceptance is deferred.

## Provider lock contract gate (v325, 2026-10-25)

The testkit now provides `make test-daimos-provider-lock-contract-v325`.
It checks source-level invariants of the resident provider bridge:
private AC1..AC7 preservation through wait/retry, MEMFS's direct route,
physical-provider serialization, completion unlock and wakeup, and
restoration of the provider result. This static gate passes for v324.
It is not evidence of correct scheduling under simultaneous requests;
a native multi-process contention test is still required before
changing the lock lifecycle. No additional RAM reduction was made.

## PDP-6 provider-lock register recovery probe (v326, 2026-10-25)

`daimos-testkit` provides `make test-daimos-provider-lock-native-v326`,
which extracts and executes the production `fs_provider_reg_call`
sequence on a native PDP-6 instruction simulator. A test-controlled
lock wait deliberately overwrites request registers before it returns;
the gateway must restore its process-private saved request, call the
correct provider, preserve its return and release the lock. The probe
also verifies the non-serialized MEMFS route. Native fixture PASS.
This is a simulated wait oracle, not simultaneous process execution or
physical-device I/O; real scheduler contention remains unverified.
No production change or permanent memory reduction was made in v326.

## Provider gateway dispatch extension (v327, 2026-10-25)

`daimos-testkit` now runs `test-daimos-provider-lock-native-v327`,
including the existing simulated lock-wait and register-recovery test,
plus D6FS/TSFS service-jump selection and invalid provider-ID rejection.
The extracted production gateway passes on the PDP-6 simulator. The
fixture uses controlled wait/wakeup and service oracles, not real
concurrent processes or physical filesystem I/O. Such a contention
regression remains outstanding. No production changes or RAM savings
were introduced.
