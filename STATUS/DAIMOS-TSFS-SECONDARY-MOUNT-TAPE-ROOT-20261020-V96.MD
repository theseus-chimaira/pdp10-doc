# DAIMOS secondary TSFS mount on tape-root system — V96

**Historical result, superseded 2026-10-22 (V165):** This experiment
does **not** describe a supported filesystem interface. TSFS is
accessible only as the boot-root filesystem; post-boot MOUNT.TSFS was
retired and its dependent tests were removed. Image creation and
DECtape writes belong in userspace tooling and `dd`. The historical
observations below are preserved for diagnostic provenance only.

## Scope

The non-size functional acceptance of an independently mounted TSFS volume
was reproduced under a 256K PDP-6 SIMH boot. The OS booted from a TSFS
root on DTC0 and attached a separate three-member TSFS set on DTC4–6.
Strict 32K and permanent-size testing were **not** performed and remain
deferred.

The secondary set was built with `mktsfs -n 3 -i 1:2345 -g 7`, using a
small manifest with `/README`, `/DOC/INDEX`, and `/SYSTEM/EXEC/HELLO`.
The bootable root was also generated with `mktsfs`, keeping the normal
DAIMOS userspace programs and adding `/MOUNT/DT0` and `MOUNT.TSFS`.

## Observed result — 3/3 PASS

The v96 SIMH harness successfully executed:

1. `MOUNT.TSFS /DEV/DTC4 /MOUNT/DT0` — PASS; no `MOUNT FAILED` output.
2. `STAT /MOUNT/DT0` — PASS; read-only directory `TYPE 1`, `WORDS 0`,
   `MODE 0555`.
3. `LS /MOUNT/DT0` — PASS; `README` and `DOC` entries present.

The harness's authoritative three-probe report is preserved at
`$TMPDIR/daimos-tsfs-tape-root-v96/report-v96.md`; serial simulator output
is in `$TMPDIR/daimos-tsfs-tape-root-v96/run-v96/dcs0.log`.

## Interpretation and remaining acceptance work

The mount succeeds with the existing tape-root provider, without making
TSFS permanently resident in a disk-root kernel. The original group 027
`test-daimos-tsfs-mount` and `test-daimos-tsfs-dir` fixtures boot from D6FS,
where the optional TSFS provider is not loaded; their failure is **not**
evidence that secondary TSFS mounting on a supported tape-root system is
broken.

V96 verifies mounting, mounted-root metadata, and root-directory listing.
It does not yet replace the complete original directory/stat oracle with
all of its nested-file checks, nor does it establish atomic group-027
acceptance. The reusable tape-root test fixture and full group rerun remain
pending. Other failing non-size functional groups remain separate.
