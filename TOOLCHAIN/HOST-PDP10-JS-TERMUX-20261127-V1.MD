# Native Node.js KI10 installation on Termux

Installed pdp10-js a302500 under $HOME/cross on 2026-11-27.
Node.js v24.18.0 is sufficient; no npm dependencies or ahead-of-time
native compilation required. The host compiles the original CPU,
floating-point, I/O and boot scripts with vm.compileFunction at startup.

The installation contains the tracked checkout files in share/pdp10-js,
including the original browser front panel and diagnostics. The launcher
bin/pdp10-js resolves the adjacent share directory and executes node.js
with the supplied arguments. Run:

    pdp10-js --media-dir /path/to/images

Media defaults to the working directory. Ctrl-] exits an interactive
session. Disk writes remain cached and volatile.

All JavaScript files pass node --check. The existing host acceptance
test tests/host/pdp10-js-node-v1.cjs from daimos-testkit
origin/pdp10-js-node-v1 passes against both staging and installed copies:
bounded block reads, EOF, error, cache padding, printer callback, panel,
CTY HELP, missing disk recovery and synthetic RP03 bootstrap to HALT.
Installed launcher --help passes. Real TOPS-10 media is not present at
the documented media path; full OS boot and graphical operation have
not been tested on this host.

Package: $TMPDIR/pdp10-js-install-v1.cXYWrV1n/local.tar.gz
SHA256: 7777f7454e0cb30ba00023118a79b2cb61f658957418ee943a4db898c712e817

No emulator source changes. The pdp10-js checkout remains clean.
