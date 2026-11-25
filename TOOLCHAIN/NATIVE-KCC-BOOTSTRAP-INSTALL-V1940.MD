# Native KCC bootstrap installation, v1940

The full PDP-6 image stages the native compiler from
$PDP10_PREFIX/lib/kcc/bootstrap. Host-only KCC installation does not
install these native executable images. In a freshly installed
cross environment the image can fail with "missing KCC native
bootstrap" even though the host compiler is operational.

On 2026-11-25 the existing KCC edff620 source successfully restored
the missing native bootstrap with:

    export PDP10_PREFIX=$HOME/cross
    export TMPDIR=${TMPDIR:-$HOME/tmp}
    export PATH=$PDP10_PREFIX/bin:$PATH
    cd "$HOME/git/kcc"
    make -j4 install-native PREFIX="$PDP10_PREFIX" \
        PDP10_PREFIX="$PDP10_PREFIX" DAIMOS_REPO="$HOME/git/DAIMOS"

This installs KCC.dxr, KCPP.dxr, KPARSE.dxr, KGEN.dxr and KOPT.dxr,
the native startup/ABI objects and DAIMOS libc archive under the
bootstrap directory. No compiler source change was required.
A subsequent current GCC lowmem tape image build succeeded and
the guest booted. This verifies installation and image staging,
not on-target self-compilation. Existing v1918 local.tar.gz has
not been regenerated; its missing native bootstrap is repaired
by the installation procedure above.
