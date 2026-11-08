# v1372: convert native KCC packed ASCII7 to canonical C-SIX/S6REC

Date: 2026-11-08. Status: **host and native PDP-6 conversion PASS**.
The canonical representation remains the C-SIX transcode already
documented in `KERNEL/SELFHOSTING.MD`, not an additional ASCII file
format. Native SIXBIT source files are regular editable S6REC records.

The host converter now accepts:

    CSIX -E -P INPUT.PACKED7 OUTPUT.S6REC

The option `-P` selects KCC's five 7-bit ASCII characters per 36-bit
word. The source is read from the standard eight-byte host container
for each PDP-10 word. The lowest physical bit is unused, as in native
KCPP's packed-ASCII decoder. After unpacking, the original C-SIX
encoder applies exactly the same case/punctuation escaping,
eight-column tab expansion, CRLF normalization, strict final newline
and S6REC line framing as ordinary `CSIX -E`. `CSIX -D` is unchanged.

The versioned native C89 `userland/toolchain/csix_v1372.c` provides
the test-only guest command:

    CSIX INPUT.ASCII7 OUTPUT.S6REC

It uses 16 input words and the standard native S6REC output sink,
without a whole-file buffer or a second assembler parser. Malformed
input and interrupted conversions fail, removing the incomplete
output. Native image is octal **002700** words, BSS **000106**.

The v1372 daimos-testkit runner verifies the exact 36-bit native
output words via guest `CMP` against the host reference, confirms
native `CAT` reads the C-SIX escapes, and verifies native KCC can
consume the same output. Host tests compare both CSIX encoders
byte-for-byte, decode back to original C89 source and reject bad
padding, missing final newline and truncated word containers.
Additional native and host CRLF/tab fixtures verify the same encoded
36-bit words and eight-column tab normalization.
All eight native probes and host converter checks PASS.

**Source-format distinction:** C-SIX is syntax-blind, reversible
storage for C source and headers. Native DAS still expects regular
uppercase SIXBIT/S6REC **assembly**, not C-SIX-escaped input.
The separately documented `ASCII2SIX`, `SIX2ASCII` and `S6FILTER`
provide conventional character-line and assembler-text conversion;
they are not replaced by this C-SIX extension. There is no DAS
memory budget or decoder change.

**Compiler blocker:** native KCC currently emits malformed assembler
instructions (`FMPL 017,0` and a missing `MOVEI` mnemonic in a
minimal return-value C fixture), whereas host KCC emits the correct
`MOVEI 1,012345` / `POPJ 17,` sequence. This occurs before the
converter and with or without native `-O`. Native compiler-to-DAS
correctness and full self-hosting remain unaccepted until fixed.
