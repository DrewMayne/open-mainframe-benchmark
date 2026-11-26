# Energy method: what we can actually measure

Status: draft for comment, 2026-11-19. Applies to the energy fields of an FDR and to the "per watt" normalisation in
SPEC section 3. See SPEC section 1.7.

Sources are marked "fetched" (page read) as in [platforms.md](../platforms.md).

## 1. Labels
- RAPL figures are labelled with their domain, e.g. "RAPL package + DRAM", because they are not system power.

## 2. What we do on x86 hardware we control

### 2.1 Counters and interfaces
- **powercap sysfs:** `/sys/class/powercap/intel-rapl:<n>/energy_uj` for each package, with core and uncore
  sub-zones. Each zone also has `max_energy_range_uj`, the counter's range (kernel powercap documentation, fetched).
  That document does not list DRAM or psys zones, so the sampler discovers the zones present at run time.
- **perf:** the `power` PMU events `energy-pkg`, `energy-cores`, `energy-ram`, `energy-gpu` and `energy-psys`.
  - The kernel driver extends the raw counters in software, with a periodic timer, so the perf counts do not wrap.
  - The events can only be counted system-wide (`arch/x86/events/rapl.c`, fetched).
- **The same powercap driver serves AMD and Hygon CPUs:** AMD families 17h, 19h and 1Ah, and Hygon 18h. For them it
  registers package and core energy only, with no DRAM and no power limits (`drivers/powercap/intel_rapl_msr.c`,
  fetched). The perf driver also enables only package and per-core events on AMD (`rapl.c`).

### 2.2 Procedure (proposed)
- **Sampling:**
  - A small sampler reads every available zone at the start and end of each measured window, and every 1 s in
    between.
  - The 1 s interval is our choice. It is far shorter than any counter wrap time (section 2.3).
  - The measured window is the run-rule window of SPEC section 4: warm-up excluded, whole batch cycles.
- **Energy:** the sum of successive deltas. A negative delta means the counter wrapped, and `max_energy_range_uj` is
  added back.
- **Per-transaction figure:** total energy in the window divided by committed transactions in the window. Batch
  steps report joules per job.
- **Idle baseline:**
  - The procedure and its parameters (start delay after the last run, duration, and which services stay up) are set
    before the first measured run and recorded in the FDR.
  - A pre-run idle reading is also taken, to show drift.
- **Repeats and power state:** energy is reported per run, then as mean and 95% CI (SPEC section 4). Governor, turbo
  and C-state settings are in the manifest (SPEC 4.5), since they change energy as much as throughput.

### 2.3 RAPL's limits (stated with every result)
- **Measured on some generations, modelled on others:**
  - Intel moved RAPL from a model to actual measurement with Haswell (Hackenberg et al. 2015, fetched); Sandy Bridge
    and Ivy Bridge values are modelled.
  - On AMD Zen 2 (EPYC Rome), Schöne et al. (2021, fetched) conclude that the data is modelled, not measured, and
    should be treated as inaccurate. In their tests the package domain read well below external measurement, and
    DRAM energy is not (fully) included.
  - We have not seen a study of newer AMD generations. Until one exists, AMD RAPL figures carry "modelled, accuracy
    unverified".
- **Partial coverage:**
  - The Intel domains are package, cores (PP0), client graphics (PP1), DRAM (server parts from Sandy Bridge-EP,
    desktop parts from Haswell) and, on some Skylake models, psys. Haswell-EP has no PP0 (Khan et al. 2018, fetched).
  - Even package + DRAM leaves out power supply losses, board voltage regulators, fans, storage, NICs, the chipset
    (unless psys covers it) and everything else at the wall.
  - RAPL energy is therefore a lower bound on system energy. It may not be compared with wall-power figures.
- **Counter width and wrap:**
  - The hardware counters are 32 bits wide, in model-specific energy units, e.g. 15.3 uJ on Sandy Bridge and 61 uJ on
    Haswell and Skylake. They update about every millisecond, with jitter (Khan et al. 2018).
  - The kernel's AMD hwmon documentation for 5.11 estimates that a socket counter wraps in about 9 minutes at 240 W.
  - The sysfs counter wraps at `max_energy_range_uj`, so the sampler must read more often than the shortest wrap time
    (range divided by peak package power). The perf events do not wrap (section 2.1).
