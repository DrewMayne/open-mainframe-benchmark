# Playbook: running the benchmark end to end (draft for comment)

Status: **notes**. This is the procedure a third party follows to produce a result. The kit is not published yet,
so the steps that need it are marked **(kit not yet published)**. Everything else (permissions, hardware,
configuration records, the run plan) can be prepared now. Anything without a source is marked "(source needed)", and
anything still to be checked against the release in use is marked "(verify)".

The playbook follows [SPEC.md](../SPEC.md) and the proposed additions to it: the request profile and data variants
(proposed SPEC 1.8), the run rules 4.9-4.18, the load model (proposed SPEC 7), the equal-service comparison
(proposed SPEC 8), the scale-out tier (proposed SPEC 9) and the preconditions before any headline result (proposed
SPEC 10). Where they differ, SPEC.md wins. The deployments, and what is counted as a core, are described in
[ARCHITECTURE.md](ARCHITECTURE.md); how CPU time is counted, in [CPU-ACCOUNTING.md](CPU-ACCOUNTING.md); the
registrations, the run registry and the tuning logs, in [REGISTRY.md](REGISTRY.md). Every result ends in an FDR
([FDR-TEMPLATE.md](FDR-TEMPLATE.md)). The default value of every method parameter is in proposed SPEC 7.5; a value
marked "(proposed)" below is that default.

**Which tier a result belongs to.** Every result names one of three tiers (ARCHITECTURE.md, section 1):
- **equal core counts on a single box,** for the registered per-core hypotheses such as H1, in their named scoring
  configurations;
- **equal service** (exploratory, with no registered prediction, and never a headline result): for a stated
  throughput within a p99 bound fixed before the runs, what each side needs (section 7);
- **the scale-out tier** (section 10), single-writer, also exploratory.

**What is reported first.** An FDR that scores a registered hypothesis reports its verdicts first, before any other
figure, whatever they are (proposed SPEC 4.12). Until the preconditions of proposed SPEC 10 hold, every FDR summary
also says "Preliminary" and lists the preconditions not met.

## Contents
1. Before you start: permission, licences, registration and the run plan
2. Prerequisites
3. Build
4. Configuration baseline
5. Load driver: sizing and validation
6. Calibration: closed-loop capacity and the open-loop rate sweep
7. Equal-service sizing (exploratory)
8. Run rules
9. The bookkeeping profile and the live-database variant
10. The scale-out tier
11. Batch alongside online, with priority variants
12. Measurement
13. Teardown
14. Disclosure in the FDR

Appendix A: run plan template. Appendix B: checklist.

## 1. Before you start: permission, licences, registration and the run plan
1. **Confirm that the platform's terms allow performance testing and publishing results,** or get the owner's
   written permission (README, "Permitted platforms only"; [platforms.md](platforms.md)). Record who gave it and
   when (FDR 9). Also agree with the owner whether an IPL or re-activation of the partition, and an abrupt stop for
   the durability test, are allowed (sections 8 and 9.2).
2. **Check the software terms too. This is a gate, on both sides.** Database and other licences may restrict
   publishing benchmark results ([prior-art.md](prior-art.md), SB02; a search-result snippet, not verified). For
   every licensed product in the run, check the licence agreement and the product's licence information for
   conditions on benchmarking or publishing results, and record what they require (FDR 9):
   - on z/OS: z/OS, CICS TS, Db2 for z/OS, Enterprise COBOL, ICSF and the Java runtime;
   - on Linux, either architecture: Db2 for Linux, UNIX, and Windows, if used; the distribution's subscription
     terms; any hypervisor; any vendor accelerator library (for example a QAT driver stack); a commercial JVM, if
     used.

   The licence agreements of the programs in the run, IBM's and others' (for example a commercial JVM, a Linux
   distribution's subscription terms or a hypervisor's), are to be checked for benchmarking clauses. Such a clause
   may allow disclosure of results only under conditions, for example disclosing the methodology, running at
   current fix levels, or following the vendor's tuning guidance. Where a condition cannot be met by a configuration
   (for example, an untuned configuration where tuning guidance must be followed), that configuration is not
   published, and the FDR says so, naming the term and the configuration withheld. Where a condition requires
   current fixes, a reproduction may run later fix levels than the original; the FDR lists every version that
   differs, and the reproduction criterion of SPEC 10 applies to the result as published. **No result from a
   licensed program is published until this check is done and recorded.** These terms have not been checked yet.
   PostgreSQL's licence permits use "for any purpose" and places no condition on publishing results [S36].
