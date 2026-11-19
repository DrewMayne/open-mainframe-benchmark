# Open Mainframe Benchmark: specification, draft v0.3.0 (2026-11-19)

Status: draft for comment. Items marked **(proposed)** are not yet decided. Version numbers follow semantic versioning:
results always name the spec version and track they ran against.

Changes in v0.1.1: two implementation tracks (Java first, then COBOL) under one spec (1.0, 1.5).
Changes in v0.2.0: analytics track, TPC-H-derived and compliance-ready (1.6); rules for using TPC materials (6).
Changes in v0.3.1: TPC labelling aligned with TPC Policies v6.20 (8.1.4, 8.1.5); written permission requested before publishing derived results (6).
Changes in v0.3.0: optional TPC-derived modules: TPC-E, TPC-DI, TPCx-V, plus the TPC-Pricing and TPC-Energy methods (1.7).

## 1. Workload

### 1.0 Two tracks, one specification
The benchmark has two implementation tracks. They share the same transactions, data, batch suite, accelerated
extension, driver, metrics, run rules and FDR. Results always name their track.
- **Java track (built first).** The transaction and batch logic is written in Java and runs on the same JVM family
  (IBM Semeru / OpenJ9) on x86 Linux, Linux on Z and z/OS. Because the code and the JIT are the same everywhere, it
  answers the hardware question most cleanly. On z/OS it also shows the effect of zIIP engines, which run Java work.
  Its accelerated steps use the standard Java APIs: `java.util.zip` (DFLTCC on Z through Semeru), JCE with OpenJCEPlus
  (CPACF on Z), and ONNX Runtime or the zDNN-backed path for scoring.
- **COBOL track.** The CardDemo-based application in COBOL (1.1). It answers the migration and rehosting question,
  because COBOL/CICS business applications are what those claims are about. On z/OS it uses IBM Enterprise COBOL
  under CICS. On Linux it uses GnuCOBOL. **Caveat, stated with every COBOL-track result:** GnuCOBOL compiles via C and
  is not the compiler production mainframes use, so Linux-on-Z COBOL figures don't represent Enterprise COBOL on z/OS.
- **Track equivalence:** both tracks must produce identical business results (account balances, posted transactions,
  statements) from the same seed. That is checked by a shared golden-output test.

### 1.1 Base application (proposed)
- **Core (COBOL track; the Java track reimplements the same transactions): [AWS CardDemo](https://github.com/aws-samples/aws-mainframe-modernization-carddemo)** (Apache 2.0). It is a COBOL credit-card management application: CICS online transactions over VSAM (KSDS with alternate indexes), plus a batch suite.
- **Why CardDemo:** it already exists, it is open, and it is used to test mainframe migration tooling, including performance. Results on it can be compared directly with migration claims.
- **On z/OS:** it runs as-is (CICS, VSAM, JCL, DFSORT).
- **On Linux (s390x and x86):** CICS and VSAM are replaced by a thin portable layer:
  - indexed files through GnuCOBOL file I/O;
  - a pool of long-lived COBOL workers driven by the load generator, in place of CICS;
  - a driver that bypasses the 3270 screens and calls the business logic with the same inputs.
- The port must not change business logic. Differences are listed in a PORTING.md with the reason for each.

### 1.2 Online transactions
- The CardDemo online set (account view and update, card list and update, transaction list, view and add, bill payment), weighted by a **transaction mix table** (to be defined in v0.2, with the weights published).
- Think time is zero in closed-loop mode. In open-loop mode, an arrival rate is offered (see section 4).

### 1.3 Batch suite
- **B1 posting:** transaction posting (CardDemo POSTTRAN), interest calculation (INTCALC) and statement creation (CREASTMT).
- **B2 extract → sort → merge** of the transaction history, at a stated record count and size. Sorts use the platform's normal utility: DFSORT on z/OS, GNU sort on Linux. Both are reported.
- **B3 archive:** compress closed statements and transactions.

### 1.4 Accelerated extension (proposed, switchable)
These are optional steps added to the online path, each a single library call so that it can be switched between
"plain" and "accelerated" on every platform:
- **Sign** each bill payment, using ECDSA P-256 through OpenSSL. It uses CPACF/KDSA on Z (z15+), and software or available instructions on x86.
- **Compress** each statement or document attachment with zlib. It uses DFLTCC on Z (z15+), and software, zlib-ng or QAT on x86.
- **Score** each transaction with a small fraud model, at batch size 1. It uses zDNN or the IBM Deep Learning Compiler on Z (z16+ AIU), and ONNX Runtime CPU (AVX2, AVX-512 or AMX) on x86.

Results are always reported with and without the extension.

### 1.5 Data
- **Storage per track (proposed):** the COBOL track uses GnuCOBOL indexed files on Linux and VSAM on z/OS. The Java track uses a portable keyed store, to be chosen in v0.2. Both load from the same generated record files.
- A deterministic generator with a scale factor S (accounts = 10,000 × S, transactions per account stated).
- The same seed gives byte-identical data files on every platform. Golden checks compare decompressed bytes, because hardware and software deflate output differ.

### 1.6 Analytics track: derived from TPC-H, compliance-ready (proposed)
The reporting and analytics side of mainframe work: large scans, joins, sorts and aggregations over a standard data set.
- **Data and queries:** generated with TPC's own `dbgen` and `qgen` from the TPC-H kit (spec revision 3.0.1 at the time
  of writing; use the latest revision on tpc.org and name it in every result). Each user downloads the kit from tpc.org
  and accepts the TPC EULA. It is never included in this repository (see section 6).