- **Permissions after Platypus (CVE-2020-8694, CVE-2020-8695):**
  - Unprivileged RAPL readings were used as a power side channel against Intel CPUs (Platypus site; Intel
    INTEL-SA-00389, both fetched).
  - Linux fixed CVE-2020-8694 by making `energy_uj` readable by its owner (root) only (kernel commit 949dd01,
    "powercap: restrict energy meter to root access", fetched).
  - The perf route needs `perf_event_paranoid` set to 0 or lower, or CAP_PERFMON, because the events are system-wide
    (kernel perf-security documentation, fetched).
  - The Platypus authors say AMD Zen exposes the same kind of interface. AMD's hwmon driver documentation restricts
    its energy attributes to owner and group, citing CVE-2020-12912.
- **Effect on our scripts:**
  - The sampler runs as root (sudo), as a separate process that only reads counters and writes a timestamped CSV. The
    workload itself never runs as root.
  - We do not make `energy_uj` world-readable and do not lower `perf_event_paranoid` system-wide. Either would reopen
    the side channel on the machine.
  - The manifest records which route was used.
- **Virtual machines and cloud:**
  - WSL2 (Hyper-V) has no powercap tree at all (Ubuntu WSL bug 2022963, a user report with no Microsoft reply,
    fetched). The CNCF Kepler post says VMs generally do not expose power metrics, so tools fall back on trained
    models (fetched).
  - Where a hypervisor does pass the counters through, they measure the whole package, shared with other guests and
    the host. Khan et al. read package and DRAM counters on 2016-17 Amazon EC2 instances, but noted that vCPUs need not
    map to fixed physical cores and that other tenants affect the readings.
  - **Rule:** energy is measured only on bare-metal Linux. VM and cloud-VM results report energy as "not measured
    (virtualised; counters absent or not attributable)". Bare-metal cloud instances will be checked one by one; not
    verified yet.

### 2.4 Later upgrade: an external wall-power meter
- **What it adds:**
  - A meter from SPEC's accepted list, read through PTDaemon, would measure AC input for the whole box, including PSU
    losses, fans, storage and NICs.
  - It would also allow a check of RAPL against the wall, as Hackenberg et al. and Khan et al. did.
- PTDaemon licence terms for our use have not been checked.

## 3. IBM Z: what can and cannot be metered

### 3.1 Findings (neutral)
- **No RAPL-style counter for Linux on Z.** The Linux RAPL drivers above are x86 drivers: the powercap driver matches
  only Intel, AMD and Hygon CPUs (`intel_rapl_msr.c`).
- **z15 and earlier:** we found no IBM source for per-partition power. CPC-level figures existed on the HMC; the only
  source for that is a non-IBM 2026 talk (Velocity Software), so it is weak.
- **z16 and LinuxONE 4: partition power on the HMC, for the machine's owner.**
  - A SHARE 2023 session says the z16 single-frame model introduced power reporting by partition, using
    "underlying hardware monitoring and algorithms" that the abstract does not describe (fetched).
  - IBM's LinuxONE sustainability page says the HMC Environmental Dashboard shows power for a system and a
    partition, starting with LinuxONE 4, and that the HMC Web Services API exposes power data (fetched).
  - The open-source HMC Prometheus exporter added a per-partition power metric for z16 in April 2024 (fetched).
  - All of this is read from the HMC, i.e. by the CPC owner or operator, not from inside the partition.
