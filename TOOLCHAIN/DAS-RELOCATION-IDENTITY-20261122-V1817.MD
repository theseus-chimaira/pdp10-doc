# DAS DOBJ1 relocation identity correction v1817

Date: 2026-11-22. This records a completed host assembler correctness
fix and source-linked regression. Native guest execution is not implied.

## Defect

The historical v1453 DAS test, preserved in a detached dirty testkit
worktree, demonstrated that DAS silently assembles an expression such
as `EXTERNAL_B-EXTERNAL_A` into a constant, even though both symbols
are independently relocated by the future linker. The original
assembler tracks a signed relocation count and used a final count of
zero as proof that the difference was absolute. This is unsound for
distinct relocation bases. DOBJ1 cannot encode arbitrary differences
between separately relocated symbols or sections.

## Correction

The host DAS expression evaluator now checks the identity of the two
relocation bases before allowing plus/minus terms to cancel:

- A local TEXT section base cancels the same local TEXT section base;
  independently relocated TEXT/DATA/BSS sections do not cancel.
- Two references to the same external symbol can cancel; references
  to different external symbols cannot.
- Parenthesized unary minus and reversed operand order must preserve
  this rule. A noncancelling multi-base expression is rejected rather
  than emitting a DOBJ1 record whose relocation metadata is lost.
- A single remaining relocation continues to carry its original
  section/symbol identity and addend.

The common relocation algebra is used by the host and native source
parser definitions. Actual KCC-generated native DAS1 and DAS2 target
assembly/object files match the pre-fix native phase outputs byte for
byte; the native phase build does not exercise the affected expression
evaluator code path. Both native phase DXRs link successfully; their
image sizes are 035205 and 032730 octal words respectively. These
figures are build output, not acceptance of a target resident limit or
native guest execution. No fixed DAIMOS KCORE code was changed.

## Verification and preservation

The original recovered v1453 fixtures and expanded valid/invalid
cases were consolidated under
`pdp10-testkit/misc/das/symbol-difference-v1817/`, not OS testkit,
because assembler semantics are owned by the standalone PDP10
toolchain. A registered fast test checks four invalid differences
and two valid cancellation patterns, including the expected same-
section machine-word constant. Old DAS accepts the invalid original;
corrected DAS rejects it with the explicit relocation-identity error.

The complete registered `das-fast` suite passed 13/13. Its existing
native relmap spill test originally failed to link due to an omitted
diagnostic `dsys_writechar` syscall mock, identically with old DAS.
The host test now supplies that bounded mock; its relocation-bit
assertions are unchanged and pass. The original recovered untracked
files were separately archived before retirement from the old tree.

The source fix is published on `das/main` as `ed2d2e5`, together
with the earlier DAS commit `aa74f42`. The PDP10-testkit regression
is published as `b8b6021`. Publication of both DAS commits was
completed on 2026-11-23 (v1827): the DAS repository was already
using HTTPS, but lacked DAIMOS's local Android Git credential
helper and `credential.useHttpPath=true`. Using the same helper as
DAIMOS allowed the authorized push to
`https://github.com/theseus-chimaira/das.git`; those local DAS
Git settings are now persistent. `main` matches `origin/main`,
with no changes to code or history in the publication step.
Full native/guest DAS execution is still a separate source-matched
acceptance gate.
