# KCC freestanding buffered stdio (v59u)

KCC feb0f8c, 441d6f1; testkit 1907fb65. Follow-up to v59t.

The original sweep has 54 sprintf-related link_fail cells (50
KCC, four GCC). KCC lacked a compiler-owned memory formatter;
GCC's indirect calls in 930513-1.c materialize sprintf with MOVEI,
which the testkit selector failed to recognize. The actual format
requirements are 32 integer/literal-string cells, twelve %.0f,
eight default %f (the variadic-manyargs companion), and two %g.

KCC now builds a byte-identical source mirror of GCC's existing
p10stdio-format-v2.c under runtime/libc, using KCC's own stdarg.h
and portable PDP-6 target instructions. The mirror keeps standalone
KCC source builds independent of a GCC source checkout; it must
remain synchronized with the shared C engine. Do not mix the two
compiled va_list representations. The installed KCC assembly and
archive are separate from GCC's, under lib/kcc/freestanding:

    p10stdio-format-v2.s
    libp10stdio-v2.a

The normal cross.mk freestanding installation and uninstall rules
include them; freestanding-stdio also builds the archive alone.
Core libp10c-v2.a remains unchanged. The optional KCC archive is
17912 host bytes (GCC's existing archive 11008); these are not
linked target memory measurements. The formatting engine adds no
heap, writable static state or OS dependency. The four buffer
entry points and callback extension use the same source, but
the callback's private va_list remains compiler-specific. No
console or file-stream implementation is added. KCC stdio.h now
declares const format pointers and snprintf/vsnprintf correctly.

Correct KCC limits.h signed 36-bit INT_MAX/LONG_MAX from
0343777777777 to 0377777777777, with corresponding minima. This
also repairs the formatter's width/precision overflow boundary.
The selector recognizes MOVEI function-address relocations and
selects the same versioned member name from the actual compiler's
directory. It does not import hosted DAIMOS libc or define stubs.

Final original 54-cell replay: no link failures, 34 raw PASS,
20 run_fail. Only 32 of the passes establish correct formatting:
920726-1.c, 960327-1.c, 980605-1.c and the stack-tenargs frame
companion each 8/8. All twelve 930513-1.c cells fail at runtime
because %.0f is unsupported; all eight variadic-manyargs cells
reach exit(41) (formatter returned -1 for %f, after va_arg checks).
The two struct-ret-1.c cells compare two output strings while
ignoring sprintf's return. The engine rejects %g at the first
conversion and terminates the partial buffer. These raw PASS
cells are not accepted as evidence of correct floating formatting
or struct argument readback. Preserve the historical oracle;
add exact full-string and length assertions in a future companion
when implementing real floating formatting. No changes to the
original sources or runtime acceptance limits.

New boundary regression 22/22 PASS: header limits, maximum and
minimum signed and unsigned decimal output, indirect sprintf,
bounded octal output, precision overflow and INT_MIN width.
The existing comprehensive buffered formatting oracle passes
8/8 KCC PDP-6/KA10/KI10/KS10 opt/noopt combinations, including
native va_list forwarding, flags, width, precision, truncation,
9-bit character output and explicit unsupported float errors.
The unchanged GCC v2 suite passes 16/16. Both archive extraction
checks PASS; KCC negative link without the formatter fails as
expected. Selector 34 host controls and common runtime parity
audit PASS. Historical cohort plus acceptance: 100 guest cells,
80 raw PASS, of which two weak historical PASS; 20 remaining
runtime failures. No general floating-point formatter claim.

Original 404 link cohort now 162 confirmed runtime PASS, 24
invalid registrations corrected, 218 unresolved/unverified
(including the 22 incomplete floating-format cells). Original
sweep unchanged. Next implement real portable D-format %f/%g
with independent numerical and formatting oracles; the existing
DAIMOS %.0f helper is bounded and cannot alone cover this scope.
Keep the memory-only interface distinct from console services.

Evidence: `$HOME/tmp/pdp10-sprintf-v59u.sgVSpWgK`, replay-v1/v2,
boundary-v2, stdio-v2.log, gcc-guards-v2.log, build-integrated-v2.log.
Installed incremental archive install-v2/local.tar.gz SHA256:
`cda75c91b54622e7cc325125b747c21a85889a62742bfbd17d3fbc73a5a8c471`.
KCC installed libp10stdio-v2.a SHA256:
`524e3c10b7c711880396e73af1bdd10c8eecb51a01ba37db4aea48749f916fb8`.
Shared source SHA256 (both copies):
`d01247c3a9989e2c71e06839a10a247c7db4b1cb666bb238d3fd3e8dedb0cb66`.
