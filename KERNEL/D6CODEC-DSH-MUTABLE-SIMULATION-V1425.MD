# DSH mutable-memory compression: deterministic PDP-6 simulation v1425

Date: 2026-11-10. **Completed test-only simulation, not a production RAM
compression feature or a captured live shell memory image.**

## Input and model

The source is the uncompressed native DSH DXR at
`DAIMOS/build/system/boot/pdp6/userland-build/userland/dsh.dxr`.
The actual DXR2 header reports 15,331 initialized words, of which
15,330 are TEXT and one is initialized writable data. BSS is 1,011
words. The kernel allocates 1,024 words for the startup stack plus
16 initial logical user words, rounded to 1,024-word APR granularity:
the initial physical VM size is 17,408 words.

In a *hypothetical* PURE-prefix drop, sector-aligned immutable
reclamation omits the first 15,232 words and leaves 2,176 words.
The last 114 words of TEXT remain in that suffix because of the
128-word sector boundary. Those 114 words and the one initialized
writable word are copied from the **real executable** for the
four source-shaped models. The deliberately incompressible control
instead randomizes the entire suffix, including the TEXT tail.

**Current DSH does not have the PURE DXR2 flag**. No existing
DAIMOS kernel path currently removes this hypothetical text prefix.
The experiment answers how well such a retained suffix might
compress if that policy were implemented.

The compiled DSH .bss declarations account for 966 of 1,011 BSS
words: among them, the 31-word input line, 454-word shell state,
192-word external command buffer, and 200-word inline AST node.
The remaining linked BSS is left zero. Synthetic startup records,
environment variable descriptors/payloads, aliases, jobs, parser
workspaces, and sparse stack contents follow DSH source conventions,
including counted SIXBIT records and two-word heap block headers.
Synthetic BSS placement is not asserted to be a linker-exact memory
dump. The generator uses a fixed random seed for the few simulated
nonzero stack words.

The initial process break is 17,408, above the reserved stack.
Small DSH heap allocations therefore force a full additional
1,024-word extent. Heavy function AST state adds 1,600 allocated
heap words (eight 200-word nodes), and requires two more APR blocks.
Scenarios model a just-loaded shell, idle prompt, command, loaded
functions/jobs, and completely random data as an incompressible
negative control.

## Physical retained suffix sizes (36-bit words)

| Simulated state | Raw | Grouped v1420 | Block v1424 | Block saved |
|---|---:|---:|---:|---:|
| Just after EXEC | 2,176 | 138 | 148 | 93.2% |
| Idle prompt, 6 variables | 3,200 | 280 | 358 | 88.8% |
| Running command, 12 variables | 3,200 | 463 | 618 | 80.7% |
| Function-heavy, 20 variables | 4,224 | 646 | 868 | 79.5% |
| Random/control | 2,176 | 2,249 | 2,193 | -0.8% |

For clarity, if compression were restricted to strictly writable
words **without** the 114-word immutable TEXT tail:

| Simulated state | Raw writable | Grouped | Block |
|---|---:|---:|---:|
| Just after EXEC | 2,062 | 21 | 34 |
| Idle prompt | 3,086 | 162 | 244 |
| Running command | 3,086 | 347 | 504 |
| Function-heavy | 4,110 | 530 | 754 |
| Random/control | 2,062 | 2,131 | 2,079 |

Every number is a codec payload count, not the size of an installed
compressed-process record. Future headers, metadata, compressed
buffer ownership, temporary workspaces and codec residency are not
included. Incompressible content expands under both formats and
requires a raw fallback if this experiment is used to design a
real memory-pressure policy.

## Test evidence

The generator and reproducible native runner live in
`daimos-testkit/tests/validation/d6codec-shell-mutable-v1425/`.
All ten host encode/decode pairs (five scenarios times two physical
regions) roundtrip exactly with independently checked codec
implementations. The PDP-6 simulator also executed the **real**
v1420 grouped decoder and v1424 register-resident block decoder
on each of the five complete physical suffix snapshots. All ten
native tests PASS and compare every decompressed word.

This establishes promising data compressibility for a representative
source-shaped synthetic shell, not actual dynamic shell footprints,
peak RAM savings, encoder time, nested VFS stack safety or a working
swapless mutable-image transaction. Next meaningful evidence is an
actual native DSH process-memory capture at prompt and after
real commands, followed by the same source-exact comparison and
end-to-end MM admission testing. The production decoder, DSH binary
and boot media remain untouched.
