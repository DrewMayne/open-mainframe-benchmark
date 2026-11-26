# CPU accounting: one basis on both sides (draft for comment)

Status: **notes**. This page defines how CPU time per transaction is counted, so that the figure means the same
thing on x86 Linux, Linux on Z and z/OS. It supports proposed SPEC 3 ("Cost per transaction") and 8.4, and FDR
change F9 ([ARCHITECTURE.md](ARCHITECTURE.md), section 10). No kit code is published and no kit run has been made.
Items marked **(proposed)** are not yet decided. Anything without a source is marked "(source needed)", and
anything taken from experience or an older release, still to be checked against the release in use, is marked
"(verify)".

## 1. The rule
**The reported CPU figure is the busy processor time of the whole system under test during the measured window, by
engine type, divided by the committed requests in that window (proposed SPEC 7.1).** CPU per completed request,
business rejections included, is reported beside it.
- "Whole system under test" means every CPU, engine, node and partition the system under test may use: the
  application, the database, any counter store, cache or load balancer, and any coupling facility partition.
  Driver hosts are excluded, and their CPU is reported separately.
- "Busy" includes time that is not charged to any process, job or transaction: interrupt handling, kernel threads,
  and on z/OS uncaptured time. Per-process, per-cgroup, per-job and per-transaction figures are a **breakdown
  only**. They are never the reported figure, because each leaves some work out, and leaves out different work on
  each platform.
- "By engine type" means CPs and zIIPs are reported separately on z/OS, and an x86 core-second is never added to a
  Z engine-second.
- The same window, the same transaction count (committed requests, proposed SPEC 7.1) and the same rule apply on
  both sides.
- **On IBM Z, one basis for every partition:** the PR/SM view, for z/OS and for Linux in an LPAR alike (section 3).
  On x86, the busy time of the cpuset plus the confinement residue (section 2). The basis is validated against a
  known load on each platform before it is used (section 8).
- **SMT.** With SMT on, Linux busy time is in hardware-thread seconds, and is reported as such. A cross-platform
  CPU-per-transaction comparison pairs only runs with the same SMT state on both sides, or is labelled "unequal
  SMT". On z/OS with multithreaded zIIPs, the multithreading figures of [S25] (capacity factor, MT PROD) are applied.
  The scoring configurations run without SMT on both sides (docs/HYPOTHESES-ADDENDUM.md).

## 2. Linux (x86 and Linux on Z)
- **Reported figure on x86:** for each CPU in the system under test's cpuset (proposed SPEC 8.2), on every node, the
  busy time over the window from `/proc/stat`: user, nice, system, irq and softirq [S38]; plus the **confinement
  residue** of the other online CPUs (their busy time less the background load's cgroup and less the baseline,
  proposed SPEC 8.2), so that work outside the cpuset is still counted, as all work in a Z partition is. Idle and
  iowait are not busy.
- **Linux on Z in an LPAR:** the reported figure is the PR/SM view of section 3. The `/proc/stat` busy time of the
  counted IFLs is reported beside it.
- **Steal** (time the hypervisor gave to other work [S38]) is reported beside it, never added to busy time. On
  shared capacity it is part of the other-load disclosure (FDR 2).
- **Guests.** Where the system under test runs as a guest (z/VM or KVM on IBM Z; KVM or another hypervisor on x86),
  the reported figure is the busy time of the processors the guest runs on, from the hypervisor's view: the z/VM
  monitor data, or the KVM host's `/proc/stat`. The guest's own figure is a breakdown. The layer is named in every
  result. The scoring configurations run Linux directly in an LPAR on IBM Z and on bare metal on x86, so that
  neither side has a hypervisor layer the other lacks (docs/HYPOTHESES-ADDENDUM.md).
- **Guest columns:** if a node is itself a hypervisor host for part of the system under test, its guest time is
  stated so that it is not counted twice.
