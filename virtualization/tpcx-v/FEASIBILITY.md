# Consolidation module (ROADMAP T4), derived from TPCx-V: feasibility

Status: feasibility study, 2026-11-19. Nothing has been built or run. Every figure below is from the kit's own documents
or the TPCx-V specification (revision 2.1.10) unless it is marked **(estimate)** or **(inferred)**. The kit and the
specification are kept outside this repository (SPEC.md section 6). Clause numbers point into the specification.

> This workload is derived from the TPCx-V Benchmark and is not comparable to published TPCx-V Benchmark results, as
> this implementation does not comply with all requirements of the TPCx-V Benchmark. (Disclaimer wording from TPC
> Policies v6.20, 8.1.5; see "Labelling" below.)

## Verdicts

| Question | Verdict | Main reason |
| --- | --- | --- |
| A compliant TPCx-V configuration (1 Tile, 50 Load Units, kit as provided), on the 32 GB Ryzen desktop | **not feasible** | 13 VMs at the User's Guide minimum of 4 GB each need 52 GB. The database disks need about 920 GB, and E: has 117 GB free. |
| A reduced, derived run (1 Tile, 10 Load Units, undersized VMs, deviations disclosed) on the same desktop, bare-metal Linux + KVM | **feasible**, once a dedicated NVMe disk (1-2 TB) is added and Linux is installed on it | It fits in about 30 GB of RAM (estimate) and about 185 GB of disk. CPU is fine, because consolidation is meant to overcommit. |
| The same reduced run under Hyper-V on Windows (no Linux host) | **feasible**, but second choice | Hyper-V is a commercially available VMMS (clause 1.1). It brings Windows host noise and a different I/O stack from KVM on x86 and Z. |
| Nested KVM inside a Linux VM on Hyper-V, or inside WSL2 | **not feasible** for results; usable only to smoke-test scripts | Two layers of virtualisation would dominate what is measured. |
| A run on Linux on Z under KVM or z/VM | **not feasible today** | No platform we have permits benchmarking (docs/platforms.md Q2). The code has porting risks (below), and the EULA questions below are open. Technically it can be ported: the kit is source, Java plus C++, with no x86-only build flags. |
| The lighter consolidation test proposed below (our own workload, N tenants at fixed service levels) | **feasible** on x86 now (bare metal or Hyper-V), and on Z as soon as any Linux guest platform permits benchmarking | It needs no TPC material, and its scale is chosen to fit. |

**Recommendation:**
- Build the lighter consolidation test first (C1 below). It answers the consolidation question on both architectures with
  no EULA exposure.
- Keep the derived from TPCx-V run as a secondary x86 cross-check (C2), once a 2 TB NVMe disk is fitted to the PC.
- Treat a derived from TPCx-V run on Z as a stretch goal. It needs a permitted platform and answers from the TPC on the two
  EULA points below.

## 1. What the kit contains, and what else is needed

**Verified, by reading the extracted kit** (`E:\tpc\TPCx-V\TPCx-V_Benchmarking_Kit_v2.1.10`, from the 3.7 MB TPC
download):

| Component | What it is |
| --- | --- |
| `VGen/` | C++ data and transaction-input generator (its input files are the TPC-E EGen flat files: Company, Security, names and so on), plus `create_TPCx-V_flat_files.sh` |
| `VDb/pgsql/` | PostgreSQL DDL, 19 transaction scripts in PL/pgSQL (34 `language plpgsql` functions), load, index and analyse scripts, and the legacy 2.1.9 User's Guide (HTML) |
| `VDriver/` | Java (RMI) and C++ (JNI, ODBC) driver: prime driver, customer emulators (VCE), market exchange emulator (VMEE), data maintenance (VDM), the Tier A `vconnector`, the reporter (Executive Summary) and `runme.sh` |
| `xVAudit/` | Java self-audit tools (atomicity, consistency, isolation, RI, cardinality, schema) and a PostgreSQL JDBC jar |
| `config/vcfg.properties` | Run configuration (the 4-Tile reference testbed) |
| Docs | `README.md`, `INSTALLATION_GUIDE.md`, `BENCHMARK_RUN_GUIDE.md` (2.1.10, RHEL 10.2 / JDK 21 / PostgreSQL 16) and an `EULA.txt`/`.pdf` in each component |