3. **On a system without that permission, do functional checks only:** compiles, installs and small pass/fail
   checks, with no timing recorded ([PORTING-zOS.md](PORTING-zOS.md), "Functional verification versus
   measurement"). Sections 6-11 of this playbook are never run there.
4. **Check pre-registration.** If the result will score a hypothesis, the hypothesis must already be registered:
   committed in HYPOTHESES.md, with its scoring configuration in an entry of
   [HYPOTHESES-ADDENDUM.md](HYPOTHESES-ADDENDUM.md) whose signed tag and archive deposit have an archive creation
   time before the first run of the series ([REGISTRY.md](REGISTRY.md); SPEC 4.7), after the pilot and power
   analysis of SPEC 10 (precondition 6). Every machine of the series must already be named in the machine registry,
   before any kit run on it, calibration included. The run must be in that hypothesis's scoring configuration. Note which hypotheses the run plan scores. Equal-service series and scale-out results score no
   registered hypothesis. Any run outside a named scoring configuration (another profile, data variant, track, core
   count, scale factor, machine class or tier) is reported as an unregistered result, and never scores or re-scores
   a registered hypothesis.
5. **Write the run plan (Appendix A), commit it and tag it before the first run of the series, calibration
   included.** It fixes the tier, the calibration size, the calibration bound L_cal, the timeout T, the rules that
   derive the service targets and the latency bounds, the declared configurations, the failure threshold, the repeat
   count and seeds, the rate ladder, the node counts and the priority variants. After calibration, the derived
   targets and bounds are added to the plan, committed and tagged before the first counted run (proposed SPEC 8.1).
   Any other value chosen after looking at data starts a new, labelled series; the results under the committed plan
   are still published in full beside it (proposed SPEC 4.13).
6. **Open the series in the run registry,** and start its tuning log ([REGISTRY.md](REGISTRY.md)). From then on,
   every run is entered in the registry before it starts, and every configuration change is entered in the tuning
   log before the first run it affects (proposed SPEC 4.13 and 4.14).
7. **Plan the machine time.** A run takes at least warm-up plus window plus data reset plus the idle period. At the
   proposed defaults (a warm-up of 10 to 30 minutes and a 10-minute window) a run is at least 20 minutes before data
   reset, so one point at 6 repeats is at least 120 minutes, plus the three or more boots or IPLs between its
   repeats, and a fine pass of k steps is at least k times that. The run plan states the planned total for each series, and the total is agreed
   with the system's owner before the first run (CONTRIBUTING.md, "Agreed load").

## 2. Prerequisites

### 2.1 System under test
- **Equal core counts:** a Z partition and an x86 server with the same number of cores or IFLs (SPEC 2), in the
  scoring configurations of the addendum. Cross-platform ratios need a current general-purpose x86 server part on
  bare metal (proposed SPEC 2, "A fair x86 baseline").
- **Equal service:** a Z partition and an x86 server that can each be run at the declared set of sizes (engines or
  cores), including the common calibration size and at least twice it (proposed SPEC 8.1 and 8.2).
- **Scale-out tier:** an x86 cluster of identical application nodes (enough for the largest N in the run plan),
  plus the database, counter and session store and load balancer nodes declared in the plan; and the Z
  configuration it is compared with.
- **z/OS products** for the z/OS legs: CICS TS, VSAM, DFSORT, WLM in goal mode, ICSF, on-chip compression enabled,
  Semeru Java, and Db2 for z/OS for the live-database variant (PORTING-zOS.md, "What the system must provide").
  Before any z/OS run plan that uses compression, establish from IBM's documentation whether a priced z/OS software
  feature must be enabled (in IFAPRDxx) for the compression paths used, and record it as a cost item for the z/OS
  legs. As we understand it, z/OS use of the on-chip compression accelerator through zlib and the system services
  needs the z/OS zEDC software feature, while DFLTCC on Linux on Z needs no priced z/OS feature (verify both).
- **For counted results on IBM Z,** the owner grants: Linux directly in an LPAR, or a z/OS partition; dedicated or
  entitled engines; IPLs or re-activations between repeats, and an abrupt stop for the durability test; and
  authorisation for the CPU-measurement facility counters and for the partition data the CPU basis reads (global
  performance data for Linux in an LPAR). A z/VM or KVM guest suits development and labelled variants only.
- **Database products** for the live-database variant (proposed SPEC 1.8.4), each the newest release generally
  available on every platform of the comparison when the run plan is committed, the same version and fix level on
  both sides of a form A comparison:
  - form A, same product (platform comparison): PostgreSQL on Linux on Z and on x86 Linux; Db2 for Linux, UNIX, and
    Windows on Linux on Z and on x86 Linux, in a release whose IBM system requirements list Linux on IBM Z (the FDR
    cites that page);
  - form B, whole stack (whole-stack comparison): Db2 for z/OS, against Db2 for Linux, UNIX, and Windows or
    PostgreSQL on x86 Linux.
- **Systems programmer help on z/OS:** a WLM service class (or classes) for the benchmark, access to SMF and RMF
  data, the RMF and SMF interval settings of section 4.3, and, for equal service, a way to change the partition's
  engine count between points: reserved logical processors in the image profile, varied with the z/OS
  `CONFIG CPU(n),ONLINE|OFFLINE` command; on Linux on Z, `chcpu -e|-d` (or `-c|-g` for standby CPUs). Check the
  command syntax against the release in use.
- **Root (or CAP_PERFMON) on x86 nodes** for the energy sampler and the `perf` counters, and root for the cpuset,
  IRQ-affinity and packet-steering settings (section 4.2).

### 2.2 Load driver hosts
- Hardware separate from the system under test, never sharing a machine, hypervisor host or CPC with it.
- Enough of them to pass the validation in section 5 at the highest rate in the run plan.
- The same driver hosts, at the same network distance, for both platforms where possible. If not, the idle
  round-trip time from driver to system under test is measured and reported for each side, and the network-distance
  rule of proposed SPEC 8.1 applies.

### 2.3 Network
- A dedicated or quiet network between the drivers and the system under test, and between cluster tiers.
- Record link speeds, the switch model and the idle round-trip time on every link (proposed FDR change F2).
- No artificial delay on either side. If a hop exists on one side only, it is stated (prior-art "Traps to avoid",
  remote x86 plus network delay).

### 2.4 Time
- All hosts take time from one source, and the offset of each is recorded at the start and end of every run. On
  Linux, use NTP or PTP. On z/OS, record the Server Time Protocol (STP) setting in CLOCKxx [S33] and the timing
  network's external time source, if any (source needed for the external time source options), and the offset
  from the common source.

## 3. Build
All steps in this section are **(kit not yet published)**.
1. Clone the kit at a tagged commit and record the git SHA (SPEC 4.5).
2. Build the track being run (Java or COBOL, SPEC 1.0) under the **build policy** of proposed SPEC 1.0: the same
   compiler release, compiler family and major version, optimisation level and options on both sides; code generated
   for the measured machine on both sides (`-march` for the Z machine's architecture level and for the x86 part's
   microarchitecture), with each distribution's default build as a labelled variant; GnuCOBOL's `binary-byteorder`
   set to `native` [S47], after the REDEFINES and record-layout check of proposed SPEC 1.0; the same indexed-file
   handler and version, and the same GMP version, on both sides; the same portable library builds in plain mode,
   and each platform's vendor-optimised builds in accelerated mode (proposed SPEC 2, "A fair x86 baseline"). For
   the Java track on x86 Linux and on Linux on Z, also build the HotSpot variant (proposed SPEC 1.0). Record every
   flag and every library build (FDR 3).
3. Generate the data at both scale factors of the run plan (proposed SPEC 1.5: the cache-resident S and the
   memory-resident S) with the stated seed, with the record counts, record sizes, key-access distribution and
   payload sizes of the request stream (proposed SPEC 1.5). Record the checksums of the generated record files
   (FDR 4).
4. For the bookkeeping profile, generate the principals, sessions, limits and permission data from the same seed
   (proposed SPEC 1.8.3), and the TLS certificate and keys of the run plan (proposed SPEC 1.8.1).
5. Load the data into the store chosen: the file-based store with its label and durability class, or the live
   database in the form the plan names (proposed SPEC 1.8.4). The load step writes binary fields in the platform's
   native byte order.
6. **Run the golden business-output check** (SPEC 1.0, track equivalence; proposed SPEC 1.8.5). A build that fails
   it is not measured. For a live-database run, the check passes on each database used.
7. **Run the acceptance tests** (proposed SPEC 4.15): atomicity, isolation and durability, on each side and in each
   data variant used, and publish their output (FDR 6).
8. Build and self-test the load driver (section 5).

## 4. Configuration baseline
Settle the configuration, record it, then freeze it. Every item below goes into the run manifest (SPEC 4.5) and the
FDR, in machine-readable form where the platform allows. A change after the freeze is entered in the tuning log
before the run it affects (proposed SPEC 4.14), and a change that alters any fixed value of the run plan starts a
new series.

### 4.1 Every platform
- CPU model, machine type, its general-availability date and its class (proposed SPEC 8.2, "SKU"), cores or engines
  used, SMT or MT state, memory, the last-level cache available to the counted cores, storage type and size, network
  adapters.
- Processors and adapters not counted as cores (ARCHITECTURE.md, section 1.2): on Z, SAPs, IFPs, the on-chip DPU
  (z17), channels, network adapters and their offloads, and cryptographic adapters; on x86, network adapters and
  their offloads, storage controllers, and QAT, IAA or DSA where present; on both, the storage subsystem. Any device
  that runs a step of the request pipeline or terminates TLS is counted instead (proposed SPEC 8.4).
- OS and kernel or release; compiler, runtime and library versions and builds; the build policy record (section 3).
- Accelerated-extension steps and their mode, with the capability masks of plain mode (SPEC 1.4).
- The tier (equal core counts, equal service, or scale-out), the request profile (P0 or P1, with the TLS setting),
  the data variant, its label and durability class, and for a live database its form; the scale factor; the
  transport (receiving program and class, proposed SPEC 1.1); the availability design (A0 or A1).
- The licence metric of each software product (per core, per Processor Value Unit, MSU-based or other), for the
  pricing appendix (FDR 8).
- **The guarantees** (proposed SPEC 1.8.3, 1.8.4, 4.11 and 4.15): the durability setting or class; the storage
  class, write-cache protection, replication and log copy count of the log device, with its write-latency baseline;
  the counter store's durability class; the access-record buffering; the isolation level. They must match on both
  sides.
- **Security and audit settings that add work per request,** at the shipped defaults on both sides unless the run
  plan says otherwise, and recorded: on Linux, the CPU vulnerability mitigations as the distribution ships them, the
  audit daemon and the mandatory access control system as shipped; on z/OS, which has no shipped default for them,
  the SMF record types and RACF logging options of the kit's z/OS configuration checklist. Any departure is
  labelled. A mitigations-off series, on both sides together, may be run as a labelled variant.
- **Per-transaction monitoring** off in counted runs on both sides, with interval data on (CPU-ACCOUNTING.md,
  section 5).
- **Every limit that could refuse a request,** each set so that it does not refuse work at the highest planned rate
  and fits the driver's connection pool P (proposed SPEC 1.8.2 and 7.1), with its value recorded:
  - on Linux: the TCP listen backlog (`net.core.somaxconn`, `net.ipv4.tcp_max_syn_backlog` and the listener's own
    backlog), open-file limits, worker-pool and queue limits, and the database's maximum connections (PostgreSQL
    `max_connections`);
  - on z/OS: the TCP/IP `SOMAXCONN` setting, the backlog of the CICS TCPIPSERVICE, the CICS MAXSOCKETS limit, the
    CICS maximum tasks and transaction class limits (including purge thresholds), and the DB2CONN thread limits with
    threads set to wait rather than fail;
  - the exact parameter names are to be checked against the release in use (source needed);
  - a limit raised after a refusal is entered in the tuning log, and the series continues only if no
    decision-point run has yet been made; otherwise it starts a new, labelled series (proposed SPEC 1.8.2).
- **Lock and deadlock settings,** recorded (proposed SPEC 1.8.2): for example the Db2 lock-timeout and
  deadlock-detection settings, PostgreSQL `deadlock_timeout` and `lock_timeout`, and the CICS deadlock timeout
  (source needed for the exact parameter names).

### 4.2 x86 Linux, per node
- Bare metal for counted runs. Firmware and CPU microcode versions, the power profile, memory population per channel
  and speed, the NUMA mode (sub-NUMA clustering or NPS), and the state of the CPU vulnerability mitigations
  (`/sys/devices/system/cpu/vulnerabilities/`).
- Frequency governor, turbo (on, with the counted cores capped at the part's sustained all-core frequency for
  counted series, measured once per machine with every core of the socket running the background load), C-states
  and SMT, fixed for the series; the measured mean frequency of the counted cores at every point (proposed SPEC
  8.2).
- **Confinement** (proposed SPEC 8.2 and 4.4): the cgroup v2 cpuset (`cpuset.cpus`, and `cpuset.mems` set to the NUMA
  node of the counted cores) that holds every process of the system under test [S1]; the counted cores on one
  socket, one NUMA node and one last-level-cache domain where they fit; the IRQ affinity of every device interrupt
  whose affinity can be set (`/proc/irq/*/smp_affinity_list`) [S39]; for kernel-managed interrupts (for example
  NVMe, and many multi-queue network drivers), their queue mapping, the RSS indirection table and RPS/XPS masks
  limited to queues and CPUs inside the cpuset, and where needed `isolcpus=managed_irq,<the other cores>`, with the
  `/proc/interrupts` counts per CPU recorded; cores outside the cpuset are never taken offline; the kernel threads
  moved. Record all of it.
- **Neighbour state** (proposed SPEC 8.2): loaded for counted series, with the background program, its cgroup and
  the measured utilisation of the other cores of the socket (80% ± 10 points, proposed); quiet as a labelled
  variant.
