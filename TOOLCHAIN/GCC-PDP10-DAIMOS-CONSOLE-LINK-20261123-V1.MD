# GCC PDP-10 DAIMOS console link verification (2026-11-23 V1)

The installed GCC `libp10stdio-daimos-v1.a` provides `printf`,
`vprintf`, `putchar`, and `puts` separately from the OS-independent
`libp10stdio-v2.a` formatter. These libraries are optional; the
freestanding compiler does not automatically select DAIMOS I/O.

`pdp10-testkit/misc/compat/test-gcc-p10stdio-daimos-link-v1.sh`
compiles a real GCC client, assembles the current production
`DAIMOS/userland/libc/syscall.s`, and links both GCC archives
against that real WRITECHAR veneer. The linked symbol map must
contain `printf`, `vprintf`, `putchar`, `puts`, `dsys_writechar`,
and `__p10stdio_vformat_v2`. A negative control excludes the
monitor veneer and requires link failure. Both controls PASS,
including the registered testkit dispatcher invocation.

This is a **link-only** test. It does not establish GCC program
execution under DAIMOS, GCC/KCC varargs interoperability, or a
GCC DAIMOS process startup ABI. Existing KCC/DAIMOS libc guest
validation is a separate test. The standalone GCC arena allocator
remains explicitly initialized by its caller and is not selected
by this console test.

Inventory, raw-P10RUN and fixture-integrity audits PASS. The
script-ownership audit currently reports an unrelated existing
`misc/das/symbol-difference-v1817/run-v1453.sh` registration issue;
this new test does not use that script.