- **Databases:** PostgreSQL on x86 and on Linux on Z, and Db2 for z/OS in phase 2. The query text follows the kit, with
  only the dialect changes the spec permits, each one listed in the FDR.
- **Compliance-ready:** the harness implements every rule of the TPC-H specification that can be implemented on the
  platform:
  - permitted scale factors only (1, 10, 30, 100, 300 and up; never others);
  - exact dbgen population, and query output validated against the kit's reference answers;
  - the power test and the throughput test, with the required number of query streams plus the refresh stream (RF1/RF2);
  - the ACID tests, including durability by power loss on platforms we fully control;
  - the FDR and executive summary in TPC's format, and pricing under the TPC Pricing Specification where it can be done
    with public list prices.
- **How results are labelled:** until an independent TPC-certified audit is passed and the result is accepted by the
  TPC, every result is published as **"Derived from TPC-H"** with the TPC's required disclaimer (section 6), as academic
  research with no marketing claims (TPC EULA clause 4c). TPC metric names (e.g. QphH@Size) are never used for our
  numbers. Rules that could not be met on a platform are listed in its FDR.
- **Also used as batch:**
  - the queries can run as the batch load underneath the online track, with reporting against the live database;
  - dbgen's LINEITEM data can be the standard input for the B2 extract/sort/merge job.
- **End goal:** an audited, official TPC-H result on at least one configuration, if a sponsor funds the audit and
  pricing.

### 1.7 Optional TPC-derived modules (proposed)
Each module answers a question the core tracks don't. Each is optional, run only when its question matters, and
follows section 6. Revisions are as listed on tpc.org on 2026-11-19; results name the revision used.
- **TPC-E (v1.14.0, EGen kit). A standard OLTP yardstick.** The brokerage OLTP workload, run on PostgreSQL / Db2 next
  to our CardDemo track. It shows whether our online-track ratios hold on a recognised OLTP mix, and it is the nearest
  standard analogue to zRef. It is a separate module, and CardDemo is never derived from it.
- **TPC-DI (v1.1.0, DIGen kit; listed by the TPC as obsolete, so no new official results are possible). Batch ETL.**
  It loads a warehouse from multiple source formats, including fixed-width files, with historical and incremental
  loads. It is the nightly batch job in its most mainframe-like form, measured as elapsed time and CPU for each load
  phase.
- **TPCx-V (v2.1.10, kit). Consolidation.** Many database VMs with shifting load on one host. It tests the
  consolidation claims (how many VMs per LinuxONE against per x86 host, under the same service levels) on KVM or z/VM
  on Z and KVM on x86.
- **TPC-Pricing (v2.9.0) and TPC-Energy (v1.5.0). Methods, not workloads.** Pricing defines how the optional pricing
  appendix and the open cost model (M7) are built, from public list prices with stated assumptions. Energy defines how
  power is measured where it can be (on x86 we control). On shared Z it can't be measured, and the FDR says so.

## 2. Platforms
- **Phase 1:**
  - x86 PC on bare-metal Linux;
  - the same PC in a VM;
  - a LinuxONE / Linux on Z guest, **only where terms permit benchmarking and publishing**.