- The cgroup v2 tree, with `cpu.weight` and `cpu.max` for each group, and the nice value and scheduler policy of each
  process [S1][S2]; the kernel version, which determines the fair scheduler (CFS before 6.6, EEVDF from 6.6) [S37];
  and the autogroup setting (`kernel.sched_autogroup_enabled`) [S26].
- The kernel's CPU-time accounting options (CPU-ACCOUNTING.md, section 2) [S46].
- For a cluster: the role of the node, and the load balancer's algorithm and health-check settings.
- Database: product, version and fix level, isolation level and locking clauses, durability settings (for
  PostgreSQL, `fsync`, `synchronous_commit` and `full_page_writes` [S35]; for Db2 for Linux, UNIX, and Windows, the
  logging settings), the storage class of the log device (section 4.1), replication mode and number of synchronous
  standbys [S9], connection pool size per application node, and automatic maintenance (for example autovacuum).
- Counter and session store: product and version, persistence setting and durability class, eviction policy,
  memory size.
- The Linux port's configuration (proposed SPEC 1.1): the worker model, the indexed-file handler's cache, lock and
  sync settings, and the record locking between batch and online processes.

### 4.3 z/OS
- Capacity setting, CP and zIIP counts, the number of logical processors of each type online, LPAR weight, the total
  weight of the pool and the physical processors in it, and from them the entitlement; capping (initial capping,
  absolute capping, defined capacity and group capacity limits); HiperDispatch on or off, the vertical polarisation
  of each logical processor at the start and end of each run, and the drawer and chip of each where shown; the zIIP
  multithreading mode (LOADxx `PROCVIEW`, IEAOPTxx `MT_ZIIP_MODE`) and IEAOPTxx `IIPHONORPRIORITY` (with YES, the
  default, standard CPs may run zIIP-eligible work when the zIIPs cannot keep up [S29]), and the other IEAOPTxx
  settings that decide when CPs help zIIPs (for example `ZIIPAWMT`; verify against the release in use)
  (PORTING-zOS.md, "Measurement on z/OS"). Counted runs follow the capping and entitlement rule of proposed SPEC 8.2.
- **Neighbour state** (proposed SPEC 8.2): for the loaded state, the background partition and its engines, or the
  other partitions whose work provides the load, and the utilisation of the other physical processors of the drawers
  that hold the counted logical processors, for every run, from the RMF partition data and the processor topology
  data; for the quiet variant, evidence that no other partition was dispatched on those drawers.
- **CPU basis** (CPU-ACCOUNTING.md, section 3): the effective dispatch time and LPAR management time of the
  partition, the zero-load check of the field once per machine, the SAP utilisation for every run, and the
  accounting validation of proposed SPEC 4.17.
- **No soft capping** may engage during a series: the rolling four-hour average and the WLM capping percentage are
  recorded at the start and end of each run (verify the SMF type 70 field, for example SMF70LAC, and the report).
- **Partition hygiene** (proposed SPEC 8.2): the System Recovery Boost setting and the IPL time, with every counted
  run starting after any boost period has ended; no temporary capacity record activated or deactivated during the
  series; Intelligent Resource Director weight management and vary CPU management off; the CPC's microcode driver
  and bundle level.
- **The partition runs only the system under test:** the list of address spaces allowed (the system under test
  and the system's own), checked against SMF types 30 and 72 for each window; the busy time with the system under
  test started and no load, once per series (CPU-ACCOUNTING.md, section 3).
- **RMF and SMF intervals:** the RMF Monitor I interval and its synchronisation (ERBRMFxx `INTERVAL` and `SYNC`) and
  the SMF interval (SMFPRMxx `INTVAL` and `SYNCVAL`) set so that each measured window is a whole number of RMF
  intervals and starts on an interval boundary (proposed: a 1-minute RMF interval, synchronised to SMF); only the
  records whose intervals lie wholly inside the window are used, and their start and end times are recorded (verify
  the parameter names against the release in use).
- For shared engines, the other partitions' utilisation of the pool, for every run (FDR 2, required fields).
- **Configuration capture,** in machine-readable form: the system parameter members in force (for example
  IEASYSxx, IEAOPTxx, LOADxx, IFAPRDxx and CLOCKxx), the maintenance level, an export of the WLM service definition,
  the CICS system initialisation overrides and an extract of its resource definitions, and the Db2 subsystem
  parameters.
- The WLM service definition excerpt for the classes used: goals, importance, classification rules, any resource
  group, and region or transaction management for CICS [S6][S7][S40].
- CICS: release, the system initialisation parameters that differ from defaults, the topology (regions, MRO and
  function shipping, any file-owning region), program concurrency (quasi-reentrant, or threadsafe after a code
  review [S45]), Open Transaction Environment use, MXT and TCB limits, the file definitions (recovery, strings,
  buffers) (SPEC 4.3), the system log and forward-recovery log streams (ARCHITECTURE.md, G2), and the monitoring
  options in force, and the system initialisation parameter FCQRONLY (verify its default for the release in use);
  whether data tables or any other in-memory copy of the files are used (proposed SPEC 1.1).
- VSAM or Db2 settings: share options or RLS, logging, buffer pools, isolation and locking clauses; for Db2, the
  product version and fix level, the package bind options, and the accounting and statistics trace classes in force.
- Security manager logging options in force (they add work per request).
- Coupling facility partitions and structures, if any, with their processors and their dynamic dispatching setting
  (ARCHITECTURE.md 4.1).

### 4.4 Linux on Z
- Linux directly in an LPAR for counted runs, with exactly the counted IFLs online (the others deconfigured, or
  offline with `chcpu`); dedicated, or shared under the capping and entitlement rule (proposed SPEC 8.2); for shared
  IFLs, the weight, entitlement, capping and the pool's other load; the CPU dispatching mode (polarisation,
  horizontal or vertical, as `chcpu` sets it [S56]); the SMT state; the CPC's microcode level; steal time; the
  partition's effective dispatch time from the PR/SM view (CPU-ACCOUNTING.md, section 3); and the neighbour state,
  as in 4.3. A z/VM or KVM guest is a labelled variant: then the hypervisor layer, the guest's virtual processor count and
  share setting, and the hypervisor's own processor time for the guest are recorded (CPU-ACCOUNTING.md, section 2).
- The items of 4.2 that apply (cpuset, scheduler, cgroups, kernel accounting, database settings, the Linux port's
  configuration).

### 4.5 Data reset
Writes accumulate during a run. Before every measured run, the data is restored to the generated state, by reloading
or from a snapshot taken after loading. Record the method. The time to restore is not part of any measurement.
After a reset the same cache state is restored on both sides: the database and the transaction manager are
restarted, and on Linux the page cache is dropped (`echo 3 > /proc/sys/vm/drop_caches`). VSAM files are reloaded
into freshly defined clusters, so that no split state carries over. A reload cannot clear a storage controller's
cache on either side (a DASD controller on z/OS, a SAN array on Linux), so each side records its reset method, and
the warm-up is what brings both to a steady state. Statistics are collected after every load at the same point on
both sides (proposed SPEC 1.8.4), and no buffer pool is pre-loaded: the warm-up (section 8) does that. The method is
recorded.

## 5. Load driver: sizing and validation
Requirement (proposed SPEC 7.3): the driver must be shown not to be the bottleneck at every rate it offers.

1. **Size it.** Plan the driver hosts for the highest rate in the run plan, with headroom (proposed: twice that
   rate). The schedule is generated once from the seed and split among the hosts by the stated rule (proposed SPEC
   7.1), so the arrivals are the same whatever the number of hosts. Set the persistent connection pool P by the rule
   of proposed SPEC 7.1 (the smallest power of two at or above 2 × the highest offered rate × L_cal, at most 1,024 in
   all, proposed), split evenly over the hosts, and check that it fits every connection limit of section 4.1 and
   each host's ephemeral ports. All connections, and under P1 every TLS handshake, are made before the warm-up.
