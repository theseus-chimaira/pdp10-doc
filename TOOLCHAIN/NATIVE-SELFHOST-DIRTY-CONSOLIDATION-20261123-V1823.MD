# Retired self-hosting source-staging worktree, v1823

Date: 2026-11-23. This note distinguishes recovered CAT/MAKE memory
improvements from an unaccepted self-hosting boot-image proposal.

The historical v1414 worktree proposed three related changes:

1. Stream CAT S6REC text without a 259-character local line array.
2. Store MAKE goal names as offsets into its already existing
   arena, rather than a fixed array of goal strings on the stack.
3. Stage native DAS and MAKE source trees and MAKEFILEs into the
   default D6FS root image, raising the DSH image-size ceiling
   from octal 036000 to 037000 during that experiment.

The first two changes were previously integrated with corrections
in v1814. CAT's streaming implementation resides in a separate
archive member, not the common libc text object, so unrelated
commands avoid extra permanent TEXT. The staged old CAT path
would have subjected long text to incompatible sink buffering.
The current v1814 tests were rerun: exact S6REC records containing
0, 7, 258, 259 and 1000 characters pass, along with terminal
output and malformed record cases. Native MAKE's old frame of
587 words shrank to 122 words, saving 465 stack words. MAKE
object TEXT is 7748 versus the old 7876 words; BSS remains 839.

The self-hosting staging feature was **not** merged. Its shell
script and two native MAKEFILEs are preserved as candidate source,
but they were not verified as a native GCC/KCC/DAS rebuild, actual
native MAKE execution, or a complete runtime acceptance test.
The current DSH image still has a separate pre-boot correctness
gate: the earlier full-image experiment measured octal 036133,
which exceeds the configured limit of 036000. Simply lifting the
gate to 037000 is not accepted as a size or correctness fix.

All eight tracked/untracked dirty files from the old self-host
branch, their exact SHA256 manifest, original Git base, and a
Git binary patch were preserved under
`~/backups/daimos-selfhost-v1413-v1414-dirty-retire-v1823-20261123/`.
The old worktree is now clean. No production staging, limit, or
kernel source changed. Native compiler correctness and a safe
real-guest self-host sequence remain open in DAIMOS TODO.MD.
