# KCC optional #line filename and blank-line accounting (v59d)

KCC bf24e1e; testkit aa6d1c94. All four KCC cells of the
historical `line4.c` source crashed in d_line on the November 25
full sweep. The filename token is optional, but the wide-string
check dereferenced it before checking for NULL.

Guard both filename operations. After the crash was removed, the
unchanged upstream source exposed an independent off-by-one error:
the tokenizer had already counted an immediately following blank
line before resetting the logical line number. Retain that increment
when the lookahead character is newline. Neither fix changes target
code generation or ABI.

The original four codegen cells and all 22 new KCC semantic cells
of `kcc-line-directive-v59d.c` pass. The latter covers a numeric
directive, macro-expanded line and filename, two consecutive blank
lines, and filename preservation across a subsequent number-only
directive. No full preprocessor conformance claim is made.

Evidence: `$HOME/tmp/kcc-line-v59d.C9J4aU89`, `after-v3` and
`regression-v1`. The first build invocation targeted an existing
relative filename instead of the absolute Makefile target and did
not rebuild; its results are not candidate evidence. The final
installed `install-v3/local.tar.gz` contains only bin/kcc; SHA256
`d6da38dedec40a6a12345092ec2f4ed2323f0d97ead185536baa4e7e4658ba18`.
