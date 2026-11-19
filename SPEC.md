# Open Mainframe Benchmark: specification, draft v0.1 (2026-11-19)

Status: draft for comment. Items marked **(proposed)** are not yet decided. Version numbers follow semantic versioning:
results always name the spec version they ran against.

## 1. Workload

### 1.1 Base application (proposed)
- **Core: [AWS CardDemo](https://github.com/aws-samples/aws-mainframe-modernization-carddemo)** (Apache 2.0). It is a COBOL credit-card management application: CICS online transactions over VSAM (KSDS with alternate indexes), plus a batch suite.
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
- A deterministic generator with a scale factor S (accounts = 10,000 × S, transactions per account stated).
- The same seed gives byte-identical data files on every platform. Golden checks compare decompressed bytes, because hardware and software deflate output differ.

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
