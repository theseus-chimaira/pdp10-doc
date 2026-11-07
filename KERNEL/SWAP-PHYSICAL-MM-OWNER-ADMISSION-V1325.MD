# v1325: failed 8K MM-owner admission in booted F07 pressure guest

Date: 2026-11-07. This is a **negative experimental result**, not
a swap-in liveness acceptance or a kernel defect. Production DAIMOS
kernel, memory allocator, process swap-in, and scheduling code remain
unchanged.

The v1322 disposable 256K PDP-6 F07 guest cloned 18 additional
24K-word-BSS user processes and reached the original INIT test-only
physical occupancy stage. At that point an extra 020000-octal-word
(8192-word) `MM_TYPE_KERNEL_DYNAMIC` extent, type 3/owner 075, was
requested using the actual `mm_alloc_aligned_noreclaim`. The
allocation failed; the guest emitted its `!t` failure marker and
HALTed. It never completed the downstream requester/provider/WAIT
milestones. This does not represent a failed production swap-in.

The post-HALT SIMH dump records `f07h_error = 777777777777` (-1),
`f07h_prepared = 0`, `f07h_count = 0`, `f07h_freed = 0`. The actual
swap-in MM allocation failure count is zero, as are swap-service
pressure, cooldown and exchange. The original cleanup did not
leave any test-owner extent. A new v1325 strict counter verifier
checks the HALT marker and those final counters, rejects altered
error/owner/pressure/cooldown values, and reports test-setup
rejection only.

The previously completed v1308 24K-BSS physical MM snapshot at
requester BRK OOM showed 15840 unallocated managed physical words,
spread across three holes, with largest free gap 14802 words and
eight resident user extents. That late snapshot **cannot** prove
the exact free-space layout at v1322's earlier owner admission;
nevertheless it explains why large reservations are risky. The
live v1322 SIMH sampler observed ordinary swapping (at least
50 successful swap-outs/33 swap-ins before the final HALT) with
zero no-partner pressure through its ten read-only samples.

The next independently versioned experiment v1323 requests one
02000-octal-word (1024-word) owner-tagged extent, preserving the
same real allocator and typed release. Its final result must be
classified separately. In particular, successful owner admission
does not imply a pending `proc_swap_in` returned MM_ERR_NOMEM, nor
does it imply that equal-sized exchange candidates are absent.

Evidence, including actual SIMH guest transcript, addresses,
sampler trace and strict verifier with SHA256 manifest, is retained
on the Termux host:

`$TMPDIR/daimos-swap-owner8k-rejection-v1325-evidence-r1.tar.gz`

SHA256:
`bf5304eb2ce0de91d463b8137c601e9c77f4223bea7cf67ebb3a3e4582800e31`.

The genuine full-system no-partner recovery gate remains OPEN.
