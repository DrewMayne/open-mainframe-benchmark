# Open Mainframe Benchmark: specification, draft v0.3.2 (2026-11-26)

Status: draft for comment. Items marked **(proposed)** are not yet decided. Version numbers follow semantic versioning:
results always name the spec version and track they ran against.

Changes in v0.1.1: two implementation tracks (Java first, then COBOL) under one spec (1.0, 1.5).
Changes in v0.2.0: an optional analytics module derived from industry-standard benchmark material (1.6), and rules for using third-party benchmark materials (6).
Changes in v0.3.1: labelling of derived work aligned with the benchmark owner's published policies; written permission requested before publishing derived results (6).
Changes in v0.3.0: further optional modules derived from benchmark material, plus pricing and energy methods (1.7).
Changes in v0.3.2 (2026-11-26), a withdrawal: the optional modules derived from benchmark material, and the rules specific to them, are withdrawn until the benchmark owners' permission is in place (1.6, 1.7, 6). HYPOTHESES.md is unchanged.

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
- **On z/OS:** it installs as shipped (CICS, VSAM, JCL, DFSORT). Running its batch and online work together needs configuration changes, or changes to the batch programs if their file access is routed through CICS (see 4.3 and docs/PORTING-zOS.md).
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

### 1.6 Optional modules (in preparation)
Optional modules derived from industry-standard benchmark material are in preparation. They will be published
only once the benchmark owners' permission is in place; until then, none is described here.

### 1.7 Pricing and energy methods (proposed)
Methods, not workloads:
- **Pricing:** the optional pricing appendix and the open cost model (M7) are built from public list prices, with
  every assumption stated.
- **Energy:** power is measured where it can be (RAPL on x86 we control; docs/methods/energy.md).
  On IBM z17 ME1 / LinuxONE Emperor 5 (machine type 9175), per-partition power is reported to the operating system
  (Linux `zpwr`, z/OS SMF 70.1 via APAR OA66018, the z/VM monitor on z/VM 7.3 with APAR VM66823 and on z/VM 7.4),
  but IBM describes it as apportioned rather than measured. For machine type 9176 (z17 ME2 and MER, LinuxONE
  Rockhopper 5) this is not verified. It is recorded in an optional FDR field labelled "vendor estimate; not
  measured" and kept out of per-watt comparisons. Older Z generations: no partition figure is visible to the
  operating system, and the FDR says so; CPC power is shown on the HMC to the machine's owner
  (docs/methods/energy.md).

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
- **MIPS:** never derived from Linux runs. A calibration of MIPS to x86 cores per workload class is planned for phase 2, against z/OS and IBM's capacity data for the measured processor model. IBM's published capacity ratings are per processor model (the LSPR multi-image table, which IBM uses as the basis for MIPS and MSUs); for a specific LPAR configuration, IBM's LSPR FAQ points to its zPCR tool (LPAR Configuration Capacity Planning).

## 4. Run rules
1. **Repeats:** at least 5 runs per point, in randomised order, with warm-up excluded. Report the mean and a 95% confidence interval, plus the median and spread.
2. **Load modes:** closed loop for capacity. Open loop at 50, 70, 90, 100, 110 and 120% of measured capacity for the degradation curve.
3. **Batch alongside online:** the measured window must span whole batch cycles, or be synchronised to the cycle start. A window that samples only one phase of a batch job biases the result; we found this in our own pilot.
   On z/OS, CardDemo as shipped does not run batch and online together: its batch sequence first closes the files in CICS (CLOSEFIL), its main VSAM clusters are SHAREOPTIONS(2 3), all 18 programs are CONCURRENCY(QUASIRENT), and all 8 files are STRINGS(1) and RECOVERY(NONE). Running both together there needs a way for batch to share the files with CICS: either VSAM record-level sharing (RLS), which needs a coupling facility (and Transactional VSAM, a separately priced feature, if the files are made recoverable), or batch file access routed through CICS (the external CICS interface, EXCI); threadsafe program definitions or several CICS regions; and string and buffer tuning. These changes are part of the z/OS configuration and are disclosed with it (docs/PORTING-zOS.md).
4. **Pinning:** cores pinned on both sides, and frequency governor and SMT recorded.
5. **A manifest per run:** CPU model and machine type, core and thread counts, memory, OS and kernel, compiler and library versions, configuration switches, and the git commit of the kit.
6. **Raw output:** every raw file is kept and published with the results.
7. **Pre-registration:** a hypothesis file entry exists, committed, before the first run that tests it.
8. **Full Disclosure Report:** every published result comes with an FDR following [docs/FDR-TEMPLATE.md](docs/FDR-TEMPLATE.md). A result without one is not a result of this benchmark.

## 5. Out of scope (v0.1)
- Audited or official benchmark status (TPC/SPEC). Results are research results. This kit is not a TPC benchmark, and results must not be presented as TPC results or described with TPC benchmark names. A separately sponsored, audited run of an actual TPC benchmark is listed as a stretch goal in ROADMAP.md.
- Db2, IMS and MQ variants of CardDemo (later, possibly).

## 6. Benchmark materials from third parties
To stay within other benchmark owners' terms:
1. **Download, never vendor.** Third-party benchmark kits and specification text are never committed to this
   repository.
2. **Permission first.** No module derived from industry-standard benchmark material, and no result from one, is
   published until the benchmark owner's written permission is in place.
3. **No borrowed names or metrics.** Our own workloads (the CardDemo-based online track and the batch suite) are
   never described with another benchmark's name, and other benchmarks' metric names are never used for our numbers.
4. **Academic framing only:** no marketing claims based on results derived from benchmark material.

TPC and TPC Benchmark are trademarks of the Transaction Processing Performance Council.