- **z17 and LinuxONE Emperor 5 (machine type 9175): partition power visible to the operating system.**
  - IBM's z17 Technical Introduction (SG24-8580-01) says z17 reports power at LPAR level, with support in z/OS, z/VM
    and Linux on IBM Z (fetched).
  - **Linux:** `zpwr` (s390-tools) shows the partition's CPU, storage and I/O power. It also shows CPC total,
    unassigned and infrastructure power, but only if the LPAR has "Global Performance Data" authority (IBM docs,
    fetched).
    - It reads the data through DIAGNOSE 0x324. The kernel code is dated 2024 and needs a facility bit (`diag324.c`,
      fetched).
    - Readings are refreshed on an interval (10 s in IBM's example) and may be cached.
  - **z/OS:** SMF type 70 subtype 1 records gain CPC- and LPAR-level power fields (APAR OA66018, z17, closed
    2025-05-09, fetched).
  - **z/VM:** z/VM 7.4 adds monitor record domain 0 record 25. Without Global Performance Data it holds only the
    current LPAR's data (APAR VM66823; record MRSYTPOW; fetched). The Performance Toolkit APAR VM66826 says the data
    exists only on z17 9175 (fetched).
- **How the partition figures are derived:**
  - IBM's z/VM dashboard deck (April 2025, fetched) calls the LPAR's figure an apportionment, based on the LPAR's
    configuration and resource use. It says the split to guests is an approximation, and that part of a Z machine's
    power is fixed whatever the workload.
  - The z17 Technical Introduction says z/OS service-class figures from post-processing are approximations.
  - The documents we found do not describe the firmware algorithm, nor say that the partition values are measured
    at any boundary. We therefore treat them as **vendor-apportioned estimates, not measurements**. This is unverified
    beyond those statements.
- **Not verified:**
  - Whether DIAGNOSE 0x324 is offered to Linux guests under z/VM, or only to Linux running directly in an LPAR.
  - Whether any LinuxONE or z/OS system we may use is a z17.

### 3.2 What our FDR says for Z results
- **Default, on every Z platform:**
  - "Energy: not measurable on this platform."
  - With the reason, e.g. "shared CPC; no energy counter for this partition is visible to us. Machine power is
    reported for the whole CPC to its owner (HMC)", or "z/VM guest; partition power not available to the guest".
  - No per-watt normalisation is computed.
- **Proposed (REVIEW): on z17, where the partition can read IBM's figures** (`zpwr`, SMF 70.1 or z/VM monitor):
  - The FDR may record them in a separate field, "IBM firmware-apportioned partition power", with CPU, storage and
    I/O watts averaged over the measured window.
  - The field is labelled "vendor estimate; not measured; not comparable with RAPL or wall power".
  - It is never used in a cross-platform per-watt comparison.

## 4. How the FDR energy section is filled
One block per result, added to FDR section 6 (or "n/a" with the reason):
- **Status:** `measured (RAPL)`, `measured (wall meter)`, `vendor-apportioned (IBM Z firmware)` or `not measurable
  on this platform`, with the label from section 1 and, if not measured, the reason.
- **Source and domains:** powercap or perf; the zones read (`package-<n>`, `dram`, `psys`, `core`); CPU vendor and
  model; whether RAPL is modelled or measured on that generation (section 2.3).
- **Access route:** root sampler or CAP_PERFMON; kernel version.
- **Sampling interval:** seconds (default 1 s); the number of samples; missing samples, and counter wraps corrected.
- **Window:** start and end timestamps, matching the throughput window.
- **Energy and power:** joules and average watts per domain over the window, mean and 95% CI over the repeats.
- **Energy per unit of work:** joules per committed transaction (online) and joules per job (batch).
- **Idle baseline:** watts per domain; its start delay after the run and its duration (minutes), as set before the
  first measured run; plus the pre-run idle reading.
- **Power settings:** governor, turbo and C-states (cross-reference to the manifest).
- **Exclusions:** what the boundary leaves out (for RAPL: PSU, fans, storage, NICs, board).
- **Raw data:** the sampler CSV and its checksum, under FDR section 7.

## 5. Sources
SPEC
- SPEC accepted power analyzers: https://spec.org/power/docs/specpower-device_list/accepted_power_analyzers (fetched)

Linux and RAPL
- Kernel powercap documentation: https://www.kernel.org/doc/html/latest/power/powercap/powercap.html (fetched)
- Kernel perf RAPL driver: https://raw.githubusercontent.com/torvalds/linux/master/arch/x86/events/rapl.c (fetched)
- Kernel powercap MSR driver (AMD/Hygon support): https://raw.githubusercontent.com/torvalds/linux/master/drivers/powercap/intel_rapl_msr.c (fetched)
- Kernel commit 949dd01, "powercap: restrict energy meter to root access": https://github.com/torvalds/linux/commit/949dd0104c496fa7c14991a23c03c62e44637e71 (fetched)
- Kernel perf-security documentation: https://docs.kernel.org/admin-guide/perf-security.html (fetched)
- Kernel amd_energy hwmon documentation (5.11): https://kernel.org/doc/html/v5.11/hwmon/amd_energy.html (fetched)
- Platypus: https://platypusattack.com/ (fetched)
- Intel INTEL-SA-00389: https://www.intel.com/content/www/us/en/security-center/advisory/intel-sa-00389.html (fetched)
- Khan et al., "RAPL in Action", ACM TOMPECS 3(2), 2018, doi:10.1145/3177754; open copy:
  https://helda.helsinki.fi/server/api/core/bitstreams/bdc6c9a5-74d4-494b-ae83-860625a665ce/content (fetched)
- Hackenberg et al., "An Energy Efficiency Feature Survey of the Intel Haswell Processor", IPDPSW 2015:
  https://tu-dresden.de/zih/forschung/ressourcen/dateien/projekte/firestarter/2015_hackenberg_hppac.pdf?lang=en (fetched)
- Schöne et al., "Energy Efficiency Aspects of the AMD Zen 2 Architecture", IEEE Cluster 2021:
  https://ar5iv.labs.arxiv.org/html/2108.00808 (fetched)
- Ubuntu WSL bug 2022963, "powercap or intel-rapl not supported": https://bugs.launchpad.net/bugs/2022963 (fetched)
- CNCF, "Exploring Kepler's potentials" (2023): https://www.cncf.io/blog/2023/10/11/exploring-keplers-potentials-unveiling-cloud-application-power-consumption/ (fetched)

IBM Z
- IBM z17 Technical Introduction, SG24-8580-01, section 5.6: https://www.redbooks.ibm.com/redbooks/pdfs/sg248580.pdf (fetched)
- zpwr, Linux on IBM Z device drivers book: https://www.ibm.com/docs/en/linuxonibm/lddd/lddd_r_zpwr_cmd.html (fetched)
- zpwr man page: https://dyn.manpages.debian.org/testing/s390-tools/zpwr.1 (fetched)
- Kernel `diag324.c`: https://raw.githubusercontent.com/torvalds/linux/master/arch/s390/kernel/diag/diag324.c (fetched)
- APAR OA66018 (z/OS SMF 70.1 power, z17): https://www.ibm.com/support/pages/apar/OA66018 (fetched)
- APAR VM66823 (z/VM monitor power): https://www.ibm.com/support/pages/apar/VM66823 (fetched)
- APAR VM66826 (Performance Toolkit; z17 only): https://www.ibm.com/support/pages/apar/VM66826 (fetched)
- z/VM MRSYTPOW record: https://www.ibm.com/support/pages/zvm/pubs/mon750/mrsytpow.html (fetched)
- IBM, "Dashboard for z/VM Energy Consumption" (April 2025): https://www.ibm.com/support/pages/zvm/related/perfkit/datapump/pdppower.pdf (fetched)
- SHARE 2023 session 31249: https://share.org/Events/Past-Events/Proceedings/Proceeding-Details/31249-partition-power-monitoring-with-the-new-hmc-environmental-dashboard-and-wsapi (fetched; abstract only)
- IBM LinuxONE sustainability content solution: https://www.ibm.com/support/z-content-solutions/sustainability-linuxone/ (fetched)
- zhmc-prometheus-exporter 1.6.0 change log: https://zhmc-prometheus-exporter.readthedocs.io/en/1.6.0/changes.html (fetched)
- Velocity Software, "z17 and zVM7.4 Power Metrics" (2026, non-IBM): http://linuxvm.org/pdfs/ukpwmt26.pdf (fetched over HTTP)