The specification PDF is **not** in the kit. It was downloaded from tpc.org (4.9 MB) to `E:\tpc\TPCx-V\spec\`.

**Not in the kit, and needed:**
- **Hypervisor.** The kit assumes VMware vSphere ESXi 9.2 plus vCenter (its clone script, `CloneVMs.ps1`, is PowerCLI).
  KVM, Hyper-V or z/VM provisioning has to be written by us.
- **Guest OS.** RHEL 10.2, or AlmaLinux / Rocky Linux 10, on every VM.
- **Packages:**
  - PostgreSQL 16 on the Tier B VMs;
  - OpenJDK 21;
  - unixODBC and psqlODBC (`psqlodbcw.so`) on the Tier A VMs;
  - GCC 14 (C++17), ClusterShell (EPEL), chrony and NFS.
- **VM images:** none are shipped. The 2.1.9 guide mentions an OVF template on tpc.org, but it is for RHEL 7 /
  PostgreSQL 10 and is not needed.
- **No pre-built binaries.** VGen, VDriver and xVAudit are built from source on the driver VM (`make`).

**Kit inconsistencies found** (verified by reading; each needs checking before a run relies on it):
1. `BENCHMARK_RUN_GUIDE.md` says the minimum for a TPC submission is "51,200 total LU" and that `VM_TILES` must be 4.
   Clauses 2.4.1.3 and 4.3.4.1 give a minimum of 50 LU per Tile, and allow 1 Tile for 50-1,000 LU. The spec governs
   on this point.
2. The run guide's table says `NUM_VMEE_PROCESSES` must be 1, but the shipped `vcfg.properties` sets 5.
3. The README refers to `CHANGES.md` and `LICENSE-TPCx-V-modifications.txt` ("the terms governing AMD's
   modifications"). **Neither file is in the download**, so the terms of the 2.1.10 modifications are unknown.

## 2. Hypervisor and guest assumptions

- **Specification:**
  - The VMMS must be "commercially available". It may not be static partitioning, whether set at boot or by operator
    intervention.
  - All I/O must be virtualised.
  - The configuration is one physical Node under one VMMS (clause 1.1, the VMMS and Virtualization Environment
    definitions).
  - The DBMS is PostgreSQL and the OS is RHEL, with newer supported versions allowed (clause 1.5).
- **What qualifies (inferred from the definition):**
  - KVM (RHEL/Alma), Hyper-V and z/VM qualify.
  - PR/SM LPARs alone probably do not, because they are static partitions defined in activation profiles.
- **The kit itself:** vSphere, RHEL 10.2, JDK 21 and PostgreSQL 16.13 (README, "Tested Environment").

## 3. Minimum scale

**Topology (verified, clauses 1.1, 4.3.4, 2.4.1):**
- A Tile is 4 Groups, and each Group is 3 VMs: Tier A (the `vconnector`) and two Tier B PostgreSQL databases (VM2 and
  VM3, each loaded with the Group's data).
- The minimum is 1 Tile = 12 SUT VMs, plus a driver.
- Groups 1-4 carry 10/20/30/40% of the load, and the share shifts across 10 Phases of 12 minutes.

**Data (verified):**
- 1 Load Unit (LU) = 1,000 customers. Every database has at least 5,000 customers (clause 2.4.1.2).
- The minimum is 50 LU per Tile, split 5/10/15/20 across Groups 1-4, in increments of 10 LU (clause 2.4.1.3).
- Initial Trade Days are 125 (clause 2.4.1.6).
- Nominal throughput is 2.00 Trade-Result/s per LU (clause 5.6.8.4), so 50 LU is 100/s.
- The measured rate must be 80-102% of nominal (clause 5.7.1.2).

**Run (verified):**
- Ramp-up is at least 12 minutes. The Measurement Interval is exactly 2 hours, covering one full 10-Phase cycle
  (clause 5.6.5.1).
- 90th-percentile response-time limits are 1-3 s per transaction type (clause 5.5.1.2).
- One run is therefore about 2 h 25 m, plus database restore between runs.

**Resources** (the User's Guide, section 2.1, gives a minimum VM of 2 vCPU, 4 GB and a 6 GB root disk; disk sizes are
from its tables, measured on PostgreSQL 10):

| Configuration | VMs | RAM at guide minimum | RAM, squeezed (estimate) | Database disk (incl. backup space) | Flat files | Nominal rate |
| --- | --- | --- | --- | --- | --- | --- |
| 1 Tile, 50 LU (smallest compliant shape) | 12 + driver | 52 GB | ~30 GB, Tier B at 2 GB | ~860 GB | 60 GB | 100/s |
| 1 Tile, 10 LU (derived; below the 5,000-customer minimum) | 12 + driver | 52 GB | ~30 GB | ~172 GB | 12 GB | 20/s |

**The squeezed budget (estimate):**
- host 2 GB;
- driver VM 4 GB, with fewer VCE/VMEE processes than the reference;
- Tier A 4 x 2 GB, one `vconnector` each (the guide starts each with `-Xmx1600m`);
- Tier B 8 x 2 GB (`shared_buffers` around 512 MB).

That totals about 30 GB, so the databases (7-144 GB each at 50 LU) run far beyond their RAM and are I/O-bound.
Disclose it as a deviation.

**This desktop (Ryzen, 8 cores / 16 threads, 32 GB; disks C: NVMe 12 GB free, E: NVMe 117 GB free, D: SATA HDD 423 GB
free):**
- **CPU:** 14-26 vCPUs on 16 threads is an overcommit of up to 1.6x. That is what a consolidation test exercises, so
  it is not a blocker. Whether 100 Trade-Result/s (50 LU) is reachable is unknown until measured; 20/s (10 LU) very
  likely is **(estimate)**.
- **RAM:** the binding constraint. 50 LU at guide sizes does not fit; either configuration fits only squeezed.
- **Disk:**
  - neither configuration fits in E:'s free space;
  - D: is a slow HDD, and SPEC labels it a slow-disk row only;
  - so add a dedicated 2 TB NVMe disk (this also keeps the benchmark data off E:, which other benchmarks use).
- **Hosting choices:**
  1. **Bare-metal Linux (AlmaLinux 10) + KVM/libvirt, on the new disk (dual boot).**
     - Best fidelity, and the same hypervisor family as KVM on LinuxONE.
     - SPEC section 2 already lists "x86 PC on bare-metal Linux" as a phase 1 platform. **Preferred.**
  2. **Hyper-V on Windows 11 Pro.**
     - Works without a reinstall (VMs created with `New-VM`, differencing disks).
     - Windows background load and Defender scanning add noise, which has to be disclosed.
  3. **Nested KVM** (in a Hyper-V Linux VM, or in WSL2, which is Hyper-V underneath).
     - Supported on AMD with Windows 11, but it measures nesting. Only for script smoke tests at toy scale.
- **Quiet machine:** every measured run needs the PC otherwise idle for about 2.5 hours per run, and SPEC 4.1 asks for 5
  repeats per point: about 12-15 hours of quiet time per configuration point, plus loads.

## 4. Linux on Z (KVM or z/VM): what would need porting

- **No x86 binaries.** Everything builds from source. A grep of the build files found no `-m64`, `-march` or x86
  intrinsics: VGen and VDriver use `g++ -std=c++17 -fPIC -O -fpermissive`.
  - The Java parts (RMI) are portable.
  - PostgreSQL 16, OpenJDK 21, unixODBC/psqlODBC and GCC exist for s390x in RHEL **(inferred; check against the RHEL 10
    for IBM Z package list)**.
- **Risk 1, verified in source:** the C++ components exchange raw C++ structs over TCP:
  - `SendToMarketImpl.cpp:136` and `TxnHarnessSendToMarket.cpp:72` send `sizeof(TPCE::TTradeRequest)`;
  - `MeeListener.cpp:169` receives it;
  - `MarketFeedDriver.cpp:189` and `TradeResultDriver.cpp:176` send requests the same way.

  Byte order and struct layout must therefore match between the driver VM and the Tier A VMs. A big-endian s390x SUT
  driven from an x86 driver would break. Either run the driver on s390x too, or byte-swap, which is a kit
  modification.
- **Risk 2, inferred:** plain `char` is unsigned on s390x and signed on x86. Legacy code built with `-fpermissive` may
  depend on it.
  - **The check:** generate the flat files on both architectures and compare them byte for byte, the way
    `analytics/tpch/scripts/verify_kit.sh` does for dbgen.
  - Identical files mean the generator ported. Different files mean it did not.
- **Risk 3, verified:** the 2.1.10 README describes "three critical C++ bug fixes" in `Socket.cpp` for RHEL 10. That
  shows the socket layer is fragile on new platforms.
- **Hypervisor on Z:**
  - z/VM is a commercial VMMS.
  - KVM on IBM Z (RHEL) runs in an LPAR we would need to control.
  - Whether KVM can run inside a z/VM guest is **not verified**. Ask the platform provider.
  - A shared z/VM guest has no hardware counters (docs/platforms.md Q4), so CPU would come from OS accounting only.
- **Terms:** no platform we have permits benchmarking. The Community Cloud forbids it (docs/platforms.md Q2), and the
  other routes need written permission first.

## 5. EULA and policy points (verified by reading `VDb/EULA.txt` and TPC Policies v6.20)

1. **Clause 3:** the software may be used "only ... on computer systems under Your direct control". Whether a donated
   or cloud LinuxONE guest counts is **an open question for info@tpc.org**. Until it is answered, run only on hardware
   we control.
2. **Clause 4(a)(iii):** no submitting or contributing "the Materials, or any part thereof, to any open source software
   project" without the TPC chair's written consent.
   - This repository may hold only our own scripts.
   - **Patches to the kit (porting fixes, byte-swapping) cannot be committed here either**, because a diff carries kit
     code. They stay under `E:\tpc\` or need TPC permission.
3. **Clause 4(b):** results may be disclosed only:
   - as a TPC Benchmark Result;
   - as academic research that "does not imply or state a marketing position"; or
   - with TPC written permission plus a non-comparability statement.

   SPEC section 6.4 already takes the academic route.
4. **Policies 8.1.4 / 8.1.5:**
   - public material must prefix the benchmark name with "Derived from";
   - it must carry the disclaimer quoted at the top of this file;
   - it may not use the primary metric;
   - using the benchmark's name in relation to derived work requires the TPC's written permission.

   **This is wider than SPEC section 6's wording** ("TPC-x-derived; not comparable to published TPC-x results"). It
   affects every TPC-derived track, so it is listed for review (Choices, item 3).

## 6. Effort and risk

| Work | Effort (estimate) | Risk |
| --- | --- | --- |
| Add a 2 TB NVMe disk; install AlmaLinux 10 + KVM (dual boot) | 0.5-1 day + hardware (about CAD 150-250) | low |
| Template VM, then 13 clones via `virt-install`/cloud-init (our replacement for `CloneVMs.ps1`); hosts, SSH, chrony, NFS | 2-3 days | low |
| Build VGen/VDriver/xVAudit; generate flat files; load 8 databases (10 LU) | 1-2 days | medium: the kit's own docs disagree (section 1) |
| First complete `runme.sh` run, then debugging | 3-5 days | medium-high: a distributed Java/RMI/C++ kit with many ports and 2.1.10 socket fixes |
| Harness: manifest, deviation register, FDR from the kit's run report, compliance checks (HARNESS.md, COMPLIANCE.md) | 4-6 days | low |
| **x86 derived from TPCx-V module, total** | **about 3 weeks** | medium |
| s390x port: build, flat-file byte comparison, driver on s390x, byte-order fixes kept out of the repo | +2-4 weeks, once a platform exists | high: platform access, EULA clause 3, unknown generator portability |
| Lighter consolidation test (below), x86 and s390x | 2-3 weeks of harness once the M1 Java track exists | low-medium |

## 7. A lighter test that answers the same question (proposed: C1)

**The question:** how many database-plus-application tenants one host runs at a fixed service level, on LinuxONE
against x86, with load shifting between tenants.

The lighter test answers it without TPC material:
- **Tenant:**
  - one VM (or container, as a separate row) running PostgreSQL plus our own application process with the online-track
    mix (the M1 Java track);
  - each tenant has a fixed data size (e.g. S = 1, about 1-2 GB) and a fixed RAM allowance (e.g. 1.5 GB).
- **Load:**
  - open loop (SPEC 4.2), with a fixed offered rate per tenant;
  - per-tenant shares follow our own published phase schedule (step and skew changes every N minutes, with the total
    held constant), so the hypervisor's re-balancing is exercised;
  - the schedule is our own design, not the TPCx-V Phase table.
- **Service level:**
  - per tenant, p99 latency at or below L ms and zero errors, in every phase;
  - the host passes at N tenants only if **every** tenant passes in **every** phase.
- **Metric:** the largest passing N, per host and per core, with CPU per transaction, steal time and the p99 of the
  worst tenant. Repeats and manifest follow SPEC section 4.
- **Platforms:**
  - KVM on bare-metal x86;
  - Hyper-V (optional row);
  - cgroups containers (a no-hypervisor row);
  - KVM on LinuxONE and z/VM, wherever terms permit.

  N can be scaled to fit 32 GB (e.g. N = 4-16 tenants of 1.5 GB).
- **Before any run:** a pre-registered hypothesis (SPEC 4.7) is needed. HYPOTHESES.md has none for consolidation yet.

**Why it can go first:**
- no EULA exposure, no "direct control" question, and no 920 GB of disk;
- it runs on any permitted Linux-guest platform;
- it reuses the online track's driver and golden checks.

**What it does not give:** recognition as a standard workload. That is what the derived from TPCx-V cross-check (C2) adds on
x86.

**What would make the full derived from TPCx-V module feasible, in order of value:**
1. A 2 TB NVMe disk in this PC, which is enough for a 10 LU derived run and close to the 50 LU shape.
2. A 64-128 GB x86 server or cloud bare-metal host for the compliant 50 LU shape at guide VM sizes (docs/platforms.md
   lists m7i.metal-24xl at about $4.84/h on demand). A 10-run campaign is roughly 30 instance-hours **(estimate)**.
3. A LinuxONE LPAR with KVM, or a z/VM system with permission to benchmark and publish, plus TPC answers on EULA
   clauses 3 and 4(a)(iii).

## Choices I made
1. **Kit location:** extracted to `E:\tpc\TPCx-V\` as briefed. The spec PDF is at `E:\tpc\TPCx-V\spec\`. Nothing from
   either is in this repository; clause numbers and short definitions are paraphrased.
2. **Verdict split:** each verdict is split by configuration rather than given as one yes/no.
3. **Policies v6.20 wording flagged:**
   - "Derived from" and the 8.1.5 disclaimer are used here, as well as the SPEC wording.
   - Whether SPEC section 6 should adopt them, and whether to write to the TPC for name permission, is for the
     maintainer. It is not decided here.
4. **Order of work:** the lighter test (C1) is recommended before the derived from TPCx-V run (C2). This is a proposal for
   the maintainer, not a decision.
5. **Squeezed VM sizes:** the RAM figures (2 GB Tier B, 4 GB driver) are my estimates, not measurements.