- **The confinement check** uses the same counters on the CPUs outside the cpuset: their busy time during the
  window, less the CPU time of the background load's cgroup in the loaded neighbour state, and less a baseline
  measured with the system under test stopped and the background load running, is the residue. It must stay within
  the bound of proposed SPEC 8.2 (proposed: 1% of the counted cores' busy time), and it is added to the reported
  figure. Interrupts are handled on the counted cores (IRQ affinity [S39], queue mapping and packet steering, as in
  proposed SPEC 8.2), so their time lands in the irq and softirq columns of the counted CPUs.
- **Kernel accounting, and how s390 and x86 differ.** The precision of these columns depends on how the kernel
  accounts CPU time, and the defaults differ by architecture [S46]:
  - on s390, the kernel always selects virtual CPU time accounting (`select VIRT_CPU_ACCOUNTING`), the
    "deterministic task and CPU time accounting" of `CONFIG_VIRT_CPU_ACCOUNTING_NATIVE`, which charges time exactly
    at kernel entry and exit; it also accounts stolen time on logically partitioned machines [S46];
  - elsewhere, including x86, the default is tick-based sampling (`CONFIG_TICK_CPU_ACCOUNTING`, which "depends on
    !S390"), unless the kernel is built with `NO_HZ_FULL`, when `CONFIG_VIRT_CPU_ACCOUNTING_GEN` is the default
    [S46]. Whether each distribution's x86 kernel also accounts interrupt time separately
    (`CONFIG_IRQ_TIME_ACCOUNTING`) is checked against its kernel configuration (verify);
  - on s390 with SMT on, how the kernel scales thread time by multithreading utilisation is to be checked against
    the kernel in use (verify). The scoring configurations run without SMT.

  The kernel version and its CPU-time accounting options (`CONFIG_TICK_CPU_ACCOUNTING`,
  `CONFIG_VIRT_CPU_ACCOUNTING_GEN`, `CONFIG_VIRT_CPU_ACCOUNTING_NATIVE`, `CONFIG_IRQ_TIME_ACCOUNTING`) are recorded
  for every node. A comparison where the two sides use different accounting methods is labelled "unequal kernel
  accounting", and the FDR states the direction of the expected bias where it is known.
- **Breakdown:** cgroup `cpu.stat` and `getrusage` per process; network softirq and kernel-thread work charged to
  whichever task happened to run, or to `ksoftirqd`, is not in them, which is why they are not the reported figure.

## 3. IBM Z partitions (z/OS, and Linux in an LPAR)
- **Reported figure, one basis for every Z partition:** the partition's **effective dispatch time** by engine type
  (CP, zIIP, IFL) over the window, in the PR/SM view: on z/OS, the sum of SMF70EDT over the partition's logical
  processors of each type, in the PR/SM partition data of the RMF CPU Activity data (SMF type 70 subtype 1) (verify
  the field names, SMF70EDT and SMF70PDT, against the release in use). This is the basis of the capture-ratio method
  [S25], which uses "the actual amount of CPU time the partition was dispatched to physical processors". It includes
  time that no address space or enclave is charged for.
  - For Linux in an LPAR, the same partition data: read through `hyptop` with global performance data authority, or
    from the RMF partition data of a z/OS partition on the same CPC (verify both routes against the releases in
    use). Where the PR/SM view cannot be read for a partition, the operating system's view is reported, labelled
    "OS view only", and no cross-platform CPU-per-transaction figure is drawn from it.
  - The partition's **LPAR management time** (total dispatch time, SMF70PDT, minus effective dispatch time) is
    reported beside it, listed and not counted (ARCHITECTURE.md, section 1.2).
  - The operating system's own view (the MVS view of busy time; `/proc/stat` busy on Linux) is reported beside it.
- **Zero-load check of the field.** Before the first counted run on a machine, with the system under test started
  and offered no load, the partition's effective dispatch time must be below 5% (proposed) of its online time in
  the engine mode used. If it is not (for example, dedicated processors that report online time as dispatch time),
  the operating system's view is the reported figure for every Z partition of that comparison, and the FDR says so.
- **Interval alignment.** SMF type 70 and 72 records are written once per RMF Monitor I interval, so the interval is
  set to fit the measured window: the RMF interval and its synchronisation (ERBRMFxx `INTERVAL` and `SYNC`) and the
  SMF interval (SMFPRMxx `INTVAL` and `SYNCVAL`) are set so that the window is a whole number of RMF intervals and
  starts on an interval boundary (proposed: a 1-minute RMF interval, synchronised to SMF, with the window start
  aligned to it; verify the parameter names against the release in use). Only records whose intervals lie wholly
  inside the window are used, and their start and end times are recorded.
- **The partition runs only the system under test.** This is the z/OS counterpart of the x86 confinement. During a
  counted run the partition runs nothing but the system under test and the system's own address spaces, as listed
  in the run plan: no other user's jobs, TSO users or started tasks. The SMF type 30 and type 72 data for the window
  are checked against that list, and a run with other work is invalid. The partition's busy time by engine type,
  with the system under test started but offered no load, is measured once per series and reported beside the
  result.
- **Capture ratio.** Work that the system does on behalf of other work, but cannot attribute to any workload, job
  or transaction, is "uncaptured"; the capture ratio is the CPU time attributed to all measured work divided by the
  partition's CPU time [S25]. It is computed from the RMF Workload Activity data (SMF type 72) and CPU Activity
  data (SMF type 70) by the method of [S25]: (CP APPL% + zIIP APPL%) / ((number of CPs × CP%) + (number of zIIPs ×
  zIIP% × MT PROD%)), where MT PROD is 100% without multithreading. It is reported for every run, in total and for
  CPs alone, as [S25] recommends when zIIPs run with multithreading.
- **zIIP normalisation.** On a sub-capacity model, zIIPs run at full speed while the CPs do not, and RMF adjusts
  zIIP service time to an equivalent CP time by a normalisation factor (zNF), kept in the SMF type 72 record [S25].
  The reported figure is raw engine time by type, not normalised time, and the zNF is stated. Any zIIP time taken
  from SMF type 30 is put on the same basis before it is compared (source needed for the SMF type 30 field that holds
  the factor). Scored runs that compare CPs with zIIPs use full-capacity CPs (docs/HYPOTHESES-ADDENDUM.md, H6).
- **zIIP-eligible work on CPs.** With IEAOPTxx `IIPHONORPRIORITY=YES` (the default), standard CPs may run
  zIIP-eligible work when the zIIPs cannot keep up [S29]. The zIIP-eligible time that ran on CPs (SMF type 30) is
  reported, and the setting is recorded.
- **Work done for the transaction in other address spaces:** the Db2 address spaces (including the lock manager),
  TCP/IP, the security manager and others run on the transaction's behalf. They are inside the partition's busy
  time, so the reported figure counts them; the per-address-space split (SMF type 30) is reported as a breakdown.
- **SAP utilisation.** For every z/OS run, the SAP utilisation from the RMF I/O queuing activity data (verify the
  record, SMF type 78 subtype 3) is reported beside the result, because channel-program processing runs on SAPs,
  which are listed and not counted, while on x86 the interrupt and softirq time of I/O lands on the counted cores
  (proposed SPEC 8.5).
- **CICS per-transaction figures are a breakdown only.** The task CPU time in CICS performance-class monitoring
  (SMF type 110) covers the transaction's own task, not the region's own work (dispatching, system tasks and other
  region overhead) (source needed for the exact field definitions). It is reported per transaction type, with the
  region's address-space time (SMF type 30) beside it.
- **Multithreading (SMT) on zIIPs** changes how busy time relates to capacity; the MT mode and the RMF
  multithreading figures (capacity factor, MT PROD) are recorded [S25].
- **Shared partitions:** the partition's physical processor consumption and the other partitions' consumption of
  the pool, from the RMF partition data, are reported (FDR 2, required fields).

## 4. Several nodes or partitions
The busy time is summed over every node or partition of the system under test, by engine type, and divided by the
transactions completed. The per-tier split (application, database, counter store, load balancer, coupling facility)
is reported beside the total.

**Coupling facility partitions.** For a coupling facility partition, the busy time counted is the coupling facility
processor busy time from the RMF Coupling Facility Activity data (SMF type 74 subtype 4), not the partition's
dispatch time from SMF type 70: a coupling facility on dedicated processors polls for work and shows as fully
dispatched whatever its load (verify the record fields against the release in use). Its SMF type 70 dispatch time is
reported beside the counted figure, and its dynamic dispatching setting (DYNDISP) is recorded.

## 5. Monitoring is work
On z/OS, CICS performance-class monitoring and Db2 accounting traces add CPU to every transaction; on Linux the
samplers add some too. The options in force are recorded on both sides.
- **Scored and counted runs** run with per-transaction monitoring off on both sides (CICS performance-class
  monitoring and Db2 accounting classes beyond the minimum needed for SMF type 30 and type 72 data; on Linux, the
  samplers at the rate fixed in the run plan). Interval data (RMF, SMF types 30, 70, 72 and 74; `/proc/stat`) stays
  on.
- **The per-transaction breakdown** comes from one point per series repeated with per-transaction monitoring on. The
  difference in CPU per transaction between the two is reported as the cost of monitoring
  ([PLAYBOOK.md](PLAYBOOK.md), section 12). The monitored point scores nothing.
- **The audit stream of the invariant check** (proposed SPEC 1.8.5) is written in every measured run, on both
  sides, so its cost is part of the measured work. One point per series is also run with it off, and the difference
  is reported.

## 6. What this basis does not capture
The reported figure counts the cores and engines the system under test runs on. It does not count processors that
do work outside them: on IBM Z, System Assist Processors, Integrated Firmware Processors, the z17 Data Processing
Unit, and the processors in channels, adapters and the storage subsystem; on x86, the processors in network adapters,
storage controllers and on-die accelerators such as QAT, IAA and DSA. These are listed with every result, on both
sides (proposed SPEC 8.4), and are not converted into core time. Any component that runs a step of the request
pipeline is not in this list: it is part of the system under test and is counted (proposed SPEC 8.4).

## 7. Workload characterisation
Busy time says how much CPU a transaction took, not what kind of work it was. Every counted series therefore also
reports, where the platform allows (proposed SPEC 3):
- on IBM Z, the CPU-measurement facility counters (SMF type 113 on z/OS; the `cpum_cf` counter sets on Linux in an
  LPAR authorised for them), with the L1 miss rate and the relative nest intensity, and the LSPR workload category
  they imply (verify the counter set names against the release in use);
- on x86, instructions retired, cycles and last-level cache misses per transaction, from `perf`;
- on both, a breakdown of CPU per transaction into transport, data adapter, business logic, data manager and
  operating system.

Where counters cannot be read (for example in a z/VM guest), the FDR says so and the result is labelled "not
characterised".

## 8. Validating the basis
Four accounting systems are compared here (Linux `/proc/stat` on two architectures, the PR/SM partition data, and
the MVS view), so the basis is checked against a known load before it is used (proposed SPEC 4.17). Before the first
counted run on each machine, and again after any change of kernel, z/OS level or CPC microcode, the kit's
calibration load is run on every engine type of the system under test at 25%, 50% and 80% (proposed) of one engine.
It is a single-threaded CPU-bound loop whose CPU time per operation is first measured alone on one otherwise idle
engine. The CPU per operation given by the reported basis must match that time within 3% (proposed) at each rate,
and the capture ratio (z/OS) or the confinement residue (Linux) is reported with it. A machine that fails gives no
counted result until the cause is found and recorded in the tuning log.

## Sources
Fetched 2026-11-26. The numbering matches [ARCHITECTURE.md](ARCHITECTURE.md) section 12.
- [S25] J. Burg and B. Snyder, IBM, *z/OS: Calculating the Capture Ratio* (white paper, 2017-06-29): the definition
  of uncaptured time and the capture ratio; the formulas with zIIPs, sub-capacity models (zNF) and SMT (MT PROD); the
  recommendation to compute a CP-only ratio too when zIIPs run with SMT:
  https://www.ibm.com/support/pages/system/files/inline-files/zOS_-_Calculating_the_Capture_Ratio_v1.pdf
- [S29] IBM z/OS 2.2, Reviewing z/OS parameter settings (IIPHONORPRIORITY) (an older release: verify against the
  current one): https://www.ibm.com/support/knowledgecenter/en/SSLTBW_2.2.0/com.ibm.zos.v2r2.ieaw100/ieaw100160.htm
- [S38] Linux man-pages, proc_stat(5): https://man7.org/linux/man-pages/man5/proc_stat.5.html
- [S39] Linux kernel, SMP IRQ affinity: https://docs.kernel.org/core-api/irq/irq-affinity.html
- [S46] Linux kernel, `init/Kconfig` ("Cputime accounting": `TICK_CPU_ACCOUNTING` "depends on !S390 &&
  !NO_HZ_FULL"; `VIRT_CPU_ACCOUNTING_NATIVE`, "Deterministic task and CPU time accounting") and `arch/s390/Kconfig`
  (`select VIRT_CPU_ACCOUNTING`): https://github.com/torvalds/linux/blob/master/init/Kconfig,
  https://github.com/torvalds/linux/blob/master/arch/s390/Kconfig
