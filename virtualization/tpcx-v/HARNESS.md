# Consolidation module: harness sketch (design only; nothing here is built yet)

This sketches two harnesses that share one layout, following the conventions of `analytics/tpch/`:
- **C2**, the x86 cross-check derived from TPCx-V;
- **C1**, the lighter consolidation test recommended first (FEASIBILITY.md section 7).

The conventions are:
- path variables in a git-ignored `config.env`;
- bash for host steps and standard-library Python for manifest, checks and FDR;
- one results folder per run.

**Nothing from the TPC kit is copied here.** Scripts point at the user's own extracted kit through `TPCXV_KIT`, and
any kit patch lives beside the kit, never in this repository (EULA clause 4(a)(iii); FEASIBILITY.md section 5).

## Layout (proposed)

```
virtualization/tpcx-v/
  FEASIBILITY.md         feasibility and verdicts
  HARNESS.md             this file
  COMPLIANCE.md          rule-by-rule map: what C2 implements, what it can't, and how each deviation is disclosed
  config.example.env     path and topology variables (copy to config.env, git-ignored)
  scripts/
    env.sh               loads config.env; checks TPCXV_KIT has VGen/ VDb/ VDriver/ xVAudit/; warns when WORK is on a 9P/DrvFs mount
    host_check.sh        host facts before anything starts: KVM present, nested off, SMT state, governor, free RAM/disk, other load
    provision_kvm.sh     template VM -> 12 SUT VMs + driver via virt-install/cloud-init; static IPs, /etc/hosts, SSH keys, chrony, NFS
                         (replaces the kit's vSphere-only CloneVMs.ps1; our own code)
    provision_hyperv.ps1 the same, with New-VM and differencing disks (optional row)
    build_kit.sh         on the driver VM: make VGen, VDriver, xVAudit from $TPCXV_KIT, unmodified; writes build-info.txt (compiler, flags, checksums)
    verify_gen.sh        portability check: flat files from this build vs. a reference build on another architecture, byte for byte
    gen_flat.sh          runs the kit's create_TPCx-V_flat_files.sh for the chosen LU; records sizes and checksums
    load_db.sh           runs the kit's VDb setup.sh on all 8 Tier B VMs (each Group's scaling_tables_dirs set); keeps setup.out
    snapshot.sh          after load: libvirt external snapshots or cpio backups of each /dbstore, so every repeat starts from the same state
    run.sh               one measured point: restore -> time-sync check -> runme.sh (kit) -> collect -> xVAudit -> checks
    campaign.sh          randomised order over points x 5 repeats (SPEC 4.1); refuses to start if host_check.sh fails
  py/
    common.py            LABEL / disclaimer strings, paths, JSON helpers (as analytics/tpch/py/common.py)
    manifest.py          SPEC 4.5 manifest: host (lscpu, memory, kernel, governor, SMT, libvirt/QEMU version) + per-VM (vCPU, pinning,
                         RAM, disk bus/cache mode, guest kernel, PostgreSQL settings) + kit build-info + git commit
    collect.py           copies the kit's run report (mix log, reporter output, audit_check.log, runtime.properties,
                         failed-transactions log, host stats) into results/<run-id>/raw/ with checksums
    checks.py            our pass/fail checks over the kit's output (COMPLIANCE.md "auto" rows): response-time limits,
                         average <= p90, mix ranges, 80-102% of nominal, per-Group 98-102% share, MI length, ramp-up, zero errors
    steal.py             host-side steal/ready time per VM (from /proc and virsh domstats), sampled once a second
    fdr.py               fills docs/FDR-TEMPLATE.md from manifest + checks + raw; every gap printed "TODO (operator)"
  tests/
    test_pure.py         unit tests for checks.py and fdr.py on synthetic logs (no kit needed)
```

## Run flow (C2, one point)

1. `host_check.sh` checks that the machine is quiet: no other benchmark, no backup or indexing, and Defender
   exclusions if Hyper-V. Its output goes into the manifest.
2. `snapshot.sh --restore` resets all 8 databases to the post-load state.
3. Before the run, the time-sync check: the kit aborts above its tolerance, and the spec asks for a check before and
   after.
4. `runme.sh -u -s`. It is always marked unofficial: `-u` is the kit's own flag for a non-compliant run.
5. `collect.py`, then `xVAudit` (all 11 reports), then `checks.py` and `manifest.py`.
6. Results go to `results/<date>-<platform>-tpcxv-derived-<LU>lu/run<k>/`. `fdr.py` runs over the repeats.

## Run flow (C1, the lighter test)

The same scripts handle provisioning, manifest, steal and the FDR. The workload comes from the online track's driver
(M1) instead of the TPC kit:
- `tenants.sh` creates N tenant VMs (or containers);
- `load_phases.json` is our own phase schedule;
- the driver runs open loop at a fixed rate per tenant;
- `checks.py` applies the service level (per-tenant p99 and zero errors, in every phase);
- `search.py` finds the largest passing N by bisection, then confirms it with 5 repeats.

No TPC kit is involved, so `TPCXV_KIT` is not needed and no TPC label applies.

## Labelling (C2)
- Every result file, chart and FDR carries the Policies 8.1.5 disclaimer and the SPEC section 6 label.
- Figures are named in our own words, for example "Trade-Result transactions per second, derived from TPCx-V, not
  comparable". The primary metric's name is never used, and no price-performance figure is produced.
- The kit's Executive Summary HTML is kept in `raw/`, unpublished. Because it is TPC-formatted output carrying the TPC
  metric name, the FDR summarises it instead.

## What needs a quiet machine or other hardware
- **A quiet machine for every measured run:** about 2.5 h per run, 5 repeats per point.
- **On the PC:** a dedicated 2 TB NVMe disk and bare-metal Linux for the preferred KVM row.
- **Elsewhere:** a server with 64-128 GB for the 50 LU shape at guide VM sizes, and a permitted s390x platform for the Z
  rows.