- **Phase 2:**
  - an x86 server with QAT and AMX (cloud bare metal);
  - z/OS with CICS, VSAM, DFSORT and WLM;
  - dedicated or characterised LinuxONE capacity.
- **Equal counts:** runs use equal counts of cores or IFLs (2 and 4 to start). SMT state is recorded, along with whether a "vCPU" is a core or a thread.

## 3. Metrics
- **Throughput:** transactions per second, per core.
- **Cost per transaction:** CPU time per transaction (user + sys), from OS accounting. Hardware counters may be unavailable in Z guests, so wall-clock and OS CPU time are the common basis.
- **Latency:** p50, p90, p99 and p99.9 (HdrHistogram).
- **Batch:** elapsed time and CPU time per job, alone and alongside online work.
- **Interference:** online capacity and p99 with batch present, divided by the same without batch.
- **Steal time** and other-load indicators on shared platforms.
- **Normalisations:** per core at the stated clock, and per watt where measurable (RAPL on x86; Z guests generally cannot meter, which is stated). Per dollar is optional, using published list prices with assumptions stated.
- **MIPS:** never derived from Linux runs. A calibration of MIPS to x86 cores per workload class is planned for phase 2, against z/OS and the measured configuration's published capacity rating.

## 4. Run rules
1. **Repeats:** at least 5 runs per point, in randomised order, with warm-up excluded. Report the mean and a 95% confidence interval, plus the median and spread.
2. **Load modes:** closed loop for capacity. Open loop at 50, 70, 90, 100, 110 and 120% of measured capacity for the degradation curve.
3. **Batch alongside online:** the measured window must span whole batch cycles, or be synchronised to the cycle start. A window that samples only one phase of a batch job biases the result; we found this in our own pilot.
4. **Pinning:** cores pinned on both sides, and frequency governor and SMT recorded.
5. **A manifest per run:** CPU model and machine type, core and thread counts, memory, OS and kernel, compiler and library versions, configuration switches, and the git commit of the kit.
6. **Raw output:** every raw file is kept and published with the results.
7. **Pre-registration:** a hypothesis file entry exists, committed, before the first run that tests it.
8. **Full Disclosure Report:** every published result comes with an FDR following [docs/FDR-TEMPLATE.md](docs/FDR-TEMPLATE.md). A result without one is not a result of this benchmark.

## 5. Out of scope (v0.1)
- Audited or official benchmark status (TPC/SPEC). Results are research results. This kit is not a TPC benchmark, and results must not be presented as TPC results or described with TPC benchmark names. A separately sponsored, audited run of an actual TPC benchmark is listed as a stretch goal in ROADMAP.md.
- DB2, IMS and MQ variants of CardDemo (later, possibly).

## 6. Using TPC materials
TPC benchmark kits and specifications are licensed by the Transaction Processing Performance Council under their own
EULAs. To stay within them:
1. **Download, never vendor.** TPC kits are never committed to this repository. Scripts fetch them from tpc.org, or
   point the user there to accept the EULA.
2. **Label every derived work and result** as the TPC Policies require (v6.20, clauses 8.1.4 and 8.1.5):
   - the prefix **"Derived from"** before every use of the benchmark name (e.g. "Derived from TPC-H Query 5");
   - this disclaimer, verbatim, with the benchmark name filled in: *"This workload is derived from the TPC-H Benchmark
     and is not comparable to published TPC-H Benchmark results, as this implementation does not comply with all
     requirements of the TPC-H Benchmark."*
   - A derived work judged too similar to the standard falls under the TPC's fair-use rules. So before results that
     use a TPC benchmark name are published, **the TPC's written permission is requested** (Policies 8.1.4), and the
     publishing scripts refuse to run until it is recorded.
3. **No TPC names or metrics on our own workloads.** The online track (CardDemo-based) is not TPC-C- or TPC-E-derived
   and is never described as such. TPC metric names (tpmC, tpsE, QphH@Size, and so on) are used only for official
   results.
4. **Academic framing only:** no marketing claims based on TPC-derived results.

TPC, TPC-H, TPC-C, TPC-E, TPC-DS, TPC-DI, TPCx-AI, TPCx-V, QphH and related names are trademarks of the Transaction
Processing Performance Council.
