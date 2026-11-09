# Native DAIMOS DAS packed ASCII7 input v1402

Date: 2026-11-09.  Scope: resident PDP-6 assembler input correctness,
not KCORE code-size optimization or production toolchain installation.

## Source format and interface

Host DAS has long supported ordinary byte-oriented ASCII source, but
the native DAS phase previously assumed every source file was S6REC.
KCC produces native assembly in packed ASCII7: five 7-bit ASCII bytes
at bit positions 29, 22, 15, 8 and 1 in successive 36-bit words.
The low bit is spare and must be zero.  A final short word may have
zero-padded character slots; embedded NULs and trailing words after
padding are rejected rather than interpreted as a shorter file.

The native DAS driver forwards `-A` to DAS1 to select this packed
word format.  `-S` and the default still use canonical S6REC.  The
new ASCII7 decoder uses the same buffered word source as S6REC; the
per-file reader state is a union, not two independent resident
buffers.  No intermediate conversion file or new kernel ABI exists.

Native lexical comparisons now accept lowercase and uppercase
assembler keywords, operations and directives, while preserving
literal ASCII data in quoted strings and exact source symbol
spelling.  The existing S6REC behavior is maintained.  The feature
does not implicitly select ASCII7 based on file content; `-A` is
required for native KCC assembler output.

## Native PDP-6 regression

`daimos-testkit/tests/validation/das-native-ascii7-v1402` rebuilds
all three native DAS binaries using the accepted v1401 host KCC and
copies the DAS source under test storage, not into production.  It
builds an isolated complete 237-file grouped D6FS root with those
three DAS images and the five corrected v1401 KCC phases; `d6fsck`
reports CLEAN.  The guest runs against cloned media.

17 strict probes pass: explicit ASCII7, explicit S6REC, default
S6REC, exact DXR host/native comparisons, native KCC-generated
lowercase ASCII7 assembled directly by native DAS with exact
host-produced DXR equality, mixed-case quoted literal equality,
and rejection of three malformed input forms.  A hand-written
ASCII7 standalone assembly program is assembled with native DAS,
compared to the host image and executed via DAIMOS RUN; its exit
status is 7.  Every expected success and failure is checked by
the guest's returned process status, not merely a transcript match.

This confirms the direct KCC -> DAS assembler-source boundary and
native DAS -> RUN path.  It is **not** proof that a fresh native C89
program has been linked and executed.  Native DAS's relocatable
`-C` output path, native DLINK integration, independent -N KCC
host equality and exact 96K self-hosting remain separate release
gates.  No production or installed toolchain image was modified.
