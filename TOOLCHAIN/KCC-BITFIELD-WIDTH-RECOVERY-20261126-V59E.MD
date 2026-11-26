# KCC malformed bit-field width recovery (v59e)

KCC d708dc9; testkit 434482f7. The November 25 sweep contained
66 KCC crashes across `991118-1.c`, `bf64-1.c`, and `920501-3.c`.
The front end diagnosed unsupported widths and types but used the
original width in both member layout and type construction. Widths
larger than a word reached sizetype, which divided by a zero count
of bytes per word. Named zero widths and widths truncated to host
int also need consistent recovery.

Validate the full INT constant in sdeclaration before narrowing
or invoking fldsize. Recover invalid named/unnamed widths with a
representable one-bit placeholder after issuing a source diagnostic;
zero remains legal for an unnamed alignment field. The member type
and layout now use the same validated width. This is diagnostic
recovery only, not support for wide bit-fields.

All 66 former ICE cells now report ordinary compile_fail and pass
the existing KCC diagnostic classifier on their logs. These are
not semantic passes. The added negative fixture exercises width 37,
named zero, negative width, and named/unnamed 4294967297ULL widths,
including member accesses after the invalid declarations. The
baseline exits with SIGFPE; the repaired compiler reports five
source errors and normal rejection. All 22 negative matrix cells
pass. All 46 selected valid packed and full-word bit-field cells
pass; the negative-policy self-test passes all 17 cases.

Evidence: `$HOME/tmp/kcc-bitfield-recovery-v59e.zymJemMz`.
Installed incremental bin/kcc-only `local.tar.gz` SHA256:
`33cd8873df55b5cd4adc0a5122b13bcf25ae72bf43f5333513470b43a7a43e24`.
The full sweep is unchanged. No native compiler rebuild, extended
addressing acceptance or DAIMOS source change is claimed.