2. **Self-test against a null responder** (kit not yet published), on separate hardware, at the self-test rate
   (proposed: twice the highest planned rate), in two modes: answering at once, and answering after a fixed delay d
   = P / (2 × the highest planned rate), at most 0.9 × T, so that every connection is busy (Little's law). Run it, or
   run it again, after the series' smallest L is fixed, and whenever the driver hosts change. Pass when, in both
   modes:
   - the requests completed within T are within 0.1% of the requests scheduled in the window (proposed);
   - the schedule lag (actual send time minus intended send time) at p99.9 is within the run plan's bound
     (proposed: 1% of L, and at most 1 ms);
   - no driver host core or thread is busier than the run plan's limit (proposed: 70%), and network utilisation
     stays below its limit (proposed: 50% of link speed).
3. **Monitor during every run.** Record driver CPU per core or thread, network utilisation and schedule lag. Lag
   counts against the driver only while a connection was writable or a new one could be opened; lag while the system
   under test blocked sends or was not accepting connections is attributed to the system under test and reported
   (proposed SPEC 7.3). A run where a driver limit is broken is invalid, and is reported as a failed run (FDR 6).
4. **Cross-check once per series,** after the series' targets and values of L are fixed, at the highest
   decision-point rate the series offers (for a sweep, one fine step below the knee of the largest configuration
   swept; in equal service and the scale-out tier, the highest target R). Run the full repeat count with the load
   spread over twice as many driver hosts. The driver is shown not to be limiting when Welch's 90% confidence
   intervals for the difference in mean p99, and for the difference in mean achieved rate, lie within ±5% of L and
   ±0.5% of R (proposed margins; two one-sided tests at 5%). If an interval contains 0 but crosses a margin, double
   the cross-check's own repeat count once; if it is still inconclusive, label the series "driver adequacy
   inconclusive". Only an interval that excludes 0 and lies outside a margin labels the series "driver not shown
   adequate": then enlarge the driver and repeat the series; the first series stays in the run registry.
5. **Latency is measured from the intended send time** of each request, never from the actual send time [S8].

## 6. Calibration: closed-loop capacity and the open-loop rate sweep

### 6.1 Closed-loop capacity (SPEC 4.2)
1. Run closed loop with zero think time (SPEC 1.2), doubling the number of connections from 1 (1, 2, 4, 8 ...).
   Stop when two consecutive doublings each raise committed throughput by less than 2% (proposed), or when timeouts,
   refusals and errors exceed the failure threshold of proposed SPEC 7.4. Each step has the full warm-up and window.
   The step with the highest committed throughput among those whose p99 is at most L_cal (proposed) is then run with
   the full repeat count.
2. The mean committed throughput at that step is the **measured capacity** C, with its 95% confidence interval and
   the p99 at that step. Business rejections are reported separately (proposed SPEC 7.1). The
   connection count is recorded; the batch-alongside-online runs use it (section 11). C is the measure the
   registered per-core hypotheses use, and SPEC 4.2 uses it to set the open-loop points of the degradation curve.

### 6.2 Open-loop rate sweep (proposed SPEC 7.2)
1. The run plan already fixes the **latency bound** L (in an equal-service series, the calibration bound L_cal), the
   timeout T (proposed: 10 × L_cal) and the failure threshold (proposed: timeouts, refusals and errors at most 0.1%
   of the requests in the window).
2. **Coarse pass (exploratory).** Offer fixed rates from low to high in even steps (proposed: 10% of C), up to 120%
   of C, so that behaviour past saturation is recorded (SPEC 4.2; H4). One run per step, used only to find the
   bracket between the last sustained and the first unsustained step. Arrivals are Poisson by default, from a
   seeded stream (proposed SPEC 7.1).
3. For each step record: offered rate, achieved rate, p50, p90, p99 and p99.9 (timeouts recorded at T), completed
   requests, business outcomes by type, errors by cause, refusals by component and cause, timeouts, late responses,
   and CPU per tier.
4. **Sustained in a repeat** (proposed SPEC 7.4): a rate is sustained in a repeat when, over that repeat's window,
   with a request's outcome taken when it completes or reaches T, even after the window ends:
   - every request whose intended send time falls in the window has an outcome, and the counts add up;
   - timeouts, refusals and errors together are at most 0.1% of those requests (the failure threshold, proposed);
     and
   - p99, with timeouts and refusals recorded at T, is at or below L.
5. **Fine pass.** Steps of 2% of C (proposed) across the bracket. Each repeat r runs every fine step once, with
   repeat r's arrival seed, and the runs of all steps and repeats are in randomised order (section 8). Only
   fine-pass results decide the knee. A refusal at a fine-pass step makes that step unsustained in that repeat
   (proposed SPEC 7.4); the run stays valid. A limit is never changed within a series once a decision-point run has
   been made: raising one starts a new, labelled series (section 4.1; proposed SPEC 1.8.2 and 4.13).
6. **The knee** (proposed SPEC 7.4): for each repeat r, K_r is the highest fine step R such that R and every lower
   fine step are sustained in repeat r. A step above K_r that passed in repeat r is a **non-monotone pass**: it never
   raises K_r, and it is reported. If the lowest fine step fails in some repeat, extend the fine pass downwards; if
   K_r is the highest fine step in some repeat, extend it upwards, until every K_r lies strictly inside, but no
   further than 50% of C downwards and 120% of C upwards (proposed). The knee is the lower median of the K_r,
   K_(⌈n/2⌉), with its order-statistic interval, that interval's nominal coverage and the between-boot flag.
7. **Past saturation.** The points at or above C, and the coarse steps above the first unsustained step, are
   labelled "past saturation", run with the same window and T on both sides, and reported with their queue growth
   (requests sent minus answered at the end of the window), timeouts, refusals by component and cause (each counted
   as a timeout at T) and achieved rate (proposed SPEC 7.2). These runs stay valid.
8. Do not stop the sweep at the knee, and never let the system refuse requests to protect latency at a decision
   point (proposed SPEC 1.8.2).

### 6.3 What calibration produces
- C (closed loop) with its 95% confidence interval; and the knee (open loop) as the lower median of the per-repeat
  knees, with its interval, its coverage and every K_r.
- The rate points for the degradation curve: 50, 70, 90, 100, 110 and 120% of C (SPEC 4.2).
- For equal service: each side's knee at the common calibration size, found with L_cal, from which the targets and
  bounds are derived (section 7.1).
- The rate ladder for the scale-out tier (section 10).

## 7. Equal-service sizing (exploratory)
Defined in proposed SPEC section 8. It answers: for a stated throughput within a fixed p99 bound, what does each
side need? It is exploratory: it scores no registered hypothesis, and none of its results is a headline result.

### 7.1 Set-up
1. **Fix the calibration in the run plan before any run** (proposed SPEC 8.1): the calibration size (the same
   number of x86 cores and of Z engines, proposed 4, with the z/OS (CP, zIIP) split declared), L_cal (proposed: p99
   of 1 s, overall), T (proposed: 10 × L_cal), and the rules that derive the targets and the bounds.
2. **Calibrate both sides** at the calibration size, with the sweep of section 6.2 and L = L_cal.
3. **Derive the targets:** declared fractions of the **lower** of the two calibration knees (proposed: 25%, 50% and
   80%). Each target is a throughput R at the transaction mix (SPEC 1.2), with the request profile, the data variant
   and form.
4. **Derive the bounds:** offer each side 50% of the lower calibration knee, open loop, at the calibration size, with
   the run plan's repeats, and record its mean p99; record also each side's mean p99 at 5% of that knee (its latency
   floor). Declare at least three values of L by the rule (proposed: 2×, 5× and 10× the higher of the two sides'
   mean p99 at 50% of the lower knee), each at most L_cal; a value the rule would set higher is set to L_cal, and the
   cap is reported. The same R and L apply to both sides. For each L, a side whose latency floor exceeds 0.8 × L
   reports "latency floor above L" instead of a sustaining configuration.
5. **Commit and tag** the derived targets and bounds as an amendment to the run plan before the first counted run.
   The calibrations are reported with the result.
6. **Declare each side's configurations,** in order of size, including the calibration size and at least twice it
   (proposed SPEC 8.2):
   - Z: for Linux on Z, 1, 2, 3 ... IFLs, Linux directly in an LPAR; for z/OS, a declared sequence of (CP, zIIP)
     pairs with at least one CP, for example (1,0), (1,1), (2,1), (2,2) ..., with the order fixed before the series.
     Engines are dedicated, or shared with per-partition absolute capping at the declared count (a whole number),
     exactly that many logical processors online, a weight that alone entitles the partition to the capped capacity,
     and the other partitions' load recorded for every run [S44]. Other shared configurations are labelled and
     disclosed only;
   - x86: the server on bare metal with 1, 2, 3 ... physical cores in a cpuset in one NUMA node that holds every
     process of the system under test, with interrupts and packet steering on the same cores and movable kernel
     threads moved (section 4.2); both hardware threads of a counted core in the set; the SMT state, governor and
     frequency cap fixed for the series; the loaded neighbour state on both sides, with quiet as a labelled variant;
   - the rule for settings that change with size (worker pools, buffer pools, connection limits), stated before the
     series. Every other setting stays the same.
7. Memory and last-level cache per configuration are stated, and memory follows the same rule on both sides, stated
   before the series (the same amount at each count, or the same multiple of the stored data).

### 7.2 Procedure
For each service target and each L, on each side:
1. Start at the smallest declared configuration.
2. Offer R open loop (proposed SPEC 7.1), with warm-up, window and the run plan's repeats (section 8).
3. **Verify the confinement** for every run: on x86, the residue (the busy time of the CPUs outside the cpuset
   during the window, less the background load's cgroup and less the baseline) is at most 1% (proposed) of the
   counted cores' busy time, and is added to the CPU figure; on Linux on Z, exactly the counted IFLs were online;
   on z/OS, the partition ran only the listed address spaces. Check the neighbour state too. Otherwise the run is invalid (proposed SPEC 8.2). On shared Z
   engines, check that the other partitions' load stayed within the run plan's limit; a repeat that breaks it is
   invalid, not unsustained.
4. If the configuration sustains R (more than half of the repeats pass, proposed SPEC 7.4), it is the **sustaining
   configuration**. Stop.
5. Otherwise, move to the next declared configuration and repeat.
6. If none sustains R, record "not sustained within the declared configurations", with the result of the largest.
7. If the smallest declared configuration sustains R, record "sustained at the smallest declared configuration": an
   upper bound on the need, reported with the CPU time per transaction (proposed SPEC 8.3).

### 7.3 What it records
For each target and side (proposed SPEC 8.4; proposed FDR change F11):
- how the target and L were derived from both sides' calibrations;
- the sustaining configuration, and the result for the next smaller one;
- cores or engines by type, counted over every tier of the system under test, with and without the tiers that exist
  only for availability; CPU time per transaction by engine type (CPU-ACCOUNTING.md); processors and adapters listed
  but not counted; memory and last-level cache; accelerators in use, with counters; the hardware generation and class
  of each side;
- power, by the energy method where measurable ([methods/energy.md](methods/energy.md)), or "not measurable" with
  the reason;
- cost, where priced (FDR 8), with the licence metric of each product;
- utilisation per tier;
- the normalisations, each labelled: the target per counted core or engine, labelled as a normalisation of the
  target and not a capacity; per-core capacity only from a sweep of that configuration; per watt and per dollar
  only where both sides are on the same basis (proposed SPEC 8.6);
- the sentence of proposed SPEC 8.7, and no single ratio of engine counts across engine types.

## 8. Run rules
These restate SPEC section 4 and add the details the playbook needs.
1. **Warm-up** is excluded from every measurement.
   - It uses the CPU-accounting interval of both sides (on z/OS the RMF Monitor I interval, which is at least 1
     minute; proposed: 1 minute on both sides).
   - It lasts at least ten intervals (proposed: 10 minutes), and ends at the first interval boundary at which, over
     the last ten intervals, the least-squares slopes of mean latency and of CPU per transaction, each multiplied by
     the span of those ten intervals, both have a 90% confidence interval within ±10% of their mean (proposed: a two
     one-sided test at 5%).
   - If that has not happened within 30 minutes (proposed), the run is invalid.
   - Throughput is not used, because below the knee an open-loop system's throughput equals the offered rate from
     the start. p99 is not used, because the 99th percentile of a single interval is too noisy for a slope test.
2. **Steady-state window:** a fixed length set in the run plan (proposed: 10 minutes), the same on both sides, or
   whole batch cycles when batch runs alongside (SPEC 4.3).
3. **Repeats:** exactly the repeat count in the run plan (at least 6, proposed SPEC 4.1, so that the order-statistic
   interval of the knee reaches 95%), the same at every point and on both sides, in randomised order, spread evenly
   over at least 3 boots of each x86 node and 3 IPLs or re-activations of each Z partition, with the data reloaded
   each time (where the owner does not allow an IPL, 3 restarts of every component of the system under test,
   labelled). Each open-loop repeat uses its own arrival seed, derived from the run plan's base seed and the repeat
   number, and the same set of seeds is used on both sides. Record the order, the boots or IPLs, the randomisation
   seed and the arrival seeds.
4. **Statistics** (proposed SPEC 4.16): for each point, the mean with a 95% confidence interval, plus the median,
   min, max, N and the between-boot and within-boot variance components (SPEC 4.1; FDR 6). Latency percentiles come
   from merged HdrHistogram data per run (SPEC 3), with timeouts recorded at T; the confidence interval is taken over
   the per-run values. The knee is reported as in section 6.2, step 6. Verdicts follow the decision rule of the
   hypotheses addendum (Fieller, Welch, the bootstrap, the boot-sensitivity rule and Holm).
5. **Confinement,** partition rules and power settings are recorded (proposed SPEC 4.4).
6. **A manifest per run** (SPEC 4.5), for every node, partition and driver host, and **every raw file kept**
   (SPEC 4.6).
7. **The run registry and the tuning log** (proposed SPEC 4.13 and 4.14): every run is entered before it starts;
   every configuration change is entered before the run it affects.
8. **Invalid runs** are kept and reported with the reason (FDR 6): driver limit broken, the confinement check
   failed, the neighbour state not shown, the frequency below its cap, a warm-up that did not end within its
   maximum, other work in a z/OS partition, other tenants' load above its limit, a System Recovery Boost period or a
   capacity change during the window, a failover during the window, soft capping engaged, or an invariant check
   (proposed SPEC 1.8.5) failing after the run. A refusal never makes a run invalid. Validity is decided by the
   kit's committed validity checker, not by the operator (proposed SPEC 4.13). An invalid run is re-run at most
   twice per point (proposed); every invalid run is counted in the FDR, by side and reason, and a point still
   invalid after its re-runs counts as not sustained in that repeat.
9. **Between runs:** data reset (section 4.5), and the same idle period before the next run (recorded).

## 9. The bookkeeping profile and the live-database variant
Defined in proposed SPEC 1.8. The procedure is the same as for the base profile, with these additions.

### 9.1 Bookkeeping profile
1. Enable the bookkeeping steps in the configuration (kit not yet published): authentication and session lookup,
   per-client counters, permission check, access record and audit record; and TLS on the client connections, with
   the protocol version, cipher suite and certificate of proposed SPEC 1.8.1, terminated inside the system under
   test.
2. Declare the counter store's durability class (volatile or durable), the same on both sides; the counters are kept
   outside the business unit of work, and the audit record commits with it where the data variant has one (proposed
   SPEC 1.8.3). Under the D-file label "no atomic unit of work", the audit record is written immediately after the
   business update, on both sides, and the label is shown with the result.
3. Confirm before measuring that every limit is set so that **nothing is refused** at a decision point (section
   4.1). The rate-limit step counts and records "would throttle" events, by intended send time. It never rejects.
4. After each run, check:
   - the number of access records, after the buffer is flushed, equals the number of requests the system under test
     received and finished, whatever their outcome, by its own count; the driver's completed, error, refusal and
     timeout counts add up to the requests sent; any difference between the system's count and the driver's
     completed count (late responses) is reported;
   - the number of audit records equals the number of committed updates;
   - the invariant check (proposed SPEC 1.8.5) passes;
   - the number of refusals, by component and cause, is reported; any refusal at a decision point makes that step
     unsustained.
5. Report the "would throttle" count, the business outcomes by type (including duplicate-key rejections) and the
   errors by cause (proposed FDR change F6).
6. Report the base and bookkeeping profiles side by side, on the same hardware, so the cost of bookkeeping per
   request can be read directly.

The golden script (section 3, step 6) checks the access and audit record contents, with clock-derived fields masked,
and the "would throttle" and denial counts against the values the seed predicts (proposed SPEC 1.8.5).

### 9.2 Live-database variant
1. Name the form in the run plan and the FDR (proposed SPEC 1.8.4):
   - **form A, same product (platform comparison):** PostgreSQL on Linux on Z against PostgreSQL on x86 Linux; Db2
     for Linux, UNIX, and Windows on Linux on Z against the same product on x86 Linux;
   - **form B, whole stack (whole-stack comparison):** Db2 for z/OS against Db2 for Linux, UNIX, and Windows or
     PostgreSQL on x86 Linux.
2. Use the newest release of each database generally available on every platform of the comparison when the run
   plan is committed, the same version and fix level on both sides of form A for the whole series, and record it.
3. Set the same durability on both sides: the commit record forced to stable storage before the commit returns
   (proposed SPEC 1.8.4), on the same storage class, with the same log copy count and replication (proposed SPEC
   4.15). Record it, with the write-cache protection and the write-latency baseline of the log device.
4. Use the same logical schema on every database. Record the isolation level and locking clauses, logging,
   replication settings, statistics collection and bind options (sections 4.2 to 4.4).
5. Run the acceptance tests (section 3, step 7) before the first counted run, the golden check against each
   database, and the invariant check after every measured run. Where the owner does not allow an abrupt stop, label
   the configuration "durability not tested".
6. Label every form B result "whole-stack comparison". Its differences include the database engine, so they are not
   attributed to the hardware alone.
7. CardDemo's own optional features that need IMS or MQ, and its additional JCL utilities, are not run (SPEC 5). Its
   Transaction Type Management feature (Db2) is not in the first online set.
8. Publish nothing from any licensed program until the licence check of section 1, step 2, is recorded.

## 10. The scale-out tier
Defined in proposed SPEC section 9. It runs the live-database variant, in a named form, under either profile. Its
x86 cluster is single-writer, and its results are labelled "scale-out tier, single-writer". It is exploratory.

### 10.1 Set-up
1. Declare the fixed tiers in the run plan: load balancer, counter and session store, database primary, replicas
   only in variant S2 or under availability design A1 (count and mode), batch nodes if any. They do not change within
   a series.
2. Declare the availability design, the same on both sides (proposed SPEC 9.2): A0 (default), no standby on either
   side; or A1, a synchronous standby on both.
3. Declare the application node type and the largest node count, N_max.
4. Place the driver hosts on their own hardware (section 2.2), sized for the highest rate in the ladder.
5. Choose the variant: **S1** (only application nodes grow, the default) or **S2** (application nodes and read
   replicas grow together, with replica reads on).

### 10.2 Rate ladder
- Fix L and T in the run plan before K_1 is measured (proposed: L = L_cal and the T of proposed SPEC 7.5); every
  K_N and every (R, N) point uses them.
- Find the single-node knee K_1: the knee (section 6.2) of the cluster with N = 1.
- The ladder is a set of fixed target rates, set in the run plan from K_1 (proposed: K_1, 2 × K_1, 4 × K_1 and
  8 × K_1, plus further doublings while N_max allows).

### 10.3 Procedure
For each target rate R in the ladder, from lowest to highest:
1. Start at the smallest N the plan allows for R (proposed: N = 1, or the N at which the previous R was sustained).
2. Offer R open loop, with warm-up, window and repeats as in section 8.
3. Record for the point (R, N): achieved rate, p50/p90/p99/p99.9, business outcomes, errors and timeouts, CPU per
   node and per tier, database commit rate and lock waits, replication lag, counter-store hit ratio, network bytes
   per transaction, and the number of database connections.
4. At every N used in the series, find C and the knee K_N by the procedures of sections 6.1 and 6.2 (coarse and fine
   passes). Per-node capacity is K_N / N.
5. For each added node, classify it (proposed SPEC 9.3), from the 95% percentile bootstrap interval of
   D_N = K_N − K_(N−1), with the per-repeat knees at N and N − 1 resampled independently, taking the first label
   that applies: **flat** if the interval lies wholly below 0.10 × K_1 (proposed); **rising** if it lies wholly
   above 0; **inconclusive** otherwise. "Flat" is reported with its margin, never as "adds nothing".
6. Add one application node and repeat, until one of these happens:
   - R is sustained (proposed SPEC 7.4). Record N_min(R), the smallest N that sustains R. Keep going for two more
     nodes, to show whether more nodes keep helping;
   - **the scaling limit:** two consecutive added nodes are flat. Record the N where this began, and which tier was
     saturated (the one whose utilisation or wait time rose);
   - N_max is reached.
7. Move to the next R.
8. Any Universal Scalability Law fit uses the pairs (N, K_N), and is reported as analysis, never as a measurement.

### 10.4 The Z comparator
- Offer the same ladder of rates to the declared Z configuration: a single LPAR at a declared set of engine counts,
  and in a later phase a Parallel Sysplex with 1, 2 and more members, with the same availability design as the x86
  side.
- For each R, record whether it is sustained, and the smallest declared configuration that sustains it (section 7.2).
- Apply the same scaling-limit test: find C and the knee at every engine count of the declared set, class each added
  engine flat, rising or inconclusive by the rule of 10.3 step 5 (with K_1 the one-engine knee), and report two
  consecutive flat engines as the scaling limit of that configuration, with the resource whose utilisation or
  waiting rose. An x86 single-server series run beside the tier is classed the same way per added core.
- A sysplex with Db2 data sharing is multi-writer. Its comparison with the single-writer cluster is labelled
  "multi-writer sysplex against single-writer cluster".

### 10.5 What the tier reports
- For each R: N_min(R) on x86 and the smallest sustaining Z configuration, each with the inventory of section 7.3.
  Cores or engines are counted over **all** tiers of the system under test (load balancer, application, counter and
  session store, database, coupling facility), with and without the tiers that exist only for availability, and
  driver hosts are excluded.
- Per-node capacity (K_N divided by N) against N; and, for each R, the achieved rate against N.
- The scaling limit, if found, with the saturated tier, and every inconclusive step. A limit is reported as the
  limit of a single-writer design with the declared fixed tiers, not of x86 clusters in general.
- Results are labelled "scale-out tier", reported apart from single-box results, and score no registered hypothesis
  (proposed SPEC 9.5).

## 11. Batch alongside online, with priority variants
Follows SPEC 4.3. Batch is the suite of SPEC 1.3. These runs are single-box, at the core count the run plan states
(equal on both sides, and in the addendum's scoring configurations, where H3a, H3b or H5 is scored). The
interference ratio of SPEC 3 needs two online measurements, so online runs in two modes:
- **closed loop,** for online capacity with and without batch (the quantity H3a and H3b are about; H5 is scored with
  online work in this mode, as the addendum states);
- **open loop at a fixed rate** set in the run plan (proposed: 70% of C, one of the SPEC 4.2 points), for p99 with
  and without batch. Results in this mode score no registered hypothesis.

**Concurrency, fixed for the scored runs** (docs/HYPOTHESES-ADDENDUM.md):
- batch in c streams on c counted cores or engines (one per counted core, proposed), the suite's data split among
  them by account key range, each stream running the jobs one at a time in the order of SPEC 1.3, in one process,
  with GNU sort at `--parallel=1`; batch elapsed time runs from the first start to the last end;
- for H3a and H3b, online work at the connection count that gave C alone on that platform and core count (section
  6.1), with one worker per connection;
- for H5, online work at c connections (one per counted core, proposed), closed loop at zero think time;
- the runnable-thread counts of online and batch are reported with every verdict.

The measured window is the smallest whole number of batch cycles that lasts at least 600 s, the same on both sides
(SPEC 4.3).

**How batch shares the data** is declared and labelled (proposed SPEC 1.8.4): on z/OS, RLS (with DFSMS
Transactional VSAM Services where the files are recoverable) or batch through EXCI, labelled "batch through EXCI";
on Linux, the data adapter's record locking between batch and online processes.

### 11.1 Baselines
1. **Online alone,** in both modes: capacity (closed loop), and throughput and latency at the fixed rate (open loop).
2. **Batch alone:** elapsed time and CPU time per job.

### 11.2 Together
3. Start batch cycles while online runs, in each mode. The measured window spans whole batch cycles, or is
   synchronised to the cycle start (SPEC 4.3).
4. Record online capacity (closed loop), throughput and latency (open loop), the interference ratio (SPEC 3), and
   batch elapsed and CPU time per job.

### 11.3 Priority variants
Each variant is a separate point with its own repeats. Values are set in the run plan before the first run; the
values that score H3a and H3b are fixed in the addendum.

Linux, per node (H3a uses L0, and H3b uses L2):
- **L0 default:** online and batch in the same leaf cgroup at nice 0. If that is the root cgroup, autogroup is off
  (`kernel.sched_autogroup_enabled=0`) or both are started in the same session, because a new autogroup is created
  for each new session [S26]. The cgroup paths and the autogroup setting are recorded.
- **L0d distribution default (unregistered):** online and batch placed as the distribution places services by
  default (for example in separate systemd service cgroups), reported as an unregistered result.
- **L1 nice:** as L0, with batch processes at a higher nice value. Under group scheduling, a thread's nice value
  affects scheduling only relative to other threads in the same task group [S26], so the L0 placement is required.
  Each nice level shifts the CPU split between two busy tasks by about the same proportion, whatever the starting
  level; the kernel documentation's example of a one-level difference is 55% / 45% [S3].
- **L2 cgroup weights:** online and batch in separate cgroups, with `cpu.weight` favouring online (for H3b: 1000 for
  online and 100 for batch, proposed). Weights share CPU in proportion among groups that are using it, so batch can
  still use CPU that online leaves idle [S1].
- **L3 cgroup cap (optional):** batch limited by `cpu.max`, which allows a group at most a set amount of CPU time in
  each period [S1]. This is a hard cap, not a share, so it is reported apart from L1 and L2.
- **L4 scheduler policy (optional):** batch under SCHED_BATCH or SCHED_IDLE [S2].

z/OS, under WLM goal mode:
- **W0 one class:** the CICS regions, managed to region goals (the mode is selected in the JES or STC classification
  rules [S40]), and the batch jobs, classified to one service class with one execution velocity goal [S7]. It is a
  single-goal baseline, not a counterpart of L0: the two dispatchers allocate CPU by different mechanisms (on Linux,
  fair shares within a task group [S26]; on z/OS, WLM adjusting resources to meet goals [S7]). The exact definition
  is recorded.
- **W1 goals:** CICS transactions with a response-time goal (average or percentile) at high importance [S6]; batch
  in a lower-importance class with an execution velocity goal, or a discretionary goal, which WLM defines as "a
  category for low priority work for which you define no performance goals" [S31].
- **W2 cap (optional):** a WLM resource group limiting batch: a resource group's maximum limits the CPU service that
  the service classes in it may use together [S30]. The resource group type and its maximum are recorded. Reported
  apart from W1, like L3.

**What the comparison is.** The Linux variants set fixed shares or caps in advance. WLM is given goals and an
importance for each class, and determines how much resource each item of work gets in order to meet them [S7]. The
result reports, for each variant, whether the online goal (or, on Linux, the latency bound L) was met, and what it
cost batch. Hypothesis H3c makes no prediction about WLM, and none is added here.

### 11.4 Reported for each variant
Online capacity and p99 relative to online alone; batch elapsed time relative to batch alone; CPU per transaction
and per job; and, on z/OS, the performance index (PI) of each service class period, from the RMF Workload Activity
report (SMF type 72 subtype 3) [S32].

## 12. Measurement
| Source | x86 Linux (and Linux on Z) | z/OS |
| --- | --- | --- |
| Latency | Driver HdrHistogram files, per operation and interval, timeouts at T | The same driver files; SMF 110 for in-system response time |
| CPU per transaction (reported figure) | x86: `/proc/stat` busy time (user, nice, system, irq, softirq) of the CPUs in the cpuset, on every node [S38], plus the confinement residue, with the kernel accounting options recorded [S46]; Linux on Z in an LPAR: the partition's effective dispatch time from the PR/SM view, with `/proc/stat` beside it; for a guest, the hypervisor's view; `cpu.stat` and `getrusage` as breakdown | RMF / SMF 70.1 partition effective dispatch time by engine type over intervals aligned with the window, the LPAR management time and the MVS view beside it, and the capture ratio from RMF types 70 and 72 [S25]; coupling facility busy from SMF 74.4; SAP utilisation; SMF 30 (CP, zIIP, and zIIP-eligible time on CP) and SMF 110 as breakdown |
| Workload characterisation | `perf`: instructions, cycles, last-level-cache misses per transaction; on Linux on Z, the `cpum_cf` counter sets | SMF 113 (CPU-measurement facility counters) |
| System | `/proc` sampler, at the interval in the run plan | RMF / SMF 70-79, at the aligned interval |
| Database | Engine statistics (commits, lock waits, deadlocks, replication lag) for PostgreSQL or Db2 for Linux, UNIX, and Windows | Db2 statistics (SMF type 100) and accounting (SMF type 101) [S28], accounting only in the monitored point |
| Counter store and load balancer | Product counters | The chosen store's statistics |
| Accelerators | QAT telemetry or device counters; library call counts | CPU-measurement facility crypto counters, SMF 30 or 113 (verify the counter names), and DFSORT message ICE267I for Z Sort |
| Energy | RAPL on bare metal ([methods/energy.md](methods/energy.md)); sum over nodes, labelled | As in methods/energy.md section 3 |
| Steal and other load | `/proc/stat` steal | RMF partition data: the other partitions' consumption of the pool |

- CPU time is counted on the basis of [CPU-ACCOUNTING.md](CPU-ACCOUNTING.md), the same on both sides.
- All samplers write timestamped files on the common clock (section 2.4).
- Measurement must not perturb the run: samplers run at the intervals recorded in the manifest, and on z/OS the SMF
  and RMF data are extracted after the run, not polled during it.
- **Per-transaction monitoring is work.** CICS performance-class monitoring (SMF 110) and Db2 accounting traces add
  CPU to every transaction, and on Linux the samplers add some too. Counted runs run with per-transaction monitoring
  off on both sides. One point per series is repeated with it on, for the per-transaction breakdown, and the
  difference in CPU per transaction is reported as the cost of monitoring (CPU-ACCOUNTING.md, section 5).
- No user IDs, host names or other identifiers are written to published files.

## 13. Teardown
1. Stop the load, then stop batch.
2. Let in-flight requests drain, and record any that never completed.
3. Collect every raw file: driver histograms, samplers, SMF and RMF extracts, database and counter-store statistics,
   manifests, the tuning log and logs. Compute checksums (FDR 7).
4. Run the golden business-output check if the run plan calls for it; the invariant check has already run after
   every measured run.
5. Remove credentials, host names and user IDs from everything that will be published.
6. Restore or delete the data, free the nodes, return the partition to its agreed engine count, and remove any WLM
   classes, cgroups, cpusets or definitions created for the run, as agreed with the system's owner.
7. Close the series in the run registry: completed, with its FDR, or withdrawn, with the reason and the raw data
   (proposed SPEC 4.13).
8. Tell the system's owner that the window has ended.

## 14. Disclosure in the FDR
Fill in every section of [FDR-TEMPLATE.md](FDR-TEMPLATE.md), with "n/a" and a reason where a section does not apply,
plus the proposed additions F1-F12 listed in [ARCHITECTURE.md](ARCHITECTURE.md) section 10:
- first, the registered verdicts the result scores, with their intervals, adjusted levels and power statements, or
  "scores no registered hypothesis" (proposed SPEC 4.12); then whether the result is preliminary (proposed SPEC
  10);
- the machine registry entries of every machine, and whether any kit run on them is missing from the run registry
  (FDR 9);
- the neighbour state and background type on each side, and the x86 frequency cap with the measured mean
  frequencies and the Z machine's rated frequency (proposed SPEC 8.2);
- committed throughput with business rejections reported separately, and any "unequal collision rate" label
  (proposed SPEC 7.1);
- the tier, and for equal service the targets, how they and L were derived, and each side's sustaining configuration
  with its full inventory (F11), with the sentence of proposed SPEC 8.7;
- the topology, network and availability design (F1, F2, F7);
- the load driver and its validation (F3);
- the data variant, its label, durability class and form, the database, the storage class and acceptance tests, the
  counter store and the priority and confinement configuration (F4, F5);
- the request profile, the zero-refusal check and the invariant check (F6);
- the rate sweep, the per-repeat knees and any scale-out series (F8);
- CPU per transaction over all tiers, with the capture ratio and the confinement check, the workload
  characterisation, and energy over several nodes (F9, F10);
- the run registry entries and the tuning log (F12);
- the licence metric of each product, in the pricing appendix (FDR 8), and the licence check of section 1 (FDR 9);
- who configured each side, and their affiliation (GOVERNANCE.md, "Independence");
- every deviation from this playbook and from SPEC section 4, with its reason (FDR 5).

## Appendix A: run plan template
Commit and tag this before the first run of the series (section 1, step 5). Values marked (proposed) are the defaults
of proposed SPEC 7.5; changing one makes the series a labelled variant that scores nothing. The plan is written as
YAML against a schema published with the kit (proposed), so that two plans can be checked and compared by
machine; the template below lists its fields.
```
spec_version:            (for example v0.4.0)
kit_commit:              (git SHA)
run_plan_tag:            omb-series-<series id>
track:                   java | cobol; for java on Linux: openj9 | hotspot
build_policy:            compiler release and major version; -O level; -march per side; binary-byteorder native;
                         file handler and version; GMP version; library builds; default-build variant (yes/no)
tier:                    equal-cores | equal-service | scale-out
hypotheses_scored:       (IDs, each in its addendum scoring configuration, or none)
profile:                 P0-base | P1-bookkeeping (TLS 1.3, TLS_AES_128_GCM_SHA256, ECDSA P-256; proposed)
data_variant:            D-file (no-atomic-uow | atomic-uow; durable | not-durable) | D-live
scale_factors:           cache-resident S (proposed 1); memory-resident S (proposed: stored data >= 8 x largest
                         shared cache on either side)
db_form:                 n/a | A-same-product | B-whole-stack; products, versions and fix levels
durability:              commit forced to stable storage (settings)
storage:                 log and data device class; write-cache protection; replication; log copy count;
                         write-latency baseline tool
acceptance_tests:        atomicity, isolation, durability (or "durability not tested", with the reason)
counter_store:           product; durability class volatile | durable (same on both sides)
availability_design:     A0 (no standby, default) | A1 (synchronous standby on both sides)
transport:               wire protocol; receiving program on z/OS; class native | Java (same on both sides)
deployment:              z-lpar | z-sysplex | linux-on-z-lpar | linux-on-z-guest (variant) | x86-single | x86-cluster
calibration_size:        cores or engines on each side (proposed 4); z/OS (CP, zIIP) split
latency_bound_cal:       L_cal (proposed p99 of 1 s, overall; fixed before calibration)
timeout_T:               (proposed 10 x L_cal; one value for every run of the series)
service_targets:         rule: fractions of the lower calibration knee (proposed 25%, 50%, 80%); derived rates
                         added as a tagged amendment before the first counted run
latency_bounds_L:        rule (proposed 2x, 5x, 10x the higher of the two sides' mean p99 at 50% of the lower
                         knee, each <= L_cal; latency floor at 5% of that knee, 0.8 x L test); derived values added
                         as a tagged amendment
failure_threshold:       timeouts + refusals + errors <= 0.001 of the requests in the window (proposed)
sustained_rule:          every request in the window has an outcome; a target is sustained in more than half of
                         the repeats
throughput:              committed requests; business rejections per 1,000 requests by type; unequal collision
                         rate margin 1 per 1,000 (proposed)
connections:             persistent pool P = next power of two >= 2 x highest rate x L_cal, at most 1,024
                         (proposed), split over driver hosts, opened before warm-up; one outstanding request per
                         connection, or out-of-order multiplexing; no connect or TLS handshake inside the window
smt:                     per side (x86 SMT; IFL/zIIP MT mode)
frequency:               governor; x86 cap at the measured sustained all-core frequency (counted); turbo uncapped
                         and turbo off (labelled variants); measured mean frequency; Z rated frequency
machines:                machine registry IDs of every machine (named before any kit run on it)
x86_part:                SKU; class (general-purpose | frequency-optimised | core-dense | stacked cache); LLC per
                         core; memory on every channel at rated speed; reason; GA date
configurations:          per side, smallest to largest, including the calibration size and at least twice it;
                         z/OS (CP, zIIP) sequence; dedicated | shared, capped and entitled
confinement:             cpuset cpus and mems; IRQ affinity; managed-IRQ handling (queue mapping,
                         isolcpus=managed_irq); RSS/RPS/XPS; kernel threads moved; /proc/interrupts per CPU;
                         residue <= 1% and counted; z/OS allowed address spaces
neighbour_state:         loaded on both sides (80% +- 10 points, proposed; kit background | other partitions);
                         quiet (labelled variant)
z_partition:             weight, pool weight, entitlement; logical processors online; capping; HiperDispatch;
                         IRD off; boost setting and IPL time; microcode level
other_load_limit:        shared Z pool: other partitions' consumption, varying by at most 10 points (proposed)
memory_rule:             same rule on both sides: same amount | same multiple of stored data; LLC stated
size_rule:               settings that change with size, and how
rmf_smf_intervals:       RMF interval (proposed 1 min) and SYNC; SMF INTVAL and SYNCVAL; window alignment
arrival_process:         poisson | constant; base seed; per-repeat seed derivation; split rule across driver hosts
driver_limits:           per-core 70%, network 50%, schedule-lag p99.9 <= min(1% of L, 1 ms); self-test at 2x the
                         highest rate, delay 0.9 x T, completions within 0.1% (all proposed)
cross_check_margins:     +-5% of L (p99); +-0.5% of R (achieved rate) (proposed)
warmup_test:             min 10 minutes, max 30 minutes; slope test over ten 1-minute intervals on mean latency and
                         CPU per transaction, 90% interval within +-10% (proposed)
window_min:              10 (proposed; same on both sides); whole batch cycles of at least 600 s with batch
repeats_per_point:       at least 6 (proposed), over at least 3 boots or IPLs; same on both sides
request_stream:          cards per account; transactions per card; record sizes; Zipf 0.99 key access (proposed);
                         payload sizes per operation
bootstrap:               10,000 resamples; seed
sweep_steps:             coarse 10% of C (exploratory); fine 2% of C (proposed)
scale_out:               variant S1 | S2; N_max; rate ladder; fixed tiers; L and T; flat margin 0.10 x K_1
                         (proposed)
batch_online:            online mode and rate; online concurrency; batch streams (one per counted core,
                         proposed); priority variants (L0, L0d, L1-L4, W0-W2) with values
accounting_validation:   calibration load at 25%, 50%, 80% of one engine; tolerance 3% (proposed); zero-load check
tuning:                  configurer per side and affiliation; budget 40 person-hours and 30 tuning runs per side
                         (proposed); reviewer per side
monitoring:              per-transaction monitoring off in counted runs; the monitored point
security_settings:       mitigations, audit, access control (shipped defaults, or labelled)
machine_time:            planned total per series, agreed with the owner
energy:                  measured | not measurable (reason)
pricing:                 priced | not priced; licence metric per product
licence_check:           products checked on both sides, date, outcome (a gate for every licensed program)
invalid_rerun_limit:     2 per point (proposed)
```

## Appendix B: checklist
- [ ] Permission to benchmark and publish, recorded (FDR 9); IPL and abrupt-stop permissions agreed; licence terms
      of every licensed product on both sides checked and recorded
- [ ] Run plan committed and tagged before the first run; derived targets and bounds tagged before the first counted
      run; hypotheses scored listed with their scoring configurations, each registered with an archive deposit
- [ ] Series opened in the run registry; tuning log started; every run entered before it starts
- [ ] Machine time planned and agreed with the owner
- [ ] Kit commit recorded; build policy applied; build passes the golden check on every store used; acceptance tests
      passed and published (kit not yet published)
- [ ] Configuration baseline recorded, in machine-readable form, and frozen; guarantees, storage class and
      availability design match on both sides; no limit can refuse a request at a decision point
- [ ] Confinement set and verified on every run (x86 and Linux on Z); z/OS partition runs only the system under
      test; Z engines dedicated or entitled; no boost period, no capacity change, IRD off; x86 frequency capped at
      the sustained all-core frequency; neighbour state loaded and shown on both sides
- [ ] Driver self-test passed in both modes; cross-check passed its equivalence margins once per series
- [ ] Closed-loop capacity and rate sweep done; per-repeat knees and the lower-median knee recorded
- [ ] Targets and bounds derived from both sides' calibrations at the common size
- [ ] Equal-service sizing done for every target and bound; sustaining configuration and the next smaller one
      recorded; the sentence of SPEC 8.7 in the summary
- [ ] All points run with warm-up, window and the fixed number of randomised repeats over at least three boots or IPLs
- [ ] Machines named in the machine registry before any kit run on them; accounting validation passed on each
- [ ] Bookkeeping counts reconciled; invariant check passed against the audit stream; refusals reported by
      component and cause
- [ ] Live-database form named; form B results labelled "whole-stack comparison"
- [ ] Scale-out series complete, labelled single-writer; scaling limit (if any), inconclusive steps and saturated
      tier recorded
- [ ] Batch-alongside-online variants run over whole cycles, at the fixed concurrency
- [ ] Monitoring cost measured once per series; workload characterisation recorded
- [ ] Raw files collected with checksums; identifiers removed
- [ ] Teardown agreed with the owner; series closed in the run registry
- [ ] FDR complete, registered verdicts first, "preliminary" while SPEC 10 is unmet, including deviations

## Sources
Fetched 2026-11-26. The numbering matches [ARCHITECTURE.md](ARCHITECTURE.md) section 12, which also lists the
sources for what is counted as a core (S17-S21, S23, S44) and for CardDemo (S22, S43). Several IBM sources below are
for older releases (z/OS 2.x, CICS TS 5.x); each fact taken from them is to be checked against the current release
(verify).
- [S1] Linux kernel, Control Group v2: https://docs.kernel.org/admin-guide/cgroup-v2.html
- [S2] Linux kernel, CFS scheduler design: https://docs.kernel.org/scheduler/sched-design-CFS.html
- [S3] Linux kernel, scheduler nice design: https://docs.kernel.org/scheduler/sched-nice-design.html
- [S6] IBM CICS TS 5.4, z/OS Workload Manager: service classes and report classes:
  https://www.ibm.com/docs/en/SSGMCP_5.4.0/fundamentals/workload/workload_lp_classes.html
- [S7] IBM CICS TS 6.2, How it works: z/OS Workload Manager:
  https://www.ibm.com/docs/SSGMCP_6.2.0/fundamentals/wlm/zos-wlm.html
- [S8] wrk2 README: https://github.com/giltene/wrk2
- [S9] PostgreSQL documentation, synchronous replication:
  https://www.postgresql.org/docs/current/warm-standby.html#SYNCHRONOUS-REPLICATION
- [S25] J. Burg and B. Snyder, IBM, *z/OS: Calculating the Capture Ratio* (white paper, 2017-06-29):
  https://www.ibm.com/support/pages/system/files/inline-files/zOS_-_Calculating_the_Capture_Ratio_v1.pdf
- [S26] Linux man-pages, sched(7), "The autogroup feature" and "The nice value and group scheduling":
  https://man7.org/linux/man-pages/man7/sched.7.html
- [S28] IBM Db2 12 for z/OS, Installation step 9: Record Db2 data to SMF (statistics, SMF type 100; accounting, SMF
  type 101; other trace records, SMF type 102):
  https://www.ibm.com/docs/SSEPEK_12.0.0/inst/src/tpc/db2z_recorddb2data2smf.html
- [S29] IBM z/OS 2.2, Reviewing z/OS parameter settings (IIPHONORPRIORITY; YES "is the default"):
  https://www.ibm.com/support/knowledgecenter/en/SSLTBW_2.2.0/com.ibm.zos.v2r2.ieaw100/ieaw100160.htm
- [S30] IBM z/OS 2.4, Defining resource groups:
  https://www.ibm.com/support/knowledgecenter/en/SSLTBW_2.4.0/com.ibm.zos.v2r4.ieaw100/rgr.htm
- [S31] IBM Db2 12 for z/OS, z/OS performance options for Db2 (WLM goal types: response time, execution velocity,
  discretionary): https://www.ibm.com/docs/SSEPEK_12.0.0/perf/src/tpc/db2z_zosperfoptions.html
- [S32] IBM z/OS 2.2 RMF, Workload Activity - SMF record type 72-3 (performance index per service class period):
  https://www.ibm.com/support/knowledgecenter/SSLTBW_2.2.0/com.ibm.zos.v2r2.erbb200/erbb200198.htm
- [S33] IBM z/OS 3.1, CLOCKxx (time of day parameters; STP):
  https://www.ibm.com/docs/SSLTBW_3.1.0/com.ibm.zos.v3r1.ieae200/clock.htm
- [S35] PostgreSQL documentation, Write Ahead Log settings:
  https://www.postgresql.org/docs/current/runtime-config-wal.html
- [S36] PostgreSQL licence: https://www.postgresql.org/about/licence/
- [S37] Linux kernel, EEVDF scheduler: https://docs.kernel.org/scheduler/sched-eevdf.html
- [S38] Linux man-pages, proc_stat(5) (the columns of the cpu lines in `/proc/stat`):
  https://man7.org/linux/man-pages/man5/proc_stat.5.html
- [S39] Linux kernel, SMP IRQ affinity: https://docs.kernel.org/core-api/irq/irq-affinity.html
- [S40] IBM CICS TS 5.4, z/OS Workload Manager: region and transaction goals:
  https://www.ibm.com/docs/SSGMCP_5.4.0/fundamentals/workload/workload_lp_jes.html
- [S44] IBM Support, Introduction to Logical Partitions: https://www.ibm.com/support/pages/introduction-logical-partitions
- [S45] IBM CICS TS 6.x, Making applications threadsafe:
  https://www.ibm.com/docs/SSJL4D_6.x/fundamentals/threadsafe/threadsafe_making.html
- [S46] Linux kernel, `init/Kconfig` and `arch/s390/Kconfig` (CPU time accounting options):
  https://github.com/torvalds/linux/blob/master/init/Kconfig
- [S47] GnuCOBOL 3.x, `config/default.conf` ("binary-byteorder: big-endian"; read on the 3.x branch, verify for the
  release used): https://sourceforge.net/p/gnucobol/code/HEAD/tree/branches/gnucobol-3.x/config/default.conf
- [S56] Linux man-pages, chcpu(8) ("Set the CPU dispatching mode (polarization)"; horizontal and vertical):
  https://man7.org/linux/man-pages/man8/chcpu.8.html
