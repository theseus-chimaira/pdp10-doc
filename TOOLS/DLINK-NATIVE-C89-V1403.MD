# Native DLINK DOBJ1 and C89 execution validation v1403

Date: 2026-11-09.  This is a completed linker correctness test,
**not** an acceptance of native DAS relocatable-object emission.

The accepted v1401 host KCC compiles a new defined C89 fixture,
`int main(void) { return 7; }`, to PDP-6 assembler source.  Host DAS
with `-C` produces `main.dobj`; existing CRT0 and syscall DOBJ1
objects are separately staged.  Host DLINK independently links
these inputs to a golden DXR at base octal 020.

The complete grouped D6FS image replaces only the five KCC and
three DAS binaries in the normal 237-file distribution with the
already tested v1401/v1402 versions.  The normal resident DLINK is
unchanged.  The three DOBJ1 inputs, C-SIX-encoded C89 fixture, and
independent host-linked DXR are imported as test-only files.  The
filesystem reports CLEAN before the isolated simulator boot.

All 5 strict PDP-6 probes pass.  Native KCC compiles the C89 source
to packed ASCII7.  Native DAS reads this source directly with `-A`
and builds a standalone DXR.  Native DLINK reads the independently
host-produced DOBJ1 inputs and emits a final DXR, which guest CMP
confirms is exactly equal to the host-linked image.  The guest then
executes this linked DXR and observes a process exit status of 7.

Thus the native linker and its observed relocation/entry/runtime
behavior are correct for this program.  The link input does not
yet originate from native DAS `-A -C`: the host object writer holds
dynamic global-symbol and relocation metadata, and the native
DAS1/DAS2 serialization does not transfer that complete state.
Full on-target object assembly and self-hosting therefore remain
open, requiring a proper bounded-memory native DOBJ writer rather
than any intermediate host conversion.

The reproducible regression lives at `daimos-testkit/tests/
validation/daimos-native-dlink-c89-v1403` and uses disposable
versioned images only.  There was no installation or replacement
of any production toolchain component.
