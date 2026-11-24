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

## Extended ISA verification (v56c)

Testkit `fdc2d3d` adds `kl10b-global-isa-v56c`. The test passes 92
instruction assertions and 20 physical memory observations using
independently constructed architectural words. All 26 one-word
global byte pointer selectors are exercised: five before-first
selectors and 21 data-slot selectors across widths 6, 7, 8, 9 and
18 bits. LDB/DPB preserve adjacent slots and trailing unused bits.
IBP and signed ADJBP cross both word and section boundaries; the
transition from section 2 offset 777777 to section 3 offset zero
preserves the selector and accesses memory, not accumulator zero.
XMOVEI returns the current-section and global indirect addresses.
Forward and reverse XBLT copy two words between sections 2 and 3,
with count exhaustion, final addresses and destination words checked.

Run `./run-misc-tests.sh run-id kl10b-global-isa-v56c`. Both registered
KL10B regressions pass. This completes the initial independent
execution milestone EA01, not the extended compiler test matrix.

## DAS baseline (v56c)

Using an absolute address 2,,1000, the installed DAS binary SHA256
2a22c152088a9cee6d3085ce787c0796d46b1b9e903217dc760b343c4997cfa1
emits incorrect words for 21 data-slot selectors and rejects all
five before-first selectors (0/26 correct). The nine-bit first-slot
example emits 331100001000 instead of 700002001000. Results and
source hashes are recorded in the testkit manifest
`PDP10-DAS-OWGBP-BASELINE-20261124-v56c.tsv`. This is observed
assembler output, not a compiler runtime failure count.

The next stage must repair selector encoding and preserve 30-bit
symbolic address relocation without modifying DAIMOS DXR formats
or adding native resident state. Existing local-address objects
and the DAIMOS loader remain outside this extended-address work.

## Independent KLH10 execution (v56e)

KLH10 fork `theseus-chimaira/klh10`, source commit `a97d836`, builds
both KL10-extend/T20 and KS10/T20 on the aarch64 Termux host.
The portability changes use POSIX SEEK_SET in place of L_SET and
enable libpcap only when the required headers and library exist.
The build links with the installed Termux libandroid-shmem library
(`LIBS=-landroid-shmem`) and disables bridge networking. Existing
upstream warnings remain; network/device operations are not tested.

The versioned distribution `klh10-build-20261124-v56e/local.tar.gz`
under TMPDIR (SHA256
2745b67365429c44c289e093c1e27e13cc8465379e319e00ce457b91cccb1871)
is installed into PDP10_PREFIX without replacing SIMH. Versioned
`bin/kn10-kl-v56e` and `bin/kn10-ks-v56e` have unversioned aliases.
The matching device subprocesses and utilities are kept separately
in `libexec/klh10-v56e/kl` and `libexec/klh10-v56e/ks`; for device
configurations use the appropriate working directory or dppath.

Testkit `013851c` registers the optional KLH10 runner. The same
architectural input images from v56b/v56c are replayed with KLH10
physical deposits/examines and its own HALT report. Both local CPU
smokes pass, both KL10B section controls pass, and all 92 global ISA
assertions and 20 physical words match. The checks require nonzero
success sentinels and the correct termination state, not host exit
status alone. Run:

```sh
./run-misc-tests.sh run-id klh10-isa-v56e --manual
```

KLH10_KL and KLH10_KS may override the executable paths. This is
additional independent ISA coverage, not yet an alternative C matrix
runner. Full extended relocation/linking and compiler acceptance
remain open.
