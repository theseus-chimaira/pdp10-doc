# Independent KL10B section execution (v56b)

Verified on the installed Open SIMH KL10 simulator (a6b511e5).
This is a freestanding toolchain execution prerequisite, not a
DAIMOS loader or kernel implementation.

The testkit registered validation `kl10b-section-execution-v56b`
constructs independent architectural instruction words and direct
private writable page-table pointers. It selects KL10B, configures
256K physical words, and uses an executive page table at physical
page 2. Separate section page tables map virtual sections 0, 1, 2
and 3 to distinct physical frames. A section-0 bootstrap executes
CONO PAG,60002 and XJRSTF to 1,,1000. The guest itself enables
TOPS-20 paging; no simulator PC-section or paging-register override
is used.

The positive test observes:

- Section 1 global indirect MOVE reads section 2, offset 1000.
- One-word global nine-bit LDB and DPB access section 2. The
  selector is in the high six bits; the address is in the low 30.
  The physical packed data word becomes octal 123456000000.
- PUSHJ executes a section 2 function from section 1. A global
  stack in section 3 stores the full return PC, 1,,1015. POPJ
  restores section 1 and the original stack pointer, 3,,1000.
- The success HALT executes at 1,,1020, with register 6 equal
  to octal 765 and TOPS-20 paging still enabled.

The negative test deliberately redirects the first global indirect
word to the same offset in section 1. It reaches the failure HALT
at 1,,1037, leaves the success sentinel zero, and leaves the data
and poisoned stack slot untouched. Both tests pass their distinct
expected outcomes. The test checks executed instruction history,
registers and physical memory, not merely simulator exit status.

Run from pdp10-testkit:

```sh
export TMPDIR="${TMPDIR:-$HOME/tmp}"
./run-misc-tests.sh run-id kl10b-section-execution-v56b
```

The test accepts SIMH_KL as an executable override, otherwise uses
PDP10_PREFIX/bin/pdp10-kl. It retains unique versioned input and log
files under TMPDIR. Testkit commit 100c86f contains the test and
registration. No compiler, assembler, linker, simulator or DAIMOS
code was changed for this result. This validates only the stated
instructions and setup, not the complete KL10B instruction set or
extended-address compiler correctness. All-selector and boundary
tests and extended relocation support remain separate obligations.
