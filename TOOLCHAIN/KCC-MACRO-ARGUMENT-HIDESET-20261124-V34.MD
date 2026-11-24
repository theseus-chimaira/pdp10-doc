# KCC macro argument recursion suppression (v34)

The preprocessor must preserve permanent macro recursion suppression
when collecting an argument from an already expanded replacement list.
Previously ccpp.c margs() copied raw tokens via tokize(), losing the
IS_MHID identifier status. This incorrectly re-enabled recursive macros.
The historical macro1.c nested stpcpy case expanded to
`2 + M(2 + 2 + M(9))` rather than `2 + M(2 + M(9))`.

The v34 fix preserves IS_MHID for identifier tokens originating in a
replacement list, without adding a token field or runtime dependency.
Raw source tokens retain ordinary classification.

Verification on the hosted cross compiler: staged strict macro suite
60/60 PASS; installed exact historical CPP diagnostic replay 20/20
PASS (16 GCC, four KCC). The four KCC cells formerly failed the nested
macro assertion. The v33 console-free diagnostics preserve assertion
expressions and use a nonzero source-line return rather than puts/abort.
The cumulative historical 50-cell linker-error cohort is now closed.
No full matrix sweep or native compiler validation is claimed.

KCC fix: 8fc1bdc. Testkit evidence: bcad78f. The testkit contains the
original v33 observations, the v34 exact replay manifest and the new
macro argument hideset regression.
