# Termux emulator installation (2026-11-27)

Native aarch64 Termux, clang 21.1.8, prefix $HOME/cross.
Refresh apt indexes and install libpcap and sdl2-image.
All five SIMH PDP10 targets built from abb7ee6 with
make GCC=clang pdp6 pdp10-ka pdp10-ki pdp10-kl pdp10-ks.
All register sanity checks and installed p10run C guest smoke tests pass.

Original PDP6 b60b1e4: build in emu with CC=clang and
CFLAGS="-std=gnu99 -O2 -Wall -Wno-parentheses", targets
pdp6, pdp6_s, pdp6_n, pdp6_headless_v1.
Install graphical pdp6 as pdp6-aap to avoid replacing SIMH.
Headless startup/quit passes. Graphical/serial/network operation untested.

KLH10 8433c11: out-of-tree configure with CC=clang,
CFLAGS="-O2 -std=gnu99", --prefix=$HOME/cross. Build bld-kl,
bld-ks, bld-ks-its with LIBS=-landroid-shmem and
NETLIBS="-lpcap -landroid-shmem". Stage union of model and
auxiliary executables: upstream install moves previously installed
models aside. All three installed configurations start, allocate
shared memory, and quit successfully. ITS CHAOS parsing uses strchr.

KX10 6f4f552 plus pre-existing local port changes: out-of-tree
configure, CC=clang, build existing K&R source with
CFLAGS="-std=gnu89 -O2 -fcommon -I.". Command-line usage works.
Guest execution not certified; legacy compiler warnings remain.

TS10 7638d8f plus pre-existing local port changes: clean build in
src with CC=clang and CFLAGS="-std=gnu99 -fgnu89-inline -g -O2 -c
-DHAVE_SIGACTION -DDEBUG -fcommon". VAX conditional lvalue assignments
replaced with standard C and missing declarations added.
Recursive target is phony so existing executable cannot suppress
source rebuild. NET_FILE and NET_STDIO borrow descriptors; debug
stream owns its descriptor and closes it once. Prior executable
aborted under Android fdsan on quit; installed repaired executable
exits 0 in a terminal and via -f quit.ini. Testkit e9e5eb4 adds
manual installed TS10 debug stream shutdown regression. Standalone
dump10 built with a separate non-debug vtape object; usage verified.
TS10 nonterminal piped interactive input is not certified.
No full OS boot or device functionality claims.

All fourteen emulator executables and auxiliary programs installed
via three local.tar.gz archives under
$TMPDIR/emulator-install-v60d.mZ9KFgd7:
simh-pdp6-v1 (eb6133c0bd4e885af749c083f6fcf9bd07dcb3110bae3785088d74ab4bb8f0d6),
klh-kx-v1 (b7b2837f52768cc2a701dfed41200770780bdfaae9b5d655bc66ce1da7d99c9e),
ts10-v3 (7424a8df4b7376030a64234daf2f76973fd8cc35de408c6b58e02aef2b502eef).
Existing local source edits were preserved; only new repair hunks
were committed. No DAIMOS kernel/compiler changes.
