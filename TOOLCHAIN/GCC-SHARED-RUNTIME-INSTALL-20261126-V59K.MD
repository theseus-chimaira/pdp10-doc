# GCC shared freestanding runtime installation (v59k)

GCC fa9766b6 and 1a00978f; testkit e318bc16. The November 25
full sweep recorded eight GCC setup failures: 980707-1.c and
991112-1.c in all four native PDP-6 optimization profiles.
The GCC installation omitted the C-locale source and archive.
The installed-runtime auditor also identified missing FP36
and signed D-format sources and archives. KCC already owned
and installed all three.

GCC now carries byte-identical copies of the three versioned
KCC sources, following the existing shared freestanding source
arrangement. build.sh calls build-shared-archives-v1.sh to build
three independent archives and installs them through both
privileged and ordinary branches. Existing source wildcard
installation includes the new assembly members. The frozen
core libp10c-v2.a member list is unchanged. No new runtime
implementation, OS dependency or test-local stub is involved.

The test runner now reports the missing member diagnostic for
both compilers, not just KCC. The exhaustive C-locale oracle
initially revealed four GCC link failures: its current assembly
uses .word 0,,isprint and .word 0,,isspace. The selector now
recognizes numeric-left-half data relocations, respecting local
symbol definitions. Added host controls for these references
and the missing-GCC-member diagnostic.

Executed the actual freestanding installation block from build.sh
into a new empty staging prefix using installed bootstrap tools;
all four archive copies match KCC byte-for-byte. The existing
installed audit passed against the staged GCC library and copied
KCC reference, and against the live prefix after installation.
Audit negative controls, selector controls and runner accounting
tests pass. No full compiler rebuild or privileged installation
execution is claimed. The ignored builder filename was explicitly
tracked in the second GCC commit.

Final native PDP-6 replay: 18/18 C-locale cells (the two original
sources and exhaustive 512-character/EOF oracle, both compilers);
24/24 cells from four existing signed/unsigned FP conversion
regressions. All eight original setup failures now pass.

Evidence: `$HOME/tmp/gcc-shared-install-v59k.UUUoSyVj`.
Final replay-v2 and fp-replay-v1. The incremental six-file
local.tar.gz installed in PDP10_PREFIX has SHA256
`45e6e20b96ce09fe36b91bc6a1b89c5855a70fcb9a99fe229e84b8949c5da7a8`.
Core archive SHA256 remains
`ac09b8a362e4ea3ca8a7e69a2ca5a98af5a76cc85a7db909b6b3a134afb7ee41`.
Original sweep is unchanged. The new installed modules are
separate archives, not additions to the core compiler archive.
