# Architecture model: the systems under test (draft for comment)

Status: **notes**. This page describes the deployments the benchmark compares and how each one maps to the Full
Disclosure Report. No kit code is published and no kit run has been made. Items marked **(proposed)** are not yet
decided. Facts about IBM and Linux products carry a source (section 12); anything without one is marked
"(source needed)", and anything still to be checked against the product release in use is marked "(verify)".

How the code is layered (one body of business logic, with data, transport and accelerator adapters) is described in
[PORTING-zOS.md](PORTING-zOS.md). This page places those layers on hardware. How CPU time is counted is in
[CPU-ACCOUNTING.md](CPU-ACCOUNTING.md).

## 1. What is compared
The benchmark runs one workload (SPEC 1) on two **reference deployments**:
- **Z:** an IBM Z or LinuxONE system, first as a single LPAR, later as a Parallel Sysplex.
- **x86:** first a single server (SPEC 2, phase 1), then an N-node cluster.

It compares two ways of doing **the same business work with the same guarantees**. Comparing boxes core for core is
one tier, not the whole aim. The guarantees are listed in section 2. Each deployment is configured from the kit's
published configuration checklist for its platform (proposed SPEC 10), and the FDR names who configured each side
and their affiliation (GOVERNANCE.md, "Independence"), and both sides have the same tuning budget (proposed SPEC
4.18). Tuned and untuned settings are both reported; where a licence term prevents publishing a configuration, the
FDR names the term and the configuration withheld (README, "Fair comparison"). Each side's configuration is offered for factual review to the vendors of
that platform before publication, and any review is recorded in FDR 9 (GOVERNANCE.md, "Vendor review and right of
reply").

There are three comparison tiers, and every result names its tier (proposed SPEC 2):
1. **Equal core counts on a single box (the registered per-core hypotheses).** The same number of cores or IFLs on
   each side, measuring closed-loop capacity. This answers the per-core questions in HYPOTHESES.md (H1: one IFL
   against one x86 core) in the scoring configurations named in
   [HYPOTHESES-ADDENDUM.md](HYPOTHESES-ADDENDUM.md). Their verdicts are reported first in every FDR
   that scores them (proposed SPEC 4.12).
2. **Equal service (exploratory; proposed SPEC 8).** For a stated throughput within a p99 bound fixed before the
   runs, what each side needs: cores or engines by type, memory, accelerators, power where measurable and cost where
   priced, all disclosed. Each side is run at a declared set of sizes, and the smallest size that sustains the
   target is the result. No prediction about it is registered; it scores no registered hypothesis and gives no
   headline result (proposed SPEC 10).
3. **Scale-out tier (proposed SPEC 9).** The x86 side grows node by node while the offered rate is held fixed. Its
   cluster is single-writer: one database primary takes every write. It maps where adding nodes stops raising
   capacity, and it is reported apart from single-box results.

Per-core figures from any tier are a disclosed normalisation, shown beside each result.

### 1.1 Why an equal-service comparison, beside equal cores
- **What the comparison answers.** Equal service asks what each side needs to deliver a stated throughput within a
  stated response time, which is the question a capacity plan answers. It also allows a cluster to be compared with
  a single system. Comparisons at a fixed offered load, reporting the resources each side needs, are one of the
  normalisations prior work has used (docs/prior-art.md, "How others normalised across architectures"); the only
  measured precedent we found is a study commissioned by IBM (AN01), and we found no independent one.
- **Work outside the counted cores, on both sides.** Neither an equal-core run nor an equal-service run counts all
  the processing involved, and equal service does not change that. What it changes is that the uncounted processors,
  adapters and accelerators are listed beside every result, on both sides (section 1.2):
  - On IBM Z, System Assist Processors (SAPs) run I/O operations. They are part of the base configuration, and
    adding SAPs does not change the machine's capacity setting or its MSU rating (IBM's capacity measure, which is
    also used for software pricing) [S17]. Integrated Firmware Processors (IFPs) handle firmware and PCIe
    functions [S23]. On z17, a Data Processing Unit (DPU) on each processor chip takes over the functions that I/O
    adapters used to do in their own chips [S23]. I/O also passes through channels and adapters, and some network
    adapters offload part of the TCP/IP processing [S23].
  - Each Z core has its own CPACF (the on-core cryptographic unit) [S17]. IBM describes the Z Sort accelerator
    (SORTL, used by DFSORT) as "on-core" on z16 [S17], and lists it among each core's integrated accelerators on z17
    [S23]. The DEFLATE compression accelerator (zEDC) works "on a processor chip level" [S17] ("one dedicated
    compression coprocessor per PU" [S17][S23], where PU means the processor chip). The IBM Integrated Accelerator
    for AI (the on-chip AIU) is "implemented on each PU chip and shared among all cores" [S23]; on z17, IBM states
    that each AI accelerator can "accept work from any core in the same drawer" [S55], so the number of AI
    accelerators a partition's cores can reach is counted per drawer. CMPSC, the compression call instruction used
    for dictionary-based compression (for example Db2 for z/OS row compression), is a separate facility from zEDC,
    and is listed with the accelerators. z17 also offers the IBM Spyre Accelerator as an adapter [S23].
  - x86 servers also move work off the cores, for example segmentation offload in network adapters [S21], and on
    some parts the QAT, IAA (In-Memory Analytics) and DSA (Data Streaming) accelerators (docs/platforms.md, "Cloud
    bare metal with QAT/AMX").
- **Logical processors.** Every processor a partition sees is a logical processor, which PR/SM dispatches on
  physical processors that are either dedicated to the partition or shared across partitions [S23]. For shared
  processors, the partition's share is set by its weight relative to the other active partitions, excess capacity
  from one partition can be used by another, and per-partition capping limits this [S44]. A cap is a ceiling, not a
  guarantee: a capped partition still competes by weight. A shared IFL is therefore not a fixed amount of hardware,
  and its result depends on the other partitions' load. Counted runs use dedicated engines, or shared engines whose
  weight alone entitles them to at least the capped capacity, with as many logical processors online as the cap
  (proposed SPEC 8.2), and the FDR records the other partitions' load.
- **Why per-core figures stay, as a normalisation.** Much software is licensed per core. IBM's Processor Value Unit
  licensing counts each processor core, and counts an IFL or CP engine as one core, but the PVUs per core differ by
  processor: 120 for a z17 ME1 core, and 70 for a core of a two-socket Intel Xeon server or of an AMD EPYC server
  [S19]. A per-core figure is therefore not a per-licence-unit figure. Much z/OS software is instead charged on MSUs:
  IBM describes Tailored Fit Pricing as an alternative to the rolling four-hour average, with compute "measured on
  the basis of MSUs consumed" [S20], and "the addition of ICFs, IFLs, and zIIPs does not change the system capacity
  setting or its millions of service units (MSU) rating" [S23]. The FDR states the licence metric of each product.

### 1.2 What is counted
| | Z | x86 |
| --- | --- | --- |
| Counted as cores or engines, by type | CPs, zIIPs, IFLs and ICFs (or the shared CPs a coupling facility runs on), each dedicated or shared; for shared engines, weight and capping; the multithreading (MT) mode of IFLs and zIIPs | Physical cores, by CPU model; SMT state stated; for a VM, whether a vCPU is a core or a thread |
| Listed, not counted as cores | SAPs, IFPs, the on-chip DPU (z17), channels, network adapters and their offloads, cryptographic adapters | Network adapters and their offloads (and any DPU or SmartNIC), storage controllers |
| Listed on both sides | The storage subsystem and its processors; on Z, the LPAR management time on the `*PHYSICAL*` line of the RMF Partition Data Report, and each partition's total minus effective dispatch time (SMF70PDT minus SMF70EDT; verify the field names); the SAP utilisation | (same, without the Z items) |
| Counted wherever they run | Any appliance, gateway, proxy or SmartNIC that runs a step of the request pipeline, or terminates TLS sessions, on processors of its own and independently of the counted cores (proposed SPEC 8.4). A device that only performs an operation requested by a counted core (CPACF, Crypto Express, QAT, IAA, a network adapter's inline TLS record encryption) is listed, not counted | (same) |
| Disclosed beside the count | Memory; accelerators present and in use, with counters (CPACF, on-chip compression through DFLTCC, CMPSC, Z Sort, the AI accelerator, Crypto Express adapters, and IBM Spyre Accelerator adapters on z17); the hardware generation; power where measurable; cost where priced | Memory; accelerators present and in use, with counters (AES-NI/SHA, AVX-512, AMX, QAT, IAA, DSA); the hardware generation; power where measurable; cost where priced |
| Never counted | Driver hosts | Driver hosts |

Cores are counted over every tier of the system under test: application, database, cache, load balancer and
coupling facility partitions alike. On x86, the counted cores form a cpuset in one NUMA node that holds every process
of the system under test, the confinement is verified for every run, and the residue on the other CPUs is counted;
on z/OS, the partition runs only the system under test; on Linux on Z, Linux runs directly in an LPAR with exactly
the counted IFLs online. Every Z partition is counted on the PR/SM view, its effective dispatch time (proposed SPEC
8.2; CPU-ACCOUNTING.md). The neighbour state is the same on both sides (proposed SPEC 8.2).

## 2. The equivalence contract
Both deployments must give each request the same guarantees. Where one cannot, the FDR says so, and the result is
labelled with the difference. A comparison never pairs runs that differ in these labels (proposed SPEC 4.11).

| Guarantee | Meaning |
| --- | --- |
| G1 Atomic unit of work | Each request is one unit of work. It commits completely or not at all. (In the file-based variant this holds only under the "atomic unit of work" label; see below.) |
| G2 Durable commit | A committed change survives a crash of the process, the operating system or the machine that made it: the commit record is on stable storage (a disk, or a write cache protected against power loss) before the response is sent. This is the setting for every counted result in the live-database variant and under the "atomic unit of work" label, on both sides: for example, PostgreSQL with `fsync=on`, `synchronous_commit` not `off` and `full_page_writes=on` [S35]; Db2 with its log forced to stable storage before the commit returns, with the settings named in the FDR; CICS recoverable resources, whose system log (DFHLOG) and any forward-recovery logs are MVS System Logger log streams: for each, the FDR states whether it is a coupling-facility or a DASD-only log stream and, for a coupling-facility log stream, whether it is duplexed to staging data sets (source needed). The storage class, the log copy count and the write-cache protection of the log device are declared and the same on both sides, and durability is shown by the acceptance tests, not only configured (proposed SPEC 4.15). A weaker setting is a labelled variant, never paired with a durable one. |
| G3 Isolation for updates | A record read for update stays locked until commit or rollback. A second updater waits or gets a conflict status. It never sees a half-applied change (proposed SPEC 1.8.4). In the file-based variant this holds in full only under the "atomic unit of work" label; see below. |
| G4 Identical business results | The golden script (a fixed request stream from the seed, run serially; SPEC 1.0) gives identical business output on every platform, compared byte for byte after binary fields are put in one byte order and text in one character encoding (SPEC 1.0, build policy; 1.5). A measured run is concurrent, and its final state depends on commit order, so after a measured run the check is of invariants, against an application-level audit stream that the system under test writes alongside its work: unique keys, no lost or duplicated update, balances conserved, and counts that reconcile (proposed SPEC 1.8.5). |
| G5 Bookkeeping on every request | In the bookkeeping profile (proposed SPEC 1.8): authentication, counters, the permission check and an access record for every request, and an audit record for every update. |
| G6 Nothing is refused by design | No component is configured to reject, shed or throttle a request during a measured run. Queuing is allowed and shows up as latency. A refusal that happens anyway is counted as a timeout and reported by component and cause (proposed SPEC 7.1). |
| G7 Stated recovery and availability design | The design recovery point (how much committed work a single failure can lose) is stated for each deployment, including the counter store's durability class (proposed SPEC 1.8.3). The availability design (A0, no standby; A1, a synchronous standby) is the same on both sides of a comparison, or the comparison is labelled "unequal availability design" and reported as two results (proposed SPEC 9.2). |

G1 and G2 matter on z/OS as shipped: CardDemo's files are defined RECOVERY(NONE) (PORTING-zOS.md), and CICS backs out
uncommitted changes only for resources defined as recoverable [S5]. A z/OS configuration that meets G1 therefore
needs recoverable files, or Db2, and that is part of the disclosed configuration. If batch shares recoverable files
with CICS through RLS, DFSMS Transactional VSAM Services (DFSMStvs; "TVS" in PORTING-zOS.md), a separately priced
feature, is also needed (PORTING-zOS.md).

**The file-based variant.** On Linux, GnuCOBOL indexed files and the Java track's keyed store give record locking,
but no unit of work across several files is assumed unless the kit adds one. File-based results therefore carry
one of two labels, the same on both sides (proposed SPEC 1.8.4): **"no atomic unit of work"** (z/OS files
RECOVERY(NONE), as shipped; Linux files as the store provides them), or **"atomic unit of work"** (z/OS recoverable
files; on Linux, a transaction layer in the data adapter, which is described and disclosed). G1 and G2 apply in full
only to the second label and to the live-database variant. G3 does too. Under "no atomic unit of work", a record read
for update stays locked until it is rewritten or released, not until a commit: on z/OS through CICS file control for
a non-recoverable file, and on Linux through the file handler's record lock (verify each against the release in
use). The label states this. Each D-file result also declares its durability class (durable or not durable),
established by the acceptance tests and the same on both sides (proposed SPEC 1.8.4).

## 3. The request path (both deployments)
```
 load driver hosts (own hardware, never on the system under test)
      |  TCP, request stream at a fixed offered rate (open loop)
      v
 [entry]  network adapter, TCP/IP stack, load balancing
      v
 [transport]  listener / worker pool / CICS entry program
      v
 [request pipeline]
      1 authenticate: validate the session token, look up the session        (bookkeeping profile)
      2 count: update per-client counters in the counter store; record       (bookkeeping profile)
        "would throttle" (outside the business unit of work)
      3 authorise: check principal x operation x account                     (bookkeeping profile)
      4 business transaction (CardDemo operation, plus accelerated steps if on)
      5 access record for every request; audit record for every update      (bookkeeping profile)
      6 commit step 4 and its audit record as one unit of work, then respond
      v
 [data]  file-based store or live database; counter and session store; audit sink
```
- Steps 1-3 and 5 are the bookkeeping steps (proposed SPEC 1.8.3). In the base profile only steps 4 and 6 run.
- The counters of step 2 are kept in a counter store outside the business unit of work. Each counter update is
  atomic on its own, and the store's durability class (volatile or durable) is declared and the same on both sides.
- Access records may be buffered rather than committed with the unit of work; the buffering is disclosed, and the
  same rule applies on both sides.
- The driver measures latency from the **intended** send time of each request, so a stalled system cannot hide its
  own delay (coordinated omission) [S8].
- **Back-pressure, never refusal:** when the system falls behind, requests queue, and the wait is part of their
  latency. A request with no response within the timeout is counted as a timeout and recorded at the timeout value
  (proposed SPEC 7.1).

## 4. Reference deployment Z

### 4.1 Single LPAR (phase 2 baseline)
| Element | Role | Notes |
| --- | --- | --- |
| LPAR | CPs at a stated capacity setting (full or sub-capacity; sub-capacity settings apply to CPs only), and zIIPs, which run at full speed [S23] (IFLs serve the Linux on Z deployment, 4.3); weight, entitlement and capping; the partition runs only the system under test (proposed SPEC 8.2) | Recorded as in PORTING-zOS.md, "Measurement on z/OS". z/OS needs at least one CP, and zIIPs need at least one CP [S23]. |
| Network entry | Network adapter and the z/OS TCP/IP stack | The driver connects over the network. An in-box connection (for example, HiperSockets [S15]) is never used for the driver. |
| CICS TS region(s) | Transaction manager: runs the transport entry program, the bookkeeping steps and the business logic | One region, or a routing region and application regions, declared (proposed SPEC 1.1). A program defined QUASIRENT always runs on the QR TCB [S45]. Defining a program THREADSAFE states only that its application logic is threadsafe; CICS does not make it so, and the program must serialise its access to shared resources [S45], so a THREADSAFE definition follows a review of the code. Several application regions are the alternative; with non-RLS VSAM they need a file-owning region and function shipping, or RLS (PORTING-zOS.md). The choice changes CPU per transaction. |
| Data, file-based variant | VSAM KSDS under CICS file control | Recoverable files for G1/G2 under the "atomic unit of work" label. For batch sharing: RLS with a coupling facility lock structure, or batch access through EXCI (SPEC 4.3) |
| Data, live-database variant | Db2 for z/OS, reached from CICS. It is the Z side of form B, the whole-stack comparison: against Db2 for Linux, UNIX, and Windows or against PostgreSQL on x86 (proposed SPEC 1.8.4) | Product version and fix level, isolation level and locking clauses, logging and buffer pools are recorded |
| Counter and session store | A keyed store chosen by the implementer: a VSAM file, a Db2 table, or a CICS facility (for example data tables, or in a sysplex coupling facility data tables or a named counter server; source needed) | The choice and its durability class are disclosed. Equivalent x86 choices are in 5.2. |
| Security manager | An external security manager such as RACF, called by CICS for security checks [S11] | See section 6, "Security and access control", for how platform checks relate to step 3. |
| Platform audit | SMF records: RACF writes type 80 records for logged security events [S12]; CICS monitoring writes type 110 (PORTING-zOS.md) | The logging and monitoring options in force are recorded, because they add work per request, and their cost is measured (PLAYBOOK.md, section 12). |
| WLM, goal mode | Service classes with goals and importance. CICS transactions may have only response-time goals (average or percentile) [S6]. A CICS region managed by its address-space goal (JES or STC rules) may have only an execution velocity goal [S7]. | The service definition excerpt is part of the FDR (FDR 3). |
| Batch | JES initiators, JCL, DFSORT | Same LPAR as online in the batch-alongside-online runs |
| Accelerators | CPACF, on-chip compression (zEDC, through DFLTCC), CMPSC, the Z Sort accelerator and the IBM Integrated Accelerator for AI (which IBM also calls the AIU [S23]; here "the on-chip AIU", used through the NNPA instruction), where present [S23] | docs/platforms.md; each is also switched off, and its use shown by counters (SPEC 1.4) |
| Measurement | RMF / SMF 70-79 (partition effective dispatch time by engine type over intervals aligned with the window, LPAR management time, capture ratio, coupling facility busy from SMF 74.4, SAP utilisation), SMF 30, SMF 110, and the CPU-measurement facility counters (SMF 113) | CPU-ACCOUNTING.md; PORTING-zOS.md |

**A coupling facility on a single LPAR.** VSAM RLS needs a coupling facility lock structure (PORTING-zOS.md). A
"single LPAR" configuration that uses RLS therefore also has a coupling facility. The FDR lists it, with its
processors (dedicated or shared ICFs, or shared CPs; on z17 a coupling facility can no longer use dedicated CPs
[S23]) and where it runs. Shared coupling facility processors add latency to every coupling facility request, and
the coupling facility's dispatching setting is recorded. Its busy time is counted from the coupling facility activity
data (SMF type 74 subtype 4), not from its dispatch time, because a coupling facility on dedicated processors polls
for work (CPU-ACCOUNTING.md, section 4).

### 4.2 Parallel Sysplex (later phase)
A Parallel Sysplex adds:
- two or more z/OS images (on one CPC or several);
- one or more coupling facilities holding the shared structures:
  - the Db2 lock structure, which protects shared Db2 resources [S13];
  - Db2 group buffer pools, which cache shared data pages for the members [S13];
  - the Db2 shared communications area [S13];
  - the VSAM RLS lock structure (PORTING-zOS.md);
  - the RACF database cache, if RACF sysplex data sharing is used [S34];
- **Sysplex Distributor** spreads incoming TCP connections across target stacks by a configured distribution
  method: WLM system weights (the default), WLM server-specific weights, weighted active, round robin or hot
  standby [S14]. The method is recorded;
- **CICSPlex SM workload management** routes work to the target region likely to perform best, with one of four
  algorithms (queue, goal, link neutral queue, link neutral goal) [S16];
- WLM manages goals across the whole sysplex: importance is relative to "other work in the sysplex" [S6].

IBM states that multiple coupling facilities are always required for high availability, because a single coupling
facility is a single point of failure for a Db2 data sharing group [S13b]. The FDR records how many coupling
facilities there are, where they run, and which structures are duplexed. A sysplex with Db2 data sharing is
multi-writer, and is labelled so when it is compared with the single-writer x86 cluster (section 5.2).

### 4.3 Linux on Z
On LinuxONE, or Linux in an LPAR, a z/VM guest or a KVM guest on IBM Z [S23], the deployment has the single-server
layout of 5.1, on IFLs. In the live-database variant it runs PostgreSQL, or Db2 for Linux, UNIX, and Windows,
compared with the same product on x86 (form A, same product; proposed SPEC 1.8.4; Db2 only in a release whose IBM
system requirements list Linux on IBM Z, cited in the FDR). Counted runs run Linux directly in an LPAR, with exactly the
counted IFLs online, dedicated or entitled as in proposed SPEC 8.2, and the processes of the system under test in a
cpuset. A z/VM or KVM guest is a labelled variant, with the hypervisor's time counted (CPU-ACCOUNTING.md, section 2).

## 5. Reference deployment x86

### 5.1 Single server (phase 1 baseline)
All tiers on one machine: the transport and worker pool (or JVM), the data store (GnuCOBOL indexed files, an
embedded store, or in the live-database variant a local database server: Db2 for Linux, UNIX, and Windows or
PostgreSQL, as the form requires), and batch. It is the comparator for Linux on Z (4.3) at equal core counts, in the
scoring configurations of the registered hypotheses; no registered hypothesis compares it with the z/OS LPAR of 4.1.
At equal service it is compared with either, and is run at a declared set of core counts, each a cpuset that holds
every process of the system under test.

### 5.2 N-node cluster (scale-out tier, single-writer)
| Element | Role | Notes |
| --- | --- | --- |
| Load balancer | Spreads connections or requests across application nodes | One under availability design A0, or a pair under A1, the same design on both sides (G7). Connection limits and rate limits are off (G6). |
| Application nodes 1..N | Stateless: transport, bookkeeping steps, business logic | No session or counter state is held only on one node. Nodes are identical. |
| Counter and session store | Session store and per-client counters, for example a cache tier | Product, version, persistence setting and eviction policy are recorded. Its durability class (volatile or durable) is declared and is the same as on the Z side (G7; proposed SPEC 1.8.3). |
| Database primary | All writes; all reads that feed an update | Live-database variant only: Db2 for Linux, UNIX, and Windows or PostgreSQL, in the declared form |
| Database replicas | Only in variant S2 (read-only queries; counted with the load-serving tiers) or under availability design A1 (a synchronous standby; counted separately as availability-only) | Synchronous or asynchronous is recorded. With synchronous replication, each commit waits for the standby, and the minimum added wait is the round trip to it [S9]. |
| Audit sink | Durable store for audit records | Audit records commit with the business unit of work (G1), for example in the primary database. |
| Batch nodes | Run the batch suite against the primary | Batch reaches the data over the network, as online does |
| Network fabric | Switches and links between all tiers | Link speeds, the switch model and the measured round-trip time between tiers are recorded |
| Scheduling on each node | The Linux scheduler, nice values and cgroup v2 controls | See section 6. An orchestrator, if used, is disclosed with its settings. |
| Accelerators | AES-NI/SHA, QAT, IAA, DSA, AMX, AVX-512, where present | docs/platforms.md |
| Measurement | `/proc/stat` per CPU, cgroup `cpu.stat`, database and cache statistics, load balancer counters | One clock source for all nodes ([PLAYBOOK.md](PLAYBOOK.md), section 2.4) |

**Single-writer, by declaration.** One database primary takes every write. Results of this tier are labelled
"scale-out tier, single-writer", and a scaling limit found here is reported as the limit of a single-writer design
with the declared fixed tiers, not of x86 clusters in general. A multi-writer x86 variant (a shared-data or
distributed database, as declared) is optional and may be added in a later spec version. No claim is made that the
single-writer cluster matches Parallel Sysplex data sharing.

**The file-based variant does not scale out.** GnuCOBOL indexed files and embedded stores live on one node. The
scale-out tier therefore runs the live-database variant. A shared-file design would add a different coordination
layer, so it is out of scope unless a later spec version defines it.

**Replica reads (proposed default: off).** Reading from a replica can return data older than the primary's, which
can change business results (G4). By default every business read goes to the primary, and replicas serve
availability only. Routing read-only operations (account view, card list, transaction list) to replicas is a
declared variant. When it is used, replication lag is measured, and the invariant check (G4) is run after each point.

## 6. Responsibility map
Which component does each job. "Application" means the benchmark's own code, which is the same on both sides.

| Concern | Z: single LPAR | Z: Parallel Sysplex adds | x86: N-node cluster |
| --- | --- | --- | --- |
| Request distribution | TCP/IP stack to the CICS listener; CICS routing between regions, if several | Sysplex Distributor across images [S14]; CICSPlex SM routing [S16] | Load balancer across application nodes |
| Transaction management (units of work, commit, rollback) | CICS recovery manager; Db2 in the live-database variant [S4][S5] | Db2 data sharing; RLS across images | The application's transaction calls on the database; for the file-based variant, the data adapter's transaction layer under the "atomic unit of work" label |
| Concurrency control | CICS file control record locks or RLS locks; Db2 locks | Coupling facility lock structures [S13] | Database row locks on the primary; atomic operations of the counter store for counters |
| Scheduling and priority | WLM, which determines how much resource each item of work gets, by goal and importance [S6][S7] | WLM goals across the sysplex | The Linux scheduler on each node (CFS before kernel 6.6, EEVDF from 6.6 [S2][S37]); nice values and cgroup v2 `cpu.weight` / `cpu.max` [S1][S3]. No component sets goals across nodes unless an orchestrator is configured to |
| Security and access control | CICS calls the external security manager [S11]. The application's step 3 (principal x operation x account) runs on top | The same; with RACF sysplex data sharing, the RACF database is cached in a coupling facility, and every system that uses it must be in the same sysplex [S34] | The application's steps 1 and 3, against the session store and the permission data |
| Session state | The session store of 4.1 | Shared across images (implementation choice, disclosed) | The counter and session store |
| Rate-limit bookkeeping | The application (step 2) against the counter store | Counters shared across images | The application against the counter and session store |
| Audit | The application's access and audit records (step 5); platform SMF records in addition [S12] | The same | The application's records (step 5); system logs in addition, as configured |
| Recovery and restart | CICS backs out uncommitted changes to recoverable resources when a unit of work fails, including when CICS ends abnormally [S5]; Db2 restart from its log (source needed) | Another image takes the work; the coupling facility holds shared state | The load balancer drops a failed application node; under A0 a database primary failure stops all writes until it restarts, and under A1 the database fails over to the standby; the counter store restarts according to its durability class |
| Batch and online coexistence | WLM classes on one LPAR; RLS or EXCI for shared files (SPEC 4.3) | Batch on another image, sharing data through the coupling facility | Batch nodes or shared nodes; the database handles concurrent access |
| Accelerators | CPACF, on-chip compression, CMPSC, Z Sort and the AI accelerator, used through instructions issued by the application's own task [S23] | The same, per image | CPU instructions; QAT, IAA, DSA or AMX where present |
| Accounting and monitoring | RMF partition busy by engine type, SMF 30 / 110 (CPU-ACCOUNTING.md) | Sysplex-wide RMF | Per-CPU `/proc/stat` on each node, database and cache statistics |

**Platform checks against application checks.** Step 3 is defined as a data-driven check that the same code does on
both sides (proposed SPEC 1.8). A deployment may *also* use platform security (for example, CICS resource security
through RACF). That work is extra, and the FDR records it. Whether a platform check may *replace* step 3 is an open
question (section 11).

## 7. Costs outside the application, on both sides
Every cost below is counted per request, so that the totals can be compared. The single-box costs of each side are
compared with each other, and the cluster costs with the sysplex costs.

**Network hops.**
- x86 single server: driver to the server's TCP/IP stack. After that, calls between tiers stay inside the server
  (loopback or local inter-process communication).
- Z single LPAR: driver to the TCP/IP stack. After that, calls between CICS, the data manager and the security
  manager stay inside the LPAR.
- x86 cluster: driver to load balancer, load balancer to application node, node to the counter and session store
  (once or more for steps 1 and 2), node to database primary (several round trips per unit of work), primary to
  synchronous standby (under availability design A1 only), node to load balancer to driver.
- Parallel Sysplex: coupling facility requests and routing between images are added to the single-LPAR path.
- The FDR records, for each deployment, the number of network round trips per operation type and the measured
  round-trip time on each link (proposed FDR change F2).

**Per-request work outside the application, on both sides.**
- On z/OS: security manager calls, SMF and RMF recording, CICS dispatching, and cross-memory calls to the data
  manager.
- On Linux: system calls, interrupt and softirq processing for the network and storage, and the database's
  background processes.
- All of it is measured in the CPU time per transaction of the whole system under test, like any other cost
  (CPU-ACCOUNTING.md; F9).

**Replication.**
- With synchronous replication, every commit waits at least one round trip to the standby [S9].
- Asynchronous replication avoids the wait, but a primary failure can lose committed work. The recovery point is
  then stated (G7).
- In a Db2 data sharing group, when a member updates a shared page, the coupling facility invalidates the copies of
  that page in the other members' buffer pools [S13b]. This is the sysplex counterpart of replication cost.

**Coordination.**
- Global locks: coupling facility lock structures on Z [S13], and row locks on the primary database for x86.
- Connection pools: each application node holds database connections. As N grows, the primary serves more
  connections.
- Shared counters: the rate-limit counters of step 2 are hot keys, updated by many nodes at once.

**Serialization.**
- One write path: on the single-writer x86 cluster all writes go to one primary. In a sysplex with data sharing,
  writes can go through any member, at the cost of coupling facility traffic.
- The CICS QR TCB: quasi-reentrant programs run one at a time on it (PORTING-zOS.md).
- **A workload hot spot on both sides:** CardDemo assigns a new transaction ID as the highest existing key plus one,
  by a backward browse of the transaction file, in transaction add (COTRN02C) and bill payment (COBIL00C), with no
  serialisation before the write [S43]. Concurrent adds contend on the end of the key range, and two of them can
  compute the same ID; the second then gets a duplicate-key response (DUPKEY/DUPREC), which the programs report
  ("Tran ID already exist...") without retrying [S43]. In bill payment, that response does not stop the account
  update that follows it: the program goes on to subtract the payment from the balance and rewrite the account,
  so the balance changes with no transaction record written [S43]. This is part of the workload, on every
  platform, and the port keeps it (SPEC 1.1). Duplicate-key rejections are counted as business errors (proposed
  SPEC 1.8.2), and the invariant check accounts for the bill-payment case (proposed SPEC 1.8.5).
- Data conversion and marshalling: EBCDIC/ASCII translation on z/OS, and request and record encoding between tiers
  on x86.

**A model for reading the scale-out curve.** The Universal Scalability Law separates *contention* (waiting for
shared resources) from *coherency* (the delay for data to become consistent across nodes). In that model, any
coherency cost means that throughput eventually falls, not just levels off, as nodes are added [S10]. The model
describes capacity against N, so the scale-out tier measures the knee K_N at each node count (proposed SPEC 9.3).
Fitting the model to the pairs (N, K_N) is optional, and is reported as analysis, never as a measurement.

## 8. Scaling rules
| | Z | x86 |
| --- | --- | --- |
| Vertical | Linux on Z: more IFLs. z/OS: a declared sequence of (CP, zIIP) pairs, at least one CP, up to the CPC's configuration. The CP capacity setting is recorded and fixed for the series | More cores per node, each configuration a cpuset that holds the whole system under test; same node type throughout a scale-out series |
| Horizontal | More images in a Parallel Sysplex (later phase) | More application nodes (the scale-out tier); more replicas for reads, if replica reads are on |
| What does not grow | One coupling facility configuration per series | One database primary per series (single-writer). Sharding changes the consistency model and is out of scope |
| Benchmark rule | Equal core counts in the named scoring configurations of the registered hypotheses. Equal service: the configuration grows through a declared set until the target is sustained, with dedicated or entitled engines (proposed SPEC 8) | Equal service on one server: the core count grows through a declared set (proposed SPEC 8). In the scale-out tier only the application node count N changes within a series; the database, the counter and session store and the load balancer are fixed and declared before the series starts (proposed SPEC 9) |

Two scale-out variants are defined (proposed SPEC section 9):
- **S1 (default):** only application nodes grow.
- **S2:** application nodes and read replicas grow together, with replica reads on.

## 9. Failure domains
The benchmark measures steady state. It does not inject failures during measured runs (out of scope for this
version); the durability acceptance test (proposed SPEC 4.15) stops the system under test abruptly before the counted
runs, and is not a measurement. A recovery-time series is a candidate for a later spec version. The availability
design still changes the steady-state cost, so it is disclosed.

| Failure | Z: single LPAR | Z: Parallel Sysplex | x86 cluster |
| --- | --- | --- | --- |
| One transaction | Backout of its uncommitted recoverable changes [S5] | Same | Database rollback; for the file-based variant, the data adapter's rollback under the "atomic unit of work" label |
| Transaction manager process | CICS region restarts; in-flight units of work are backed out [S5] | Work continues in other regions or images | One application node; the load balancer removes it; its in-flight requests fail |
| Counter and session store | Depends on the store chosen and its durability class (disclosed) | Shared structure; depends on duplexing | Depends on its durability class; counters in a volatile store are lost |
| Data manager | Db2: restart from its log, backing out in-flight units of work (source needed). VSAM has no log of its own: recoverable files are backed out by CICS from its system log at restart [S5]; RECOVERY(NONE) files are not recovered | Other members continue; the failed member's modify locks become retained, and the member can be restarted in light mode to recover them quickly [S27] | Under A0, restart of the primary from its log; under A1, failover to the standby, with the data loss window of its replication mode |
| Coupling facility | n/a, or RLS stops (if a CF is used) | A single CF is a single point of failure for a data sharing group [S13b] | n/a |
| Operating system image | The whole deployment | One image of several | One node |
| Machine (CPC or server) | The whole deployment | All images on that CPC | One application node; under A0, the database primary's machine is the whole deployment's write path |
| Network | The LPAR's adapters | Adapters and the coupling links | Switches and links between every tier |

For each deployment the FDR states the failure domains, the design recovery point and the design recovery time.
The recovery point and the recovery time are design targets, not measurements (proposed FDR change F7).

## 10. Mapping to the FDR
| Element | FDR field ([FDR-TEMPLATE.md](FDR-TEMPLATE.md), including the changes proposed with this page) | Gap |
| --- | --- | --- |
| Registered verdicts, first | 1 Summary, "Registered verdicts, first" | Covered |
| LPAR, capacity setting, CP/zIIP/IFL counts, weight, capping, other partitions' load | 2 Hardware (required fields for shared processors); 2 Other load (required); 2 Virtualisation | Covered |
| WLM service definition | 3 Software ("the WLM policy excerpt") | Region or transaction management for CICS, and the classification rules, are not named: F5 |
| Linux nice and cgroups | 3 Software ("cgroups weights") | `cpu.max`, `cpuset.cpus`, IRQ affinity, nice values, scheduler policies and the autogroup setting are not named: F5 |
| CICS regions, file definitions (recovery, strings) | 3 Software (non-default settings) | Covered, if listed in full |
| Coupling facility and structures | none | F1 |
| Cluster topology: node list, roles, node types | 2 lists one machine | F1 |
| Network fabric, round trips per operation | 2 "network, if used" | F2 |
| Load driver hosts and their validation | none | F3 |
| Database engine, isolation, durability, replication | 3 Software, the database product, version and durability settings | Isolation, write-cache protection and replication are not named: F4 |
| Counter and session store | none | F4 |
| Request profile and bookkeeping steps | 4 Workload | F6 |
| "Nothing refused" check, and the invariant check | 6 Failed runs | F6 |
| Rate sweep and knee | 6 Results | F8 |
| Scale-out series | none | F8 |
| Failure domains and recovery design | none | F7 |
| CPU per transaction across all tiers | 1 Summary "CPU per transaction" | Which nodes are included, and the accounting basis, are not stated: F9 |
| Energy for several nodes | 6 energy block (docs/methods/energy.md) | Sum over nodes, with switches excluded: F10 |
| Equal service: the service target and what each side needs | 1 Summary, the results by tier | Full inventory: F11 |
| Preliminary or not (proposed SPEC 10) | 1 Summary, "Preliminary or not" | Covered |
| Run registry entries and tuning log | 1 Summary (run registry entries); none for the tuning log | F12 |
| SAPs, IFPs, DPU, channels, adapters and offloads, on both sides | 2 Hardware, "processors and adapters not counted as cores" | Covered |
| Live-database form (same product, or whole stack) | 1 Summary, the data variant and its form | Database details: F4 |
| Licence metric of each product | 8 Pricing appendix, "the licence metric of each software product" | Covered |

### Proposed FDR changes
- **F1 Topology (new subsection of 2).** A table of every node or partition: role (load balancer, application,
  counter and session store, database primary, replica, batch, coupling facility, driver), machine type, cores or
  engines, memory, storage, virtualisation, and the host or CPC it runs on. A diagram is optional.
- **F2 Network (extends 2).** Link speeds, switch models, the measured round-trip time between each pair of tiers
  (idle, before the run), the number of round trips per operation type, and bytes per transaction across tiers.
- **F3 Load generation (new section, after 4).** Driver hosts and their hardware, arrival process and seeds, number
  of connections, the timeout T, the self-test and the cross-check (proposed SPEC 7.3), and the network path from
  driver to the system under test.
- **F4 Data and counter tier (extends 3).** The data variant and its label and, for a live database, its form (A,
  same product; B, whole stack); the database product, version and fix level, isolation level and locking clauses,
  durability settings and the write-cache protection of the log device, replication mode and number of synchronous
  standbys, connection pool sizes; and the counter and session store, its durability class, persistence and
  eviction policy.
- **F5 Priority and confinement configuration (extends 3).** For Linux: nice values, scheduler policies, the kernel
  version (and so the fair scheduler), the autogroup setting, the cgroup tree with `cpu.weight`, `cpu.max` and
  `cpuset.cpus` per group, and IRQ affinity. For z/OS: service classes, goals, importance, classification rules,
  any resource group, and region or transaction management for CICS.
- **F6 Request profile and checks (extends 4 and 6).** Base or bookkeeping profile; which bookkeeping steps were on;
  where sessions, counters, access records and audit records were kept and how durable each was; the count of
  "would throttle" events; business outcomes by type, including duplicate-key rejections; confirmation that no
  request was refused; and the result of the invariant check.
- **F7 Availability design (new subsection of 2).** The availability design (A0 or A1, proposed SPEC 9.2), failure
  domains, the design recovery point and recovery time, coupling facility count and duplexing, and replica count and
  mode, with the cores of availability-only tiers counted separately.
- **F8 Rate sweep and scale-out (extends 6).** The sweep table (offered rate, achieved rate, p50/p90/p99/p99.9,
  maximum, timeouts, errors, business outcomes and schedule lag), the per-repeat knees, the median knee with its
  interval and coverage, any non-monotone passes, the past-saturation points with their queue growth, and for the
  scale-out tier one row per (rate, N) point with per-tier utilisation and K_N.
- **F9 Cost per transaction (extends 1 and 6).** CPU time per transaction by engine type, summed over every node or
  partition of the system under test on the basis of CPU-ACCOUNTING.md, with the capture ratio on z/OS, the
  confinement check on Linux, the cost of monitoring, and a per-tier split. Driver hosts are excluded and their CPU
  is reported separately.
- **F10 Energy for several nodes (extends the energy block).** Per-node figures, their sum, and what the sum leaves
  out (switches, load balancer appliances, storage arrays).
- **F11 Resources at equal service (extends 1 and 2).** For each service target: how the target and L were derived
  from both sides' calibrations; the declared set of configurations on each side, the sustaining configuration,
  and the result of the next smaller one; counted cores or engines by type over every tier; CPU time per
  transaction by engine type; processors and adapters listed but not counted (section 1.2); memory and last-level
  cache; accelerators in use; the hardware generation of each side; power and cost, or the reason they are not
  stated; utilisation per tier; and the per-core, per-watt and per-dollar normalisations, each labelled.
- **F12 Registry, tuning log and machine-readable summary (new section).** The run registry entries of the series,
  its tuning log (proposed SPEC 4.13 and 4.14), and a machine-readable summary of the FDR's fields (proposed: YAML,
  with a schema published with the kit), so that FDRs can be checked for missing fields and compared automatically.

## 11. Open questions
- May a platform security check (for example, CICS resource security through RACF) count as step 3, or is it extra
  work on top of an application-level check?
- Which counter and session store on z/OS is the fair counterpart of an x86 cache tier?
- Which SQL interface does the COBOL track use on Linux? (Database versions are the newest generally available on
  every platform of the comparison when the series' run plan is committed, proposed SPEC 1.8.4.)
- After the first online set, should CardDemo's own optional Db2 component (Transaction Type Management) join a
  later set, or does the Db2 variant cover only the base application's data [S22]?
- Do the licence terms of the licensed programs in a run, on either side, place conditions on publishing benchmark
  results? Database licences may restrict this (docs/prior-art.md, SB02; a search-result snippet, not verified), and
  IBM's program licence terms, and those of any commercial Linux distribution, hypervisor or Java runtime in the
  run, may contain benchmarking clauses with conditions on disclosure (to be checked). Until they are checked, no
  result from a licensed program is published (PLAYBOOK.md, section 1). PostgreSQL's licence permits
  use "for any purpose" [S36].
- The latency bound used to define a sustained rate (proposed SPEC 7.2): one value for all operations, or one per
  operation?
- Should a Parallel Sysplex series be run on one CPC, several CPCs, or both?
- Should a multi-writer x86 variant be defined for the scale-out tier, and with which database?

## 12. Sources
Fetched 2026-11-26 unless marked otherwise. "Search result only" means the page was found by search but not read.
Several IBM sources are for older releases (z/OS 2.x, CICS TS 5.x); each fact taken from them is to be checked against
the current release before it is relied on (verify).
- [S1] Linux kernel, Control Group v2 (`cpu.weight`, `cpu.max`, `cpu.weight.nice`, `cpu.pressure`, and the cpuset
  controller's `cpuset.cpus`): https://docs.kernel.org/admin-guide/cgroup-v2.html
- [S2] Linux kernel, CFS scheduler design (SCHED_BATCH, SCHED_IDLE; "CFS is making room for EEVDF"):
  https://docs.kernel.org/scheduler/sched-design-CFS.html
- [S3] Linux kernel, scheduler nice design: https://docs.kernel.org/scheduler/sched-nice-design.html
- [S4] IBM CICS TS 5.5, Units of work:
  https://www.ibm.com/support/knowledgecenter/SSGMCP_5.5.0/administering/recovery/dfht263.html
- [S5] IBM CICS TS 5.4, Unit of work recovery (backout of uncommitted changes to recoverable resources):
  https://www.ibm.com/support/knowledgecenter/SSGMCP_5.4.0/troubleshooting/dfht229.html
- [S6] IBM CICS TS 5.4, z/OS Workload Manager: service classes and report classes:
  https://www.ibm.com/docs/en/SSGMCP_5.4.0/fundamentals/workload/workload_lp_classes.html
- [S7] IBM CICS TS 6.2, How it works: z/OS Workload Manager:
  https://www.ibm.com/docs/SSGMCP_6.2.0/fundamentals/wlm/zos-wlm.html
- [S8] wrk2 README (constant-rate load and coordinated omission): https://github.com/giltene/wrk2
- [S9] PostgreSQL documentation, synchronous replication:
  https://www.postgresql.org/docs/current/warm-standby.html#SYNCHRONOUS-REPLICATION
- [S10] N. Gunther, Universal Scalability Law: http://www.perfdynamics.com/Manifesto/USLscalability.html
- [S11] IBM CICS TS 5.5, Security checking: https://ibm.com/support/knowledgecenter/SSGMCP_5.5.0/system-programming/intro/dfha80v.html
- [S12] IBM z/OS 2.3 RACF, Record type 80:
  https://www.ibm.com/support/knowledgecenter/en/SSLTBW_2.3.0/com.ibm.zos.v2r3.icha300/type80.htm
- [S13] IBM Db2 12 for z/OS, Coupling facility structures:
  https://www.ibm.com/docs/en/SSEPEK_12.0.0/inst/src/tpc/db2z_cfstructures.html
- [S13b] IBM Db2 12 for z/OS, Planning for coupling facility availability for data sharing ("Multiple coupling
  facilities are always required for high availability because a single coupling facility is single point of
  failure for the data sharing group"):
  https://www.ibm.com/docs/SSEPEK_12.0.0/inst/src/tpc/db2z_plancfavailability.html; and How Db2 protects data
  consistency in data sharing ("The coupling facility invalidates any image of the page in the buffer pools
  associated with each member"):
  https://www.ibm.com/docs/SSEPEK_12.0.0/dshare/src/tpc/db2z_protectdataconsistency.html
- [S14] IBM z/OS 3.1 Communications Server, Sysplex distributor (distribution methods; WLM system weights are the
  default): https://www.ibm.com/docs/SSLTBW_3.1.0/com.ibm.zos.v3r1.halz002/sys_distrib.htm
- [S15] IBM z/OS 2.2 Communications Server, HiperSockets concepts and connectivity (search result only):
  https://www.ibm.com/support/knowledgecenter/SSLTBW_2.2.0/com.ibm.zos.v2r2.halz002/ip_hipersockets_concepts_connectivity.htm
- [S16] IBM CICS TS 6.1, How CICSPlex SM workload management chooses where to route work:
  https://www.ibm.com/docs/SSGMCP_6.1.0/fundamentals/wlm/wlm-algorithms.html
- [S17] IBM Redbooks, *IBM z16 Technical Introduction*, SG24-8950-01 (April 2023): section 2.3.2 (on-core
  cryptographic hardware for each core; the AI accelerator on each chip, shared among its cores; SAPs do not change
  the capacity setting or MSU rating); chapter 1 ("one dedicated compression coprocessor per PU"); chapter 5, the
  list of on-core and on-chip accelerators ("Dedicated coprocessors and accelerators for each PU core": CPACF, and
  the IBM Integrated Accelerator for IBM Z Sort, "This on-core accelerator for sort uses the sort instruction
  (SORTL)"; DEFLATE compression "on a processor chip level"; the AI accelerator "shared by all cores on the chip"); section 5.2.1 (SAPs "that run I/O operations" are always present; logical
  processors dispatched on shared or dedicated physical processors); and section 5.3.3, HiperDispatch (logical
  processors "needed for the priority (weight) of the LPAR"):
  https://www.redbooks.ibm.com/redbooks/pdfs/sg248950.pdf
- [S19] IBM, Processor Value Units (PVUs) ("each processor core on a chip"; "Each Integrated Facility for Linux (IFL)
  or Central Processor (CP) engine is equivalent to 1 core"):
  https://www.ibm.com/software/passportadvantage/pvu_licensing_for_customers.html
- [S20] IBM z/OS 3.1, What is Tailored Fit Pricing for IBM Z? (offered "alternatively to the rolling 4-hour (R4HA)
  average"; "Compute is measured on the basis of MSUs consumed"):
  https://www.ibm.com/docs/en/zos/3.1.0?topic=z-what-is-tailored-fit-pricing
- [S21] Linux kernel, Segmentation Offloads: https://docs.kernel.org/networking/segmentation-offloads.html
- [S22] AWS CardDemo README, "Core Technologies", "Optional Technologies" and "Optional Features" (Db2, IMS DB and MQ
  as optional technologies; the Transaction Type Management feature with Db2; the authorization feature with IMS,
  Db2 and MQ; the account extraction feature with MQ and VSAM; the additional JCL utilities, including "DB2 and IMS
  DB load/unload operations"):
  https://github.com/aws-samples/aws-mainframe-modernization-carddemo
- [S23] IBM Redbooks, *IBM z17 Technical Introduction*, SG24-8580-01 (Second Edition, July 2026), read as extracted
  text: section 1.2 (the IBM Spyre Accelerator, "Up to 48 adapters per system" on ME1; the on-chip DPU); section
  2.3.1 (one DPU I/O engine per PU chip); section 2.3.2 (PU characterisation: IFLs for Linux "natively or on top of
  a hypervisor like IBM z/VM or Kernel-based Virtual Machine (KVM)"; "CF partitions can run on shared or dedicated
  ICF processors, or on shared CPs. On IBM z17, the use of dedicated CPs for CF partitions was eliminated"; "Before
  acquiring zIIPs, at least one CP must be purchased"; "The addition of ICFs, IFLs, and zIIPs does not change the
  system capacity setting or its millions of service units (MSU) rating"; the on-chip AI accelerator, DPU, CPACF,
  Z Sort (SORTL) and zEDC accelerators; the maximum number of zIIPs is "one fewer than the system's maximum PU
  configuration"; each CPC drawer has "a 2.8 GB shared-victim virtual L4"); section 3.3.2 (the sort accelerator "to
  be used by DFSORT"; the AI accelerator, "Artificial Intelligence Unit (AIU)", "implemented on each PU chip and
  shared among all cores", with the NNPA instruction); section 5.1.3 ("Each processor core includes integrated
  accelerators", listing the AI accelerator, with capacity "shared across cores", CPACF, zSort and data compression);
  section 4.4
  (network adapters that offload selected TCP/IP processing); section 5.2.1 ("All physical processors are
  virtualized as logical processors"; SAPs "for I/O operations" and IFPs "for firmware and PCIe functions"; "PR/SM
  dynamically dispatches logical processors across physical processors, enabling shared or dedicated
  configurations"); and section 5.4.3, HiperDispatch: https://www.redbooks.ibm.com/redbooks/pdfs/sg248580.pdf
- [S27] IBM Db2 12 for z/OS, Active and retained locks for Db2 data sharing members ("active locks used to control
  updates to data (modify locks) become retained in the event of a failure"; "You can restart a failed member in
  light mode to quickly recover the locks held by that member"):
  https://www.ibm.com/docs/SSEPEK_12.0.0/dshare/src/tpc/db2z_activeretainedlocks.html
- [S34] IBM z/OS 2.1 Security Server RACF, Guarding against data corruption resulting from incorrect database
  sharing (in data sharing mode "a coupling facility is used to cache recently accessed RACF data"; "all systems
  accessing the database must be in the same sysplex"):
  https://www.ibm.com/support/knowledgecenter/SSLTBW_2.1.0/com.ibm.zos.v2r1.icha200/dbshi.htm
- [S35] PostgreSQL documentation, Write Ahead Log settings (`fsync`, `synchronous_commit`, `full_page_writes`):
  https://www.postgresql.org/docs/current/runtime-config-wal.html
- [S36] PostgreSQL licence: https://www.postgresql.org/about/licence/
- [S37] Linux kernel, EEVDF scheduler ("The Linux kernel began transitioning to EEVDF in version 6.6"):
  https://docs.kernel.org/scheduler/sched-eevdf.html
- [S41] GnuCOBOL 3.2, DEPENDENCIES file (the GMP or MPIR library; the indexed-file handlers):
  https://sources.debian.org/src/gnucobol3/3.2-4/DEPENDENCIES
- [S42] IBM Enterprise COBOL for z/OS 6.4, ARCH (the vector packed-decimal facility at ARCH(12) and later):
  https://www.ibm.com/docs/en/cobol-zos/6.4.0?topic=options-arch
- [S43] AWS CardDemo source at commit 59cc6c2 (2025-10-16): `app/cbl/COTRN02C.cbl` (new ID by `MOVE HIGH-VALUES TO
  TRAN-ID`, `STARTBR`, `READPREV`, `ADD 1`; `DFHRESP(DUPKEY)` / `DFHRESP(DUPREC)` reported as "Tran ID already
  exist..."); `app/cbl/COBIL00C.cbl` (the same ID logic; `MOVE ACCT-CURR-BAL TO TRAN-AMT`; `PERFORM
  WRITE-TRANSACT-FILE` followed by `COMPUTE ACCT-CURR-BAL = ACCT-CURR-BAL - TRAN-AMT` and `PERFORM
  UPDATE-ACCTDAT-FILE`, with the duplicate-key branch of the write sending a message and returning to the paragraph
  that performed the write, so that control reaches the COMPUTE that follows):
  https://github.com/aws-samples/aws-mainframe-modernization-carddemo
- [S44] IBM Support, Introduction to Logical Partitions (modified 2024-09-24): a shared partition's processor time
  is "based on the partition's logical processor weight, relative to the rest of the active partitions"; "Excess
  processor resource from one partition can be used by another partition"; "This can be limited via per partition
  capping": https://www.ibm.com/support/pages/introduction-logical-partitions
- [S45] IBM CICS TS 6.x, Making applications threadsafe ("By defining a program to CICS as threadsafe, you are
  specifying only that the application logic is threadsafe"; "you must use appropriate serialization techniques when
  accessing shared resources"), and Threadsafe programs (a QUASIRENT program "always runs under the CICS
  quasi-reentrant (QR TCB)"): https://www.ibm.com/docs/SSJL4D_6.x/fundamentals/threadsafe/threadsafe_making.html,
  https://www.ibm.com/docs/SSJL4D_6.x/fundamentals/threadsafe/threadsafe_program.html
- [S46] Linux kernel, `init/Kconfig` and `arch/s390/Kconfig`, CPU time accounting options (CPU-ACCOUNTING.md):
  https://github.com/torvalds/linux/blob/master/init/Kconfig
- [S47] GnuCOBOL 3.x, `config/default.conf` ("binary-byteorder: big-endian"; read on the 3.x branch):
  https://sourceforge.net/p/gnucobol/code/HEAD/tree/branches/gnucobol-3.x/config/default.conf
- [S48]-[S51] Zenodo, OSF, GitHub rulesets and git-tag documentation, on registration (REGISTRY.md).
- [S52] GitHub Docs, Reporting abuse or spam ("Report content", "Report abuse", "Report to repository admins"):
  https://docs.github.com/en/communities/maintaining-your-safety-on-github/reporting-abuse-or-spam
- [S53] Eclipse Adoptium, Supported platforms (lists "Linux (s390x)"): https://adoptium.net/supported-platforms/
- [S54] Linux kernel, IP sysctl (`ip_local_port_range`: "The default values are 32768 and 60999 respectively"):
  https://docs.kernel.org/networking/ip-sysctl.html
- [S55] IBM, "New Telum II Processor and IBM Spyre Accelerator" (the enhancements "enable each AI accelerator to
  accept work from any core in the same drawer"): https://www.ibm.com/blog/announcement/telum-ii
- [S56] Linux man-pages, chcpu(8) ("Set the CPU dispatching mode (polarization)"; horizontal and vertical):
  https://man7.org/linux/man-pages/man8/chcpu.8.html
- [S41], [S42], [S47] and [S53] support SPEC 1.0, [S54] SPEC 7.1, and [S52] GOVERNANCE.md, "Contact"; those pages
  cite them by number or by URL.

Further sources used by [PLAYBOOK.md](PLAYBOOK.md), [CPU-ACCOUNTING.md](CPU-ACCOUNTING.md) and
[REGISTRY.md](REGISTRY.md) are listed there, with the same numbering ([S25], [S26], [S28]-[S33], [S38]-[S40],
[S46], [S48]-[S51]).
