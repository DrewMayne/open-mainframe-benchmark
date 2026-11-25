# Vendors measure mainframes; almost nobody else does

> **Note on TPC names in this review.** Where third-party sources describe their own workloads or results with TPC
> benchmark names (e.g. "TPC-C-like", "TPC-E-derived"), those are the sources' claims, quoted for context. They are
> not TPC results and are not comparable to published TPC results; official TPC results are on tpc.org.


Status: review as of 2026-11-19. Compiled from six research strands (academic; IBM material; vendors, rehosting and consolidation; analysts, TCO and critiques; standard benchmarks and community; zRef). This is a research review, not an endorsement of any source.

Provenance markers are carried over from the notes unchanged:
- "fetched" means a researcher read the primary page or PDF text.
- "not fetched; from search snippet; not verified" means only a search-engine summary was seen. Such figures are not verified and are not to be relied on until the primary source is read.
- "spot-checked" means a second reader re-checked the figure against the fetched source.
- "our arithmetic" marks a derived figure that no source states.

Across about 125 de-duplicated items, almost no independent, reproducible, same-lab measurement of an IBM Z machine against an x86 server exists. There is one academic head-to-head with statistics, a 2014 Binghamton master's project with 4 cores a side. Beyond it are a handful of small community runs: a Go issue with ns/op on z15/z16/z17, a bbolt insert benchmark on a Marist LinuxONE guest, and one mailing-list gzip timing. Almost everything with numbers comes from a party selling one side:
- IBM, or studies IBM commissioned;
- HP(E) and the rehosting vendors, whose proprietary zRef benchmark is the closest published analogue to a CICS-under-batch-sort test.

No audited TPC, SPEC, MLPerf or Sort Benchmark entry for any IBM mainframe was found, and IBM said on record (1998, 2003) that it chose not to publish. Most of IBM's accelerator figures (AIU, DFLTCC, Z Sort, CPACF) compare Z with an older Z, with a general core on the same chip, or with software on the same box. IBM's inference comparisons with x86 (IB01; IB04 fn1 and fn4) run the x86 inference remotely or use different software on each side; none found runs the same in-process code at the same batch size on both. IBM's x86 comparisons mostly rest on one of two things:
- a remote x86 server reached over injected network delay;
- an estate model built on assumed x86 utilisation.

Equal-core tests of sort, compression, AI per transaction, signatures, a mixed workload grid, online-under-batch, random read-modify-write and choke behaviour therefore sit in a genuine gap. That gap is what this project's specification targets.

## 1. Executive summary

- **Independent same-lab work is almost absent.** The only independent academic head-to-head that was fully read is FitzGerald (Binghamton, 2014): 4 zEC12 cores against a 4-core Xeon X3440 with HT off, SysBench, 95% confidence intervals and t-tests ([report](https://www.ibm.com/support/pages/zvm/devpages/dfitzger/report.pdf), fetched; title, author and date spot-checked). Its result is split. x86 led on CPU. s390x led on memory write (30-173%) and on MySQL at L0/L1, but fell off sharply under nested I/O.
- **The reproducible community items are few and small.** golang/go#82520 gives P-256 ECDH ns/op on z15/z16/z17, accelerated against pure Go ([issue](https://github.com/golang/go/issues/82520), fetched, spot-checked). goboltbench (2025) times bbolt inserts on a Marist LinuxONE guest against EPYC and Graviton3 ([repo](https://github.com/jankammerath/goboltbench), fetched). A LINUX-390 post (2020) times gzip on z14 against z15 ([post](https://www.mail-archive.com/linux-390@vm.marist.edu/msg72967.html), fetched). A GitHub crypto suite exists but is marked low trust.
- **No mainframe appears in any standard benchmark.** No TPC, SPEC, MLPerf or Sort Benchmark (1987-2023) entry for an IBM mainframe was found. An IBM VP said in 1998 that IBM "has run the TPC-C test on MVS systems, but never released them" ([Tech Monitor](https://www.techmonitor.ai/?p=92089), fetched). In 2003 IBM said "we typically don't run industry standard performance benchmarks for any software on the mainframe" (Computerworld and InfoWorld, fetched). These are absences found by search, not exhaustive database queries.
- **Most of IBM's accelerator figures are Z against Z, and its x86 accelerator comparisons are not like for like.**
  - Telum AIU: 22x latency against a SIMD core on the same chip (ISCA 2022, not fetched; from search snippet; not verified).
  - DFLTCC: 24-42x against software zlib on 4 z15 IFLs ([GSE UK 2021](https://conferences.gs-uk.org/2021/presentations/2AV.pdf), fetched).
  - Z Sort: 20-30% less elapsed time and up to 40% less CPU, against **z14**, not against Z Sort switched off ([SHARE blog](https://blog.share.org/Article/peeking-under-the-hood-of-sort-acceleration-on-z15), fetched).
  - CPACF P-256: 48,660 signatures/s on one z15 CPACF ([6K2653EJ](https://ibm.com/downloads/cas/6K2653EJ), fetched).
  - None of these has an x86-software baseline, and that is the main gap our tests fill.
  - The exceptions are IBM's inference comparisons with x86. IB01 sets z16 inference co-located with a CICS transaction (WMLz on zCX) against TensorFlow Serving on an x86 server behind 60 ms of added network delay, simulating a remote cloud. In IBM's AI Toolkit footnotes (IB04), fn1 matches one z17 accelerator against a remote x86 server running Triton with OpenVINO on CPU (AMX), and fn4 compares Snap ML on the z16 accelerator with NVIDIA FIL on 6 Ice Lake cores.
- **IBM's x86 comparisons come in three types:**
  - **Co-location against a remote x86.** The z16's "20x lower response time" added 60 ms of `tc-netem` delay on the x86 path ([M65ZR5DV](https://ibm.com/downloads/cas/M65ZR5DV), fetched).
  - **Estate extrapolation.** "Up to 2,944 x86 cores" is mostly 2,688 modelled non-production cores at 7-15% utilisation ([Signal65](https://signal65.com/wp-content/uploads/2025/10/Signal65-TCO-Analysis_Workload-Consolidation-on-IBM-LinuxONE.pdf), fetched).
  - **Core-matched.** Snap ML on 6 z16 IFLs against 6 Ice Lake cores ([AI Toolkit](https://www.ibm.com/products/ai-toolkit-for-z-and-linuxone), fetched).
  - Only the core-matched type resembles our 2-vs-2 design.
- **zRef is the closest published analogue to online under concurrent batch.** It is HP's 2013 TPC-E-derived COBOL/CICS/DB2 application with a sort/merge batch job (zRefBR). TmaxSoft's Azure study gives 1,672 TPS online, and 1,409 TPS with 3 zRefBR jobs alongside (different user counts). zRefBR took 16 min with CICS running against 13 min without ([TmaxSoft study](https://www.tmaxsoft.com/upload/eBookMgmt/20231219134111167015.pdf), fetched, spot-checked). The kit is proprietary, every run is vendor-run, and HP's original paper could not be retrieved.
- **The only published Z-side figure for batch interference** is IBM REDP-5170: CICS CPU per transaction rose "up to 15%" on a zEC12 when batch ran in the same LPAR, through cache contention ([REDP-5170](https://www.redbooks.ibm.com/redpapers/pdfs/redp5170.pdf), fetched). No study measures WLM against Linux cgroups under the same batch load.
- **Sources contradict each other on batch.**
  - Heirloom, a rehoster, reports a 45-minute mainframe batch job taking 8 hours after a cloud move, which it illustrates with a hypothetical 2 ms per query ([Heirloom](https://heirloomcomputing.com/the-latency-challenge/), fetched).
  - Microsoft halves its MIPS-per-vCPU sizing for batch (snippet; not verified).
  - Yet TmaxSoft's zRef study reports standalone zRefBR at 68.1 min on the z/OS baseline against 6.21 min on Azure (fetched, spot-checked).
  - Both claims are shown here, and neither is independent.
- **Consolidation and TCO ratios are models, not per-core speed.**
  - The measured layer runs at about 9:1 to 16:1 cores at equal throughput, against older x86: 448 Broadwell cores against 50 IFLs at 48,974 tps ([G3QK9D8V](https://ibm.com/downloads/cas/G3QK9D8V), fetched); 256 older Xeon cores against 16 IFLs at about 10,000 req/s (Signal65).
  - IBM's own old equal-core claim is about 2x per core: "2.1 times as many Node.js Web transactions" at 4 cores each (Linux Journal, fetched).
  - MIPS-per-x86-core rules in circulation vary widely by source (VN05-VN07, VN09, VN12).
- **Batch size and placement decide AI headlines.** IBM's inference figures use batch 128-160 and threads pinned one per chip. No IBM source gives an absolute batch-1 latency. A per-transaction (batch 1) test on a 2-CP slice that does not own an accelerator is a different regime, and IBM's numbers are not its expected value.

## 2. Top 15 most relevant items, ranked

Ranked by how directly each one informs or checks one of our eight workloads, weighted by methodological quality. Catalogue IDs are in brackets.

1. **FitzGerald, "A Comparative Analysis of Nested Virtualization in x86_64 and S390x Hypervisors", Binghamton University, 2014 [AC01].**
   - Why first: it is the only independent, equal-core, same-lab, statistically reported head-to-head: 4 dedicated zEC12 cores against a 4-core Xeon X3440 with HT off, the same custom kernel on both, 95% CIs and t-tests.
   - Its split verdict is the precedent to test against: x86 ahead on CPU arithmetic, s390x ahead on memory write and on MySQL until nested ([report](https://www.ibm.com/support/pages/zvm/devpages/dfitzger/report.pdf), fetched; title, author and date spot-checked).
2. **zRef, HP's z/OS Reference Application (2013), with the TmaxSoft/Microsoft OpenFrame-on-Azure study [VN01, VN02].**
   - Why second: it is the closest published analogue to our CICS-like online test under a concurrent batch sort. It has 10 CICS transactions on a TPC-E schema, plus a batch job that extracts from DB2 and then sort/merges (zRefBR), and online and batch run together by design.
   - TmaxSoft's figures (fetched, spot-checked): 1,672 / 2,600 / 1,409 TPS; zRefBR 16 min against 13 min under CICS; standalone zRefBR 68.1 against 6.21 min; zRefBU 161.9 against 32.1 min.
   - It ranks below FitzGerald on merit, because everything is vendor-run, the kit is proprietary, the IBM baseline is undisclosed, and zRef "was not built to 'achieve the highest throughput possible'".
3. **IBM REDP-5170, "IBM CICS Performance Series: Effective Monitoring for CICS Performance Benchmarks", 2015 [IB22].**
   - The only measured Z-side number for online cost under co-resident batch: up to 15% more CICS CPU per transaction on a zEC12, from cache contention, even with dedicated CPs ([REDP-5170](https://www.redbooks.ibm.com/redpapers/pdfs/redp5170.pdf), fetched).
   - It is the reference value for our batch+online test, and a warning that run-to-run variance on a shared LPAR needs reporting.
4. **golang/go#82520, "use CPACF PCC for X25519, P-384 and P-521", 2026 [CM01].**
   - An open, reproducible `go test -bench` table across three Z generations. P-256 ECDH ns/op, accelerated against pure Go (spot-checked):
     - z17: 100,468 vs 319,715;
     - z16: 139,740 vs 638,626;
     - z15: 123,998 vs 440,240.
   - It is the best public per-operation cross-check for our CPACF signature runs ([issue](https://github.com/golang/go/issues/82520), fetched).
5. **Raspl (IBM), "Linux on IBM Z and LinuxONE: What's New", GSE UK 2021 [IB07].**
   - `minigzip -1`, software zlib against DFLTCC, on z15 with 4 IFLs, over public corpora: E.coli 33.9x, bible.txt 30.3x, world192.txt 24x, Canterbury.tar 42x ([slides](https://conferences.gs-uk.org/2021/presentations/2AV.pdf), fetched).
   - It is the closest published analogue to our DFLTCC-against-software-zlib test, and the files can be rerun exactly. Its x86 side is missing.
6. **IBM zlib PR #410 and the Linux kernel DFLTCC series, 2019-2020 [IB08].**
   - Per-corpus hardware/software ratios at level 1, with compression ratios:
     - deflate: zeroes 1.46x, random ASCII 10.44x, Dickens 6.21x, vmlinux 8.37x;
     - inflate: 1.02x to 3.90x;
     - compression ratio against software: 0.94 to 1.02.
   - It shows the method matters as much as the hardware ([zlib PR #410](https://github.com/madler/zlib/pull/410), fetched).
7. **Gentile and Shugol (IBM), "Peeking Under the Hood of Sort Acceleration on z15", SHARE blog, 2020 [IB12].**
   - The only published Z Sort numbers: up to 20-30% elapsed and up to 40% CPU against z14 DFSORT, over an 80 B to 27 KB record and 8-500 B key grid ([blog](https://blog.share.org/Article/peeking-under-the-hood-of-sort-acceleration-on-z15), fetched).
   - The grid is a design to borrow. Its baseline is the reason our ZSORT on/off contrast is cleaner.
8. **Watson and Walker, "Optimizing DFSORT Use of Z Sort Accelerator", Tuning Letter 2022 No. 2, IBM-hosted reprint [AN03].**
   - An independent consultancy's checklist of why Z Sort silently does not run: input under 32 MB, INREC/OUTREC/OUTFIL/SUM/JOINKEYS, VSAM, memory objects under 75% of the file. It also names the evidence that it did run (ICE267I, SMF 16 ICEZSRNU) ([reprint](https://ibm.com/support/pages/system/files/inline-files/Optimizing%20DFSORT%20Use%20of%20Z%20Sort%20Accelerator.pdf), fetched).
9. **IBM, "IBM z15 Performance of Cryptographic Operations (CPACF, CEX7S)", 2020 [IB13].**
   - Methodical per-CPACF figures with data-length sweeps and scaling statements:
     - P-256 sign 48,660/s and verify 19,365/s through PKCS#11;
     - AES-128-CBC about 1.13 GB/s at 64 B and about 5.1 GB/s at large sizes;
     - RSA-2048 TLS handshakes at 255/s in software against 10,847/s with a CEX7C.
   - These are the per-CP reference values for "signatures per payment" ([6K2653EJ](https://ibm.com/downloads/cas/6K2653EJ), fetched).
10. **IBM, "AI on IBM zSystems: Accelerating AI inference for high volume transactions", ADMS 2023 slides [AC10].**
    - The closest published workload to our "AI checks per transaction": a CICS credit-card transaction LINKs to an in-transaction fraud-model scoring program. Headline: 228K tx/s at 6 ms.
    - The figure is extrapolated from a **6-CP** LPAR to a 200-CP machine ([slides](https://adms-conf.org/2023-slides/ADMS23_z16.pdf), fetched).
11. **IBM, "AI Toolkit for IBM Z and LinuxONE" product page, footnotes, 2025-26 [IB04].**
    - fn4 is the only core-matched IBM inference comparison: Snap ML on 6 z16 IFLs against NVIDIA FIL on 6 Ice Lake cores, "up to 3.5x".
    - fn1 reports one z17 accelerator as "a 13-core x86 server" against current Emerald Rapids with AMX, which is a reporting unit worth borrowing ([page](https://www.ibm.com/products/ai-toolkit-for-z-and-linuxone), fetched).
12. **Lichtenau et al. (IBM), "AI accelerator on IBM Telum processor: industrial product", ISCA 2022 [AC07].**
    - The peer-reviewed accelerator-against-CPU ratio on the same chip: "22x speed up in latency compared to a general purpose core utilizing vector execution units", and 116k inferences/s at 1.1 ms per chip ([ACM DL](https://dl.acm.org/doi/10.1145/3470496.3533042), not fetched; from search snippet; not verified).
13. **Linde (IBM), "IBM z16: Reasons to upgrade from your z15 or z14", 2022 [IB01].**
    - This is the clearest example of the remote-x86 trap: "20x lower response time and up to 19x higher throughput" against an 8-core Skylake TF Serving box with 60 ms of injected `tc-netem` delay.
    - Its z16 side (z/OS, zCX, a CICS caller, the public fraud model) is a configuration we can mirror ([M65ZR5DV](https://ibm.com/downloads/cas/M65ZR5DV), fetched).
14. **Kammerath, goboltbench, GitHub, 2025 [CM03].**
    - An independent, published-script RMW/commit-style benchmark: bbolt inserts plus SHA-256, 16 workers.
    - Results: Marist LinuxONE 4 GB guest 2.826 s, Graviton3 1.850 s, EPYC 9R14 1.852 s, Apple M1 Pro 12.04 s. The M1 result shows storage and fsync behaviour dominate ([repo](https://github.com/jankammerath/goboltbench), fetched).
    - It has a single run and does not state vCPU counts.
15. **Cunningham (Heirloom), "Mainframe Migration - The Latency Challenge", 2025 [VN10].**
    - A rehoster's anecdote, used to promote its own approach: a batch job of 2.3 million DB reads took 45 minutes on the mainframe and 8 hours after the cloud move ([post](https://heirloomcomputing.com/the-latency-challenge/), fetched).
    - It is direct support for a per-record-latency RMW test, though it gives no hardware specification and its hypothetical arithmetic does not explain 8 hours.

**Just outside the 15:**
- IBM AM1PYZBB [IB11]: the only IBM measurement at **2 CPs**.
- Signal65 [AN01]: the best-documented estate model.
- IBM/Temenos [CS02]: TPS per core against offered load, plus a utilisation footnote.
- Bradbury and Hess [AC11]: cycle-normalised cross-ISA crypto.
- Sort Benchmark JouleSort [SB05]: audited Ryzen records per joule.
- OpenBenchmarking 2-vCPU z15 uploads [SB08]: the only same-size standard-suite numbers.


## 3. Full catalogue, by source type

Each entry gives:
- the title, author or org, and year;
- the URL with its fetch status;
- what was compared, the workload and the headline result;
- quality and independence;
- which of our workloads it maps to;
- what is useful to us.

Workload tags: sort, compression, AI inference, crypto, mixed grid, batch+online mix, RMW/IO, choke/limits, cost/energy.

Cross-references: "Also in" names the other strand notes that carry the same item. Where strands disagree on fetch status, the entry says which strand fetched it.

### 3.1 Academic and peer-reviewed (22 items)

**AC01. "A Comparative Analysis of Nested Virtualization in x86_64 and S390x Hypervisors"**
- **Who / when:** Daniel FitzGerald (advisor Dennis Foreman), Binghamton University, Watson School, Masters Termination Project v1.0, 14 Dec 2014. Hosted on IBM's z/VM devpages.
- **URL:** https://www.ibm.com/support/pages/zvm/devpages/dfitzger/report.pdf (fetched, PDF text extracted; title, author and date spot-checked by a second reader).
- **Compared:**

  | | s390x | x86 |
  | --- | --- | --- |
  | Machine | zEC12-H89 LPAR | IBM System x3200 M3, Xeon X3440 |
  | Cores | 4 dedicated single-threaded cores at 5.5 GHz | 4 cores at 2.53 GHz, HT disabled |
  | Memory | 16 GB | 16 GB |
  | Highest cache | 48 MB L3, 384 MB L4 | 8 MB L3 |
  | Hypervisor | z/VM 6.3 | KVM with nested virtualization ("Turtles") |

  Both sides ran a custom Linux 3.2.60 kernel built without transparent huge pages.
- **Workload:** SysBench cpu (primes), threads, memory write and read, and MySQL, at L0, L1 and L2 nesting.
- **Headline:**
  - "x86_64 CPU performance consistently outperformed S390x CPU performance irrespective of the degree of virtualization". The authors attribute this to the prime test suiting x86 floating point.
  - "an S390x virtual machine will maintain a 30-173% higher write throughput and a 22-77% faster response time" for memory writes.
  - MySQL on s390x was "vastly superior" at L0 and L1, then dropped in a "jarring" way from L1 to L2. The authors blame SIE exits on guest I/O.
- **Quality / independence:**
  - Student project, reviewed only by the advisor. It appears independent of IBM funding, though IBM hosts it.
  - Good statistics: 95% CIs and two-sample t-tests.
  - Weaknesses: a 2012 flagship mainframe against a 2009 entry-level Xeon, a toy CPU test, and Linux 3.2.
- **Maps to:** choke/limits, RMW/IO, mixed grid.
- **Useful to us:**
  - It is the closest precedent to our equal-core design (4 vs 4 here, 2 vs 2 for us).
  - Borrow: matched core counts, SMT/HT off, the same kernel build on both sides, and CI plus t-test reporting.
  - Gap it leaves: no accelerators and no modern hardware. Our runs add both.
- **Also in:** benchmarks_community item 16.

**AC02. "Data Compression Accelerator on IBM POWER9 and z15 Processors: Industrial Product"**
- **Who / when:** Abali et al., IBM. ISCA 2020, industry track (peer-reviewed).
- **URL:** https://aminer.org/pub/5f0d85c69fced0a24be4f052/data-compression-accelerator-on-ibm-power-and-z-processors-industrial-product (not fetched; from search snippet; not verified). The aminer page returned only a header.
- **Compared:** the z15 on-chip DEFLATE unit (NXU) against prior-generation and zEDC baselines. The unit is shared by the 12 cores of a chip through L3 and driven by the synchronous DFLTCC instruction.
- **Workload:** compression.
- **Headline:** "up to 280 GB/s data compression rate" on a fully configured z15. No x86 baseline appears in the snippet.
- **Quality / independence:** peer-reviewed, but written by IBM. The figure is a whole-machine aggregate.
- **Maps to:** compression.
- **Useful to us:**
  - It is the design citation for DFLTCC.
  - A 2-CP slice shares one chip's unit with up to 11 other cores, which may belong to other LPARs.
  - Gap: no per-stream figure and no x86 zlib comparison.

**AC03. "Integration of z15 processor-based DEFLATE acceleration into IBM z/OS"**
- **Who / when:** J.D. Bradbury, A. Chatterjee et al. IBM Journal of Research and Development 64(5/6), 2020.
- **URL:** https://www.bohrium.com/paper-details/integration-of-z15-processor-based-deflate-acceleration-into-ibm-z-os/812571668235943937-2701 (not fetched; from search snippet; not verified). The IBM Research page redirects to a 404.
- **Compared:** two paths into the accelerator:
  - user-space zlib, which runs DFLTCC synchronously;
  - DFSMS access methods, which go through a System Assist Processor asynchronously.
- **Workload:** compression of sequential data sets.
- **Headline:** the abstract promises "performance on selected workloads", but no numbers were seen.
- **Quality / independence:** IBM journal.
- **Maps to:** compression, RMW/IO.
- **Useful to us:**
  - The sync/async split explains why, on z/OS, a DFSMS-compressed data-set test and a zlib or Java test can show different CPU time.
  - Gap: the numbers were not read.

**AC04. "Design and verification of DEFLATE acceleration as an architected instruction in z15"**
- **Who / when:** IBM. IBM JRD, 2020.
- **URL:** https://www.aminer.org/pub/5ff7d26fd4150a363cb6b43a/design-and-verification-of-deflate-acceleration-as-an-architected-instruction-in-z (not fetched; from search snippet; not verified).
- **Compared:** none. This is a hardware design paper; it includes a dynamic-Huffman table generator.
- **Workload:** compression.
- **Headline:** no benchmark.
- **Quality / independence:** IBM journal.
- **Maps to:** compression.
- **Useful to us:** background only.

**AC05. "Design of the IBM z15 microprocessor"**
- **Who / when:** IBM. IBM JRD 64(5/6), 2020, pp. 7:1-7:18.
- **URL:** https://ieeexplore.ieee.org/document/9138680 ; https://www.researchgate.net/publication/344562481_Design_of_the_IBM_z15_microprocessor (not fetched; from search snippet; not verified).
- **Compared:** z15 against z14.
- **Workload:** the abstract mentions the hardware sort accelerator.
- **Headline:** no sort speedups were visible.
- **Quality / independence:** IBM journal.
- **Maps to:** sort.
- **Useful to us:** it confirms that **no dedicated peer-reviewed SORTL/Z Sort paper with measured speedups was found**. The only published numbers are in IB12.

**AC06. "The IBM z15 processor chip set"**
- **Who / when:** A. Saporito, IBM. Hot Chips 32, 2020 (slides).
- **URL:** https://hc32.hotchips.org/assets/program/conference/day1/HotChips2020_Server_Processors_IBM_Saporito_z15.pdf (fetched, PDF text extracted).
- **Compared:** z15 against z14.
  - Chip: 12 cores at 5.2 GHz, 256 MB L3, up to 240 physical cores.
  - Claim: "14% single thread speedup & 25% capacity growth vs z14".
- **Workload:** ECC public-key sign and verify. The deck also describes the deflate and sort units.
- **Headline:** on-core ECC on z15 against a Crypto Express6 card on z14:

  | Curve | Sign | Verify |
  | --- | --- | --- |
  | P-256 | 21.8x | 14.5x |
  | P-521 | 17.5x | 11.4x |

  - IBM's footnote: the speedup is "a combination of faster processing and the reduced latency of synchronous on core execution".
  - Deflate design: a CAM for the 512 B near history, plus a hash table out to 32 KB.
  - No sort numbers.
- **Quality / independence:** IBM slides, not peer-reviewed.
- **Maps to:** crypto, compression, sort.
- **Useful to us:**
  - The ECC baseline is a PCIe card, not software on x86, so it does not answer "CPACF vs software".
  - The near/far history design means DFLTCC compression ratios can differ from zlib at -6 or -9. Record the ratio alongside throughput.

**AC07. "AI accelerator on IBM Telum processor: industrial product"**
- **Who / when:** Lichtenau et al., IBM. ISCA 2022, industry track. DOI 10.1145/3470496.3533042.
- **URL:** https://dl.acm.org/doi/10.1145/3470496.3533042 ; https://unpaywall.org/10.1145%2F3470496.3533042 (not fetched; ACM returned 403, and the numbers come from IBM Research and Unpaywall snippets; not verified).
  - Related: the Hot Chips 33 Telum slides (Jacobi), https://hc33.hotchips.org/assets/program/conference/day1/HC2021.C1.3%20IBM%20Cristian%20Jacobi.pdf (not fetched).
- **Compared:** the on-chip AIU against a general-purpose Telum core using its vector units, on the same chip.
- **Workload:** "a global bank customer credit card fraud detection model" (LSTM-class).
- **Headline:**
  - "22x speed up in latency".
  - 116k inferences/s at 1.1 ms on one chip.
  - Over 3.5 million/s at about 1.2 ms on 32 chips.
  - Over 200 TFLOPS on 32 chips; more than 40% per-socket growth over z15.
  - 3.5M/s x 86,400 is about 300 billion/day, which matches the z16 marketing figure (our arithmetic, IBM strand).
- **Quality / independence:**
  - Peer-reviewed, but IBM-authored, with an IBM-chosen model.
  - Conflict flag: a search also returned an "ICS 2025" listing with the same title and numbers. The academic strand suspects a conflation; cite ISCA 2022.
- **Maps to:** AI inference.
- **Useful to us:**
  - The only peer-reviewed accelerator-vs-CPU ratio on the same chip. It is the analogue of our AIU-vs-CPU-per-transaction test.
  - Gap: no x86 or GPU baseline.
- **Also in:** ibm_material A5; benchmarks_community item 12.

**AC08. "IBM Telum: a 16-Core 5+ GHz DCM"**
- **Who / when:** O. Geva, C. Berry et al. ISSCC 2022.
- **URL:** https://researcher.ibm.com/publications/ibm-telum-a-16-core-5-ghz-dcm (not fetched; from search snippet; not verified).
- **Compared:** none; circuit design only.
- **Chip:** 22B transistors per chip, 8 cores, a 32 MB L2 per core. Press coverage adds a virtual 256 MB L3 and 2 GB L4.
- **Workload:** none.
- **Headline:** none.
- **Quality / independence:** IBM, peer-reviewed. Sources disagree on whether the package is a DCM or a single module.
- **Maps to:** context for AI inference and for choke/limits (cache).
- **Related:** ISSCC 2015, "22nm Next-Generation IBM System z Microprocessor" (z13 against zEC12 only), https://unpaywall.org/10.1109/ISSCC.2015.7062930 (not fetched; from search snippet; not verified).
- **Useful to us:** cache sizes for explaining noisy-neighbour effects on a shared LPAR.

**AC09. "IBM Telum II processor and IBM Spyre Accelerator chip for AI"**
- **Who / when:** C. Berry, IBM. Hot Chips 2024.
- **URL:** https://www.hc2024.hotchips.org/assets/program/conference/day1/04_HC2024.IBM.CBerry.final.pdf (fetched, PDF text extracted).
  - Related: the Telum II announcement blog, https://www.ibm.com/blog/announcement/telum-ii (not fetched; from search snippet; not verified). IBM itself says there that "TOPS alone don't tell the whole story".
- **Compared:** Telum II against Telum I only. The slides say "All projections are based on pre-release hardware".
- **Workload:** the fraud model at github.com/IBM/ai-on-z-fraud-detection.
  - Footnote: "extrapolated from IBM internal tests" on a z16 LPAR with 48 IFLs and 128 GB, Ubuntu 20.04, "8 parallel threads each pinned to the first core of a different chip", batch 128.
  - Repeated on z/OS V2R4 with 24 CPs, a single thread and batch 128.
- **Headline:**
  - AIU: 24 TOPS per chip, 192 per drawer, 768 per system, INT8 and FP16.
  - DPU: 70% less power for I/O management.
  - Spyre: "300+ TOPS" at 75 W.
- **Quality / independence:** IBM slides of projections.
- **Maps to:** AI inference.
- **Useful to us:**
  - **The benchmark model is public**, so we can run IBM's own model.
  - IBM uses batch 128. Ours is per transaction (batch 1), a latency-bound regime.

**AC10. "AI on IBM zSystems: Accelerating AI inference for high volume transactions"**
- **Who / when:** IBM. ADMS 2023 (a VLDB workshop); invited-talk slides.
- **URL:** https://adms-conf.org/2023-slides/ADMS23_z16.pdf (fetched, PDF text extracted).
- **Compared:** z16 only; a scaling claim.
- **Workload:** z/OS CICS credit-card transactions, each with an in-transaction deep-learning fraud inference.
  - CICS LINKs to the scoring program ALNSCORE over a channel.
  - WMLz 2.4 on Liberty 21.0.0.12, server-side batching of 8, 48 threads.
- **Headline:**
  - "up to 228K z/OS CICS credit card transactions per second with 6 ms response time".
  - "extrapolated from IBM internal tests" on a z/OS V2R4 LPAR with **6 CPs** and 256 GB.
  - "Results represent a fully configured IBM z16 with 200 CPs and 40 TB storage."
  - Standalone model: "Almost perfect scaling up to 32 chips" at batch 128.
- **Quality / independence:** IBM, not refereed. The 6-CP to 200-CP step is a linear extrapolation.
- **Maps to:** AI inference, mixed grid, batch+online mix.
- **Useful to us:**
  - The closest published shape to our document-size x AI-checks x signatures grid: inference inside an online transaction.
  - A measured 2-CP figure is a more defensible unit than a 200-CP extrapolation.
  - 228K / 200 CPs is about 1.1K tx/s per CP, as extrapolated, not measured (our arithmetic).

**AC11. "Fast Quantum-Safe Cryptography on IBM Z"**
- **Who / when:** J. Bradbury and B. Hess, IBM. Accepted paper at the NIST 3rd PQC Standardization Conference, 2021.
- **URL:** https://csrc.nist.gov/CSRC/media/Events/third-pqc-standardization-conference/documents/accepted-papers/hess-fast-quantum-safe-pqc2021.pdf (fetched, PDF text extracted). Slides: https://csrc.nist.gov/CSRC/media/Presentations/fast-quantum-safe-cryptography-on-ibm-z/images-media/session-9-hess-fast-quantum-safe.pdf
- **Compared:**
  - z15 LPAR at 5.2 GHz, Ubuntu 21.04 beta, clang-12 `-march=z15 -mzvector -mvx -O3`;
  - against portable C on the same box;
  - and against Skylake, ARMv8-A and ARMv7-M cycle counts **taken from the literature**, not run in the same lab.
- **Workload:** SIKE encaps+decaps; Dilithium2 using the CPACF SHA-3 instructions.
- **Headline:**
  - SIKE encaps+decaps is 4.1-5.0x faster than portable C (Table 1: 4.3x, 4.1x, 4.9x and 5.0x for p434 to p751; the text gives 4.2x for p503).
  - Encaps+decaps, in thousands of cycles:

    | Parameter set | z15 | Skylake |
    | --- | --- | --- |
    | p434 | 17,818 | 20,024 |
    | p503 | 28,667 | 27,959 |
    | p610 | 46,738 | 54,699 |
    | p751 | 75,260 | 84,553 |

    x86 is ahead at p503. The authors: "up to a factor 1.17" in cycles, with 5.2 GHz giving "a further advantage in absolute timings".
  - Dilithium2: sign 42.8 us, verify 14.7 us.
- **Quality / independence:** lightly refereed, IBM-authored. The cross-ISA columns are not same-lab, and SIKE was broken in 2022.
- **Maps to:** crypto.
- **Useful to us:**
  - A rare example of reporting cycles and wall time separately across ISAs, a normalisation worth borrowing.
  - Dilithium (now ML-DSA) gives a per-signature anchor.
  - Gap: no classical ECDSA or RSA CPACF comparison against x86 OpenSSL.

**AC12. "The RNI-based LSPR and The IBM z14 Performance Brief"**
- **Who / when:** D. Hutton, IBM. CMG 2017.
- **URL:** https://cmg.org/wp-content/plugins/s2member-files/proceedings/2017/433_Hutton.pdf (not fetched; from search snippet; not verified).
- **Compared:** Z generations.
- **Workload:** LSPR capacity categories.
- **Headline:**
  - RNI "reflects the average storage hierarchy depth at which L1 misses are typically serviced".
  - The LOW, AVERAGE and HIGH categories come from the L1 miss percentage and RNI.
- **Quality / independence:** IBM, at a practitioner conference.
- **Maps to:** choke/limits, mixed grid.
- **Useful to us:** IBM's own statement that Z capacity depends on cache behaviour, which is what shared-LPAR noise on our slice reflects.

**AC13. "Mainframe Capacity Management"**
- **Who / when:** Johnson. CMG 2017 (slides).
- **URL:** https://cmg.org/wp-content/plugins/s2member-files/proceedings/2017/224a_Johnson.pdf (not fetched; from search snippet; not verified).
- **Compared:** none.
- **Workload:** capacity-management practice.
- **Headline:** argues that mainframe capacity management should extend to cloud. No quantitative comparison.
- **Quality / independence:** practitioner.
- **Maps to:** choke/limits.
- **Useful to us:** it confirms the CMG material found is methodology, not cross-platform results.

**AC14. "Adaptive algorithms for managing a distributed data processing workload"**
- **Who / when:** J. Aman, C.K. Eilert, D. Emmes, P. Yocom, D. Dillenberger. IBM Systems Journal 36(2):242-283, 1997.
- **URL:** https://research.ibm.com/publications/adaptive-algorithms-for-managing-a-distributed-data-processing-workload ; https://dblp1.uni-trier.de/rec/bibtex/journals/ibmsj/AmanEEYD97 (not fetched; from search snippet; not verified).
- **Compared:** none.
- **Workload:** OS/390 WLM goal-mode algorithms across a Parallel Sysplex.
- **Headline:** a design paper; no cross-platform results.
- **Quality / independence:** peer-reviewed, IBM-authored.
- **Maps to:** batch+online mix.
- **Useful to us:** the citation for *why* z/OS should protect online response time under batch. Our test checks whether that holds on a capped 2-CP slice.

**AC15. "Spyre-Accelerated Retrieval-Augmented Generation on IBM LinuxONE..."**
- **Who / when:** S. Bokkasam and Pankaj D. arXiv 2608.21393, v1, 2 Aug 2026.
- **URL:** https://arxiv.org/abs/2608.21393 (fetched; abstract page only).
- **Compared:** LinuxONE with Spyre against "off-platform", "cloud-GPU and on-premises" alternatives. None of these is defined.
- **Workload:** RAG.
- **Headline:** "Early analysis points to end-to-end RAG latencies under two seconds" and "up to a 20x reduction compared to off-platform inference". No figures appear on the abstract page.
- **Quality / independence:** unrefereed preprint with speculative wording.
- **Maps to:** AI inference.
- **Useful to us:** an example of a claim with no real baseline. It is the only arXiv s390x performance item found.

**AC16. "System z and z/OS unique Characteristics"**
- **Who / when:** W.G. Spruth. Universität Tübingen technical report WSI-2010-03, 2010.
- **URL:** https://tobias-lib.uni-tuebingen.de/xmlui/handle/10900/49392 ; PDF https://publikationen.uni-tuebingen.de/xmlui/bitstream/handle/10900/49392/pdf/report_spruth_2010.pdf?sequence=1 (fetch failed with ECONNREFUSED; from search snippet; not verified).
- **Compared:** qualitative only. As of 2010, x86, Itanium and SPARC had "no equivalent to the System z LPAR implementation".
- **Workload:** none measured.
- **Headline:** lists about 40 mainframe technologies.
- **Quality / independence:** an academic author, but an advocate who co-led IBM-supported teaching.
- **Related:** Spruth and Rosenstiel, *Informatik-Spektrum*, 2011, https://link.springer.com/article/10.1007/s00287-010-0487-6 (not fetched).
- **Maps to:** context.
- **Useful to us:** teaching background; no numbers.

**AC17. AlphaSort**
- **Who / when:** Nyberg, Barclay, Cvetanovic, Gray, Lomet. SIGMOD 1994, pp. 233-242; VLDB Journal 4(4), 1995.
- **URL:** https://courses.cs.washington.edu/courses/cse590q/05wi/paper/p233-nyberg.pdf (not fetched; from search snippet; not verified).
- **Compared:** commodity Alpha processors with SCSI disks, against a 32-CPU Hypercube and a Cray Y-MP.
- **Workload:** the Datamation sort: 1M 100-byte records with 10-byte keys.
- **Headline:**
  - 7 s, 8:1 ahead of the Hypercube.
  - About 4x faster and about 100x cheaper than the Cray.
  - No mainframe DFSORT entry was found in its comparison.
- **Quality / independence:** peer-reviewed, by authors who later founded the Sort Benchmark.
- **Maps to:** sort, cost/energy.
- **Useful to us:** the classic commodity-vs-big-iron sort paper. Use 100-byte records with 10-byte keys so our DFSORT vs GNU sort runs follow sortbenchmark.org conventions.

**AC18. Bresnahan and Greenstein: mainframe to client/server adoption**
- **Who / when:** Brookings Papers on Economic Activity: Microeconomics, 1996.
- **URL:** https://www.brookings.edu/wp-content/uploads/1996/01/1996_bpeamicro_bresnahan.pdf (not fetched; from search snippet; not verified).
- **Compared:** establishments moving from mainframe to client/server.
- **Workload:** none; economic duration models.
- **Headline:** high "co-invention" costs slowed adoption.
- **Quality / independence:** peer-reviewed economics.
- **Maps to:** cost/energy.
- **Useful to us:** historical framing only.

**AC19. Duncan and Lele: Telogy Inc. mainframe replacement case study**
- **Who / when:** Journal of Software Maintenance 8:321-344, 1996.
- **URL:** https://onlinelibrary.wiley.com/doi/abs/10.1002/(SICI)1096-908X(199609)8:5%3C321::AID-SMR135%3E3.0.CO;2-4 (not fetched; from search snippet; not verified).
- **Compared:** a mainframe replaced by client/server.
- **Workload:** none measured.
- **Headline:** a case study.
- **Quality / independence:** peer-reviewed case study.
- **Maps to:** cost/energy.
- **Useful to us:** historical framing only.

**AC20. MySQL on PC Intel against z9-series mainframes**
- **Who / when:** Kacprzak and Kolasinski, 2010 (Polish paper).
- **URL:** https://yadda.icm.edu.pl/baztech/element/bwmeta1.element.baztech-9f103968-5f4f-45ae-8de4-6b983a6ab688/c/Kacprzak_Kolasinski_in_2010-6.pdf (the fetch returned 403; contents unknown).
- **Compared:** PC Intel against z9, according to the search-result title.
- **Workload:** MySQL.
- **Headline:** unknown.
- **Quality / independence:** unknown.
- **Maps to:** RMW/IO (probable).
- **Useful to us:** a possible second independent head-to-head. Retrieve it before relying on it.

**AC21. "Detecting Noisy Neighbors in CPU-Isolated Cgroups Environments"**
- **Who / when:** ICPE 2025.
- **URL:** https://dl.acm.org/doi/10.1145/3676151.3719376 (not fetched; from search snippet; not verified).
- **Compared:** Linux cgroups only; there is no Z side.
- **Workload:** co-located containers, instrumented for scheduler latency and preemption with eBPF.
- **Headline:** cgroups "don't fully prevent performance interference".
- **Quality / independence:** peer-reviewed.
- **Maps to:** batch+online mix, choke/limits.
- **Useful to us:** the x86-side method for our online-under-batch test: measure the victim's p99 and scheduler latency, not just mean throughput.

**AC22. DC-Store (noisy-neighbour containers at the storage level)**
- **Who / when:** USENIX FAST'20; the file name suggests Kwon et al.
- **URL:** https://www.usenix.org/system/files/fast20-kwon.pdf (not fetched).
- **Compared:** Linux storage isolation only.
- **Workload:** storage I/O interference.
- **Headline:** not read.
- **Quality / independence:** peer-reviewed.
- **Maps to:** RMW/IO, batch+online mix.
- **Useful to us:** a reference design for I/O interference on the PC side.


### 3.2 IBM material (31 items)

Everything in this section is the vendor's own material, so IBM is the interested party. Apart from the public fraud model and Acme Air, none of it ships a runnable harness. Where a figure says "our arithmetic", the IBM strand derived it.

**IB01. "IBM z16: Reasons to upgrade from your z15 or z14"**
- **Author, org, year:** Edward L. Linde II, IBM, 2022.
- **URL:** https://ibm.com/downloads/cas/M65ZR5DV (fetched). Also reported by Network World, https://www.networkworld.com/article/3655975/ibm-z16-a-mainframe-designed-for-ai-hybrid-cloud-security-and-open-source.html, and IBM REDP-5712, https://www.redbooks.ibm.com/redpapers/pdfs/redp5712.pdf (both not fetched; from search snippet; not verified).
- **Compared:**
  - z16 side: a z/OS 2.4 LPAR with 8 CPs, 16 zIIPs and 8 GB, running WMLz (OSCE) on zCX.
  - x86 side: TensorFlow Serving 2.4 on Ubuntu 20.04.3, 8 Skylake Xeon Gold cores at 2.30 GHz with HT, 1.5 TB of memory, RAID5 SSD.
  - Network: a Linux on Z LPAR on the same z16 bridged the network, and "Additional network latency was introduced with the Linux 'tc-netem' command to simulate a remote cloud environment with 60ms average latency."
- **Workload:** a CICS OLTP credit-card workload with in-transaction fraud detection, using github.com/IBM/ai-on-z-fraud-detection.
- **Headline:** "20x lower response time and up to 19x higher throughput when co-locating applications and inferencing versus sending the same inferencing operations to a compared x86 cloud server with 60ms avg. latency."
- **Other footnotes in the same document:**
  - fn2, 300 billion inferences/day: extrapolated from 48 IFLs, batch 128, 8 threads pinned one per chip.
  - fn11, "3.6x fewer cores": OpenShift 4.9 on 8 IFLs against 32 Cascade Lake cores. Both sides call PostgreSQL in a z16 LPAR, so the x86 side is remote from the database.
  - fn14, 25 billion encrypted CICS-VSAM tx/day: extrapolated from 2 LPARs of 6 CPs each.
  - fn15, 20 billion HTTPS tx/day: Acme Air on 24 IFLs.
  - fn13: single-thread +11% vs z15.
  - fn10: "$16,250" a year floor-space and power saving, projected from z14 data assuming 10% improvement, against Haswell/Broadwell-era x86 at surveyed utilisations, $0.10/kWh and PUE 1.67.
- **Quality / independence:** vendor test. The 20x is dominated by the injected 60 ms delay, so it measures placement, not compute. The per-day figures are extrapolations.
- **Maps to:** AI inference, batch+online mix, crypto, cost/energy.
- **Useful to us:**
  - The z16 side is a configuration we can mirror: z/OS, zCX, a CICS caller and the public model.
  - Per-engine ceilings from the extrapolated figures (our arithmetic): about 24k encrypted CICS-VSAM tx/s per CP (fn14), and about 9.6k HTTPS tx/s per IFL (fn15).
- **Also in:** benchmarks_community item 14.

**IB02. IBM z17 product page**
- **Author, org, year:** IBM, 2025.
- **URL:** https://www.ibm.com/products/z17 (fetched).
- **Compared:** z17 only. LPAR 1 ran RHEL 9.4 on 6 IFLs (SMT) with 128 GB. LPAR 2 had 2 CPs and 4 zIIPs, 256 GB, and z/OS 3.1 with zCX.
- **Workload:** a single thread doing local inference on the synthetic LSTM credit-card fraud detection model, batch 160.
- **Headline:** "Process up to 5 million inference operations per second with less than 1 ms response time". The footnote says "extrapolated from IBM internal tests" on machine type 9175.
- **Quality / independence:** vendor extrapolation from one thread.
- **Maps to:** AI inference.
- **Useful to us:** batch 160 is far from our batch 1, so this headline is not the expected value for a per-transaction test.
- **Also in:** vendors C14.

**IB03. "AI on IBM Z" page**
- **Author, org, year:** IBM, 2025.
- **URL:** https://ibm.com/z/artificial-intelligence (fetched).
- **Also covered:**
  - The zDLC 5.0.0 blog (Sunny Anand, 2025-06-10): "up to 48% reduction in latency for single-threaded inference" against z16. https://community.ibm.com/community/user/blogs/sunny-anand/2025/06/10/ibm-z-deep-learning-compiler-500 (not fetched; from search snippet; not verified).
  - A Software AG reprint of "5.6x fewer cores" against 28 Sapphire Rapids cores, https://www.softwareag.com/en/?p=10347 (not fetched; from search snippet; not verified).
- **Compared:** z17 against z16. Both had 2 GCPs and 4 zIIPs with SMT, 256 GB, z/OS V3R1 and IBM Z Deep Learning Compiler 4.3.0.
- **Workload:** the synthetic CCFD model.
- **Headline:**
  - "boost throughput up to 7.5x over IBM z16", with "8 parallel threads with batch size of 1".
  - 450 billion inferences/day at 1 ms, with the same footnote as IB02.
- **Quality / independence:** vendor, Z against Z.
  - The IBM strand's arithmetic: 5M/s x 86,400 = 432 billion/day, which does not match "450 billion".
- **Maps to:** AI inference.
- **Useful to us:** this is the only IBM test found at **batch 1 on a 2-GCP z/OS LPAR**, the closest IBM configuration to our slice. It gives no absolute latency, though.
- **Also in:** vendors C14.

**IB04. "AI Toolkit for IBM Z and LinuxONE" page**
- **Author, org, year:** IBM, 2025-26.
- **URL:** https://www.ibm.com/products/ai-toolkit-for-z-and-linuxone (fetched).
- **Also covered:** the same 450B test is described in Joy Deng, "z17 built for AI at scale with Telum II", IBM Community, 18 Jun 2025, https://community.ibm.com/community/user/blogs/joy-deng/2025/06/18/z17-built-for-ai-at-scale-with-telumii (fetched).
- **Compared, by footnote:**
  - **fn1:** z17 9175 with Ubuntu 24.04, 7 dedicated IFLs (SMT), 256 GB and FlashSystem 9500. The x86 side had 28 Emerald Rapids Xeon Gold cores at 2.20 GHz with HT and 1 TB, running Triton with OpenVINO on CPU (AMX).
  - **fn3:** 45 IFLs SMT and 128 GB, with 64 threads pinned 8 per chip (1 zDLC and 7 PyTorch).
  - **fn4:** z16 (3931) with 6 IFLs and 256 GB on Ubuntu 22.04, against 6 Ice Lake Xeon Gold cores at 2.80 GHz with HT and 1 TB.
- **Workload:**
  - fn1: an LSTM + TabFormer ensemble CCFD model driven by JMeter 5.6.3 with 64 users.
  - fn3: LSTM at batch 160, with TabFormer on 0.035% of requests.
  - fn4: a Snap ML 1.12.0 Random Forest on Triton on both sides, using the Kaggle Santander data and the "hey" load tool.
- **Headline:**
  - fn1: one Integrated Accelerator for AI is about "a 13-core x86 server".
  - fn3: 450 billion/day with "99.9 percentile response under 1 ms" (extrapolated).
  - fn4: Snap ML "up to 3.5x faster" than NVIDIA FIL on x86.
- **Quality / independence:** vendor. The x86 side is current hardware with AMX, which is better than most IBM comparisons.
  - Contradiction: this page backs "450 billion" with 45 IFLs, 64 threads and an ensemble model. IB02 and IB03 back the same headline with 1 thread and batch 160.
- **Maps to:** AI inference, mixed grid.
- **Useful to us:**
  - fn4 is the only **core-matched** IBM inference comparison (6 vs 6).
  - fn1's unit, "x86 cores equivalent to one accelerator", is worth borrowing for the AI grid.
- **Also in:** benchmarks_community item 13; the analysts strand's C04 (Cambrian-AI, AN02) uses the fn1 test.

**IB05. "IBM z16 Integrated Accelerator for AI" (Washington Systems Center deck)**
- **Author, org, year:** IBM WSC, 2023.
- **URL:** https://community.ibm.com/zsystems/uploads/document/slider/12qs4awwowtp.pdf (fetched).
- **Compared:** none. It describes the architecture.
- **Workload:** none.
- **Headline:**
  - Over 6 TFLOPS FP16 per chip, over 200 TFLOPS on 32 chips, and over 200 GB/s to and from cache.
  - Snap ML Random Forest, Extra Trees and GBM are accelerated.
- **Quality / independence:** vendor reference.
- **Maps to:** AI inference, choke/limits.
- **Useful to us:** on z16 the accelerator is one per chip, shared by all cores on it. On z17 any core can use any of the 8 accelerators in its drawer. Either way, a 2-CPU slice of a shared LPAR does not own an accelerator, and other tenants' inference work can contend with ours.

**IB06. "Taking a Peek Under the Hood of Compression on z15"**
- **Author, org, year:** Joseph Gentile and Anna Shugol, IBM Poughkeepsie z/OS Performance, SHARE blog, 2019-10-28.
- **URL:** https://blog.share.org/Article/taking-a-peek-under-the-hood-of-compression-on-z15 (fetched).
- **Compared:** z15 on-chip zEDC against a z14 with zEDC Express cards.
- **Workload:** DFSMS, Java Deflater and GZIPOutputStream, and z/TPF.
- **Headline:**
  - "up to 17 times the total compression throughput of a z14 configured with the maximum number of zEDC Express cards".
  - DFSMS: "up to 8x application elapsed time speedup vs. the zEDC Express card, with no additional CPU time".
  - Java Deflater: up to 15x (Java 8 SR6 on z15 against SR5 FP25 on z14 with the card).
  - GZIPOutputStream: up to 2x.
  - z/TPF software compression "used up to 280x the CPU".
- **Quality / independence:** vendor, Z against Z. All figures are "up to", and no configurations are given.
- **Maps to:** compression.
- **Useful to us:** if our Java runs on Semeru/IBM Java on z/OS, Deflater should reach the accelerator but GZIPOutputStream gains much less. State which API was used.
- **Also in:** academic supporting sources; benchmarks_community section 2.

**IB07. "Linux on IBM Z and LinuxONE: What's New" (GSE UK 2021, session 2AV)**
- **Author, org, year:** Stefan Raspl, IBM Germany R&D, November 2021.
- **URL:** https://conferences.gs-uk.org/2021/presentations/2AV.pdf (fetched).
- **Also covered** (all from search snippets, not fetched; not verified):
  - IBM Docs, https://www.ibm.com/docs/en/linuxonibm/lddd/lddd_r_zedc.html and https://www.ibm.com/docs/en/linuxonibm/lddd/lddd_c_zedc_know.html
  - A SHARE session PDF, https://linklings.s3.amazonaws.com/organizations/SHARE/share26/submissions/stype112/5CRWX-IBM_copy214s2.pdf (HTTP 403).
  - These repeat "up to 42x" and say output is "roughly equivalent to gzip level 1" against gzip's default of 6.
- **Compared:** `minigzip -1` with software zlib against `minigzip -1` with the Integrated Accelerator for zEDC, on a z15 with **4 IFLs**.
- **Workload:** compression time on public corpus files.
- **Headline:** E.coli 33.9x, bible.txt 30.3x, world192.txt 24x, Canterbury.tar 42x.
  - The bar-to-label mapping is as printed in the extracted slide.
  - The slides also note that it is enabled at the default level from RHEL 8.4, SLES 15 SP3 and Ubuntu 20.10, that `DFLTCC_LEVEL_MASK` turns on other levels, and that the /proc/cpuinfo flag is `dflt`.
- **Quality / independence:** vendor, but the method is reproducible: public corpora and a public tool, hardware against software on the same Z.
- **Maps to:** compression.
- **Useful to us:** the closest analogue to our DFLTCC against software zlib test; rerun these exact files.
  - Gap: no decompression figures, no compression ratios, small files, and no x86 side.
- **Also in:** vendors C17.

**IB08. zlib PR #410 and the Linux kernel DFLTCC series**
- **Author, org, year:** Ilya Leoshkevich (zlib PR, 15 Mar 2019) and Mikhail Zaslonko with Leoshkevich (kernel series, 3 Jan 2020), IBM.
- **URL:** https://github.com/madler/zlib/pull/410 and https://lwn.net/Articles/808809/ (fetched by the benchmarks strand; the IBM and academic strands saw the LWN page only as a search snippet).
- **Compared:** z15 hardware against z15 software at zlib level 1, in IBM internal tests on a native z15 LPAR. The method was `dd` on btrfs with a ramdisk, using a Fedora 30-based driver.
- **Workload:** deflate and inflate on four corpora.
- **Headline:**

  | Corpus | Deflate rate | Inflate rate | Compression ratio vs software |
  | --- | --- | --- | --- |
  | Zeroes | 1.46x | 1.02x | 1.00 |
  | Random ASCII | 10.44x | 3.00x | 0.96 |
  | Dickens | 6.21x | 3.33x | 0.94 |
  | vmlinux | 8.37x | 3.90x | 1.02 |

  - The summary claim is "up to 10 times faster compression (on level 1) and up to 4 times faster decompression".
- **Quality / independence:** IBM internal tests, but the method is stated and the code is open source.
- **Maps to:** compression.
- **Useful to us:**
  - The four corpora (zeroes, random ASCII, Dickens, vmlinux) are reusable.
  - The table shows that speedup depends on the data: zeroes are the worst case.
  - It contradicts IB07 (24-42x) and the zlib-ng figure in CM05 ("110x"). Buffer size, level and corpus explain the spread.
- **Also in:** ibm_material B3; benchmarks_community item 5.

**IB09. "HOWTO: Exploiting Hardware Compression in zlib with IBM z15"**
- **Author, org, year:** Stefan Raspl, IBM, 16 Oct 2019.
- **URL:** https://linux-on-z.blogspot.com/2019/10/howto-exploiting-hardware-compression.html (fetched).
- **Compared:** z15 hardware against software.
- **Workload:** a Python gzip level-6 one-liner over about 1.3 GB of repeated alphabet.
- **Headline:**
  - The time "should drop to about a fifth".
  - Python's small buffers keep the hardware below top speed.
  - `-DDFLTCC_LEVEL_MASK=0x7e` enables levels 1-6.
- **Quality / independence:** IBM how-to, with a reproducible one-liner.
- **Maps to:** compression.
- **Useful to us:** the level-mask trick, and the warning that buffer size matters.

**IB10. Launchpad bug LP#1884514 (zlib/gzip hardware compression enablement)**
- **Author, org, year:** IBM via bugproxy, filed 22 Jun 2020.
- **URL:** https://bugs.launchpad.net/ubuntu-z-systems/+bug/1884514 (fetched).
- **Compared:** a z15 system with 4 IFLs.
- **Workload:** not specified.
- **Headline:** "a speed up of more than 40x", described as the DEFLATE best case.
- **Quality / independence:** vendor; no method given.
- **Maps to:** compression.
- **Useful to us:** an upper-bound claim to set our results against.

**IB11. "Compression and Pervasive Encryption: z15 Offers the Best of Both Worlds"**
- **Author, org, year:** IBM IT Economics Consulting & Research, September 2019.
- **URL:** https://ibm.com/downloads/cas/AM1PYZBB (fetched).
- **Compared:** z14 and z15 with **2 CPs** each (footnote 3).
- **Workload:** a batch of 18 I/O-intensive utility jobs copying large amounts of data through BSAM and QSAM sequential files, with compression, encryption, both, or neither.
- **Headline:**
  - With on-chip zEDC plus BSAM/QSAM encryption on z15, elapsed time improved "by up to 72% while reducing CPU by up to 7% compared to not using compression and encryption".
  - z14 pervasive-encryption overhead averaged about 2.6% MIPS, from "measuring the MIPS overhead for 23 different clients".
  - The zEDC compression ratio averaged about 6:1 to 7:1 on z14; early z15 measurements showed about 8.5:1 on SMF data.
- **Quality / independence:** vendor. The 2.6% is a field average, which is unusual and useful.
- **Maps to:** compression, crypto, RMW/IO, cost/energy.
- **Useful to us:** the only IBM measurement found at our slice size of 2 CPs. Its design is worth borrowing: compress before encrypting, and measure elapsed time and CPU against a run with neither.

**IB12. "Peeking Under the Hood of Sort Acceleration on z15"**
- **Author, org, year:** Joseph Gentile and Anna Shugol, IBM, SHARE blog, 2020-12-16.
- **URL:** https://blog.share.org/Article/peeking-under-the-hood-of-sort-acceleration-on-z15 (fetched).
- **Also covered:** IBM Support's DFSORT page, https://www.ibm.com/support/pages/dfsort, and User Guide PH03207 (not fetched; from search snippet; not verified). They repeat the figures and name ICE267I as the message that reports whether Z Sort was used.
- **Compared:** dedicated z15 against dedicated z14.
  - The Db2 REORG test used LPARs with 4 GCPs, 1 zIIP and 512 GB.
- **Workload:**
  - In-memory DFSORT using 64-bit memory objects, averaged over LRECL 80 B to 27 KB, keys of 8-500 B, 100K to 100M records, FB and VB, large and extended-format data sets.
  - Db2 12 REORG with SHRLEVEL CHANGE: 20 partitions, 100M rows averaging 243 B.
- **Headline:**
  - Batch sort elapsed time is "up to 20-30% depending on record size" lower (more benefit for small records), and CPU up to 40% lower.
  - REORG is up to 12% lower in elapsed time and 19% lower in CPU, using up to 2x the memory.
  - Prerequisites are BSAM and zHPF. Incompatible statements fall back silently.
  - SORTL sorts up to 128 input lists and suspends on interrupts.
- **Quality / independence:** vendor, Z against the previous Z. A search summary attributed 12%/19% to "zBNA estimates"; the article ties them to REORG.
- **Maps to:** sort.
- **Useful to us:**
  - The baseline is z14 software sort, not Z Sort switched off on the same z15. Our ZSORT on/off contrast on z17 is cleaner.
  - Borrow the record-size, key-length and record-count grid.
  - The gain shrinks for long records, where I/O dominates.
- **Also in:** academic supporting sources; benchmarks_community item 4.

**IB13. "IBM z15 Performance of Cryptographic Operations (CPACF, CEX7S)"**
- **Author, org, year:** IBM Marketing Communications, Server Group, May 2020.
- **URL:** https://ibm.com/downloads/cas/6K2653EJ (fetched; text extraction scrambled some table columns).
- **Compared:** a z15 8561-770 or -703, a single LPAR, mostly with 4 dedicated CPs.
  - Figures are given for 1 CPACF, and "In all cases the throughput with 4 CPACFs was four times the throughput of 1 CPACF."
  - The test cases are written in assembler and issue MSA instructions directly.
- **Workload:** clear-key AES, PKCS#11 elliptic-curve operations, and System SSL handshakes.
- **Headline:**
  - KMC AES-128-CBC: 17.7M ops/s at 64 B (about 1.13 GB/s), rising to about 5.1 GB/s from 4 KB to 1 MB.
  - AES-256-CBC: about 1.04 GB/s at 64 B and about 3.7 GB/s at large sizes.
  - Sign, in ops/s: P-256 48,660; P-384 25,804; P-521 18,156; Ed25519 25,389; Ed448 16,278.
  - Verify, in ops/s: P-256 19,365; Ed25519 18,877.
  - P-256 key generation: 22,122/s.
  - System SSL on z/OS 2.4 with 4 CPs, full RSA-2048 handshakes: 255/s in software against 10,847/s with one CEX7C (42.5x), with CP utilisation down 55%. With session IDs cached: 38,098/s.
- **Quality / independence:** vendor, but unusually complete: it sweeps data lengths and states how throughput scales. It has no x86 figures.
- **Maps to:** crypto.
- **Useful to us:**
  - About 48.7k P-256 signatures/s per CP on z15 is the reference for "signatures per payment". Our ICSF, libica or Java API path may land higher or lower.
  - Verify is about 2.5x slower than sign, so the payment test must say which operation it times.
- **Contradiction (encryption overhead):** IBM's own figures vary: 2.6% field average (IB11), 0.49% for CICS/Db2 (IB14), and about +7.6% for encrypted IMS VSAM (7.47 against 6.94). The IMS figure comes from https://community.ibm.com/community/user/ibmz-and-linuxone/blogs/paul-gandolfo/2023/03/07/osamldsencryption (not fetched; from search snippet; not verified).

**IB14. "Ready for the Cloud with IBM z/OS Container Extensions"**
- **Author, org, year:** IBM IT Economics, February 2021.
- **URL:** https://ibm.com/downloads/cas/ZEM93ORZ (fetched).
- **Compared:** Db2 v12 on a z15 T01 in every case.
  - Monolithic test: zCX co-located with Db2 in an LPAR with 2 CPs, 6 zIIPs and 128 GB, against a 32-core Haswell x86 server with 512 GB running 2 Docker containers that reach Db2 on z/OS over SSL.
  - Microservices test: 3 CPs and 12 zIIPs on the Z side.
- **Workload:** JMeter driving a 50,000-user banking simulation.
  - Footnote 11: CICS 5.4 + Db2 12 in an LPAR with 8 CPs, 1 zIIP and 32 GB, driven by Rational Performance Tester with 50,000 users, with DFSMS encryption plus compression on against off.
- **Headline:**
  - Monolithic: "44% more" transactions per second per core (fn9). The fn13 TCO model is set to 3,572 against 1,996 TPS per core.
  - Microservices: "54% more" (fn10). The fn12 TCO model is set to 1,515 against 702 TPS per core.
  - 3-year TCO 43% and 69% lower.
  - fn11: "10,108 TPS per utilized processor" unencrypted against 10,059 encrypted (0.49%).
- **Quality / independence:** vendor.
  - The x86 side pays a network and SSL hop to the database, its hardware is old (Haswell), and the TCO assumptions favour Z: x86 dev/test at 100% of production against 25% on Z.
  - The per-core TPS figures are the TCO models' settings ("configured to run an average of", fn12-13), which give 3,572/1,996 = 1.79x and 1,515/702 = 2.16x for Z. The measured-results footnotes (fn9-10) give no per-core numbers, but the body text says the microservices scenario "delivered on average two times more transactions per core" (Figure 3), in line with 2.16x. The headlines' 44% and 54% match 1 - 1,996/3,572 and 1 - 702/1,515 (our arithmetic): the x86 shortfall, stated as "more" for Z, which understates the ratio.
- **Maps to:** batch+online mix (online), crypto, cost/energy.
- **Useful to us:**
  - "TPS per core" is the unit of our mixed grid.
  - About 10k CICS/Db2 TPS per processor is a per-engine ceiling to set our CICS-like rate against.

**IB15. IBM LinuxONE 5 page**
- **Author, org, year:** IBM, 2025.
- **URL:** https://www.ibm.com/products/linuxone-5 (fetched; re-checked 2026-11-25).
- **Compared:** one 9175 Max136 (136 configurable PUs) against 23 x86 servers, each with "two 5th gen Intel Xeon Platinum 8592+ processors and 64 cores per CPU". Both sides ran RHEL 9.5 with KVM.
- **Workload:** WebSphere Liberty v25 + EDB Postgres for Kubernetes v1.25 on OCP 4.17, "extrapolated to a complete customer IT solution".
- **Headline:** one Max136 against 23 compared x86 servers (fn1), and up to 76% lower 5-year TCO against x86 "three-year-old and older" (fn2). The "up to 2,944 cores" figure is on IBM's Sustainability page (IB16, fn1), not on this page.
- **Quality / independence:** a vendor estate model.
  - Internal contradiction: one disclaimer says "Intel Xeon Gen 2 and older", which conflicts with the 8592+ in fn1.
  - These are the Signal65 (AN01) numbers.
- **Maps to:** cost/energy.
- **Useful to us:** a worked example of the estate-model trap; see AN01 for how it decomposes.
- **Also in:** vendors C14.

**IB16. "Sustainability with IBM Z" page**
- **Author, org, year:** IBM, 2025-26.
- **URL:** https://www.ibm.com/products/z/sustainability (fetched).
- **Compared, by footnote:**
  - **fn1/fn2:** one z17 9175 MAX 136 against 23 servers of 2x Xeon 8592+, the same configuration as IB15. Carbon is from Electricity Maps for New York.
  - **fn3:** against **72** "two-year-old" servers of 2x 28-core Xeon Gold 5420+, running the MegaCard benchmark on Liberty v24 and RHOCP 4.17 with EDB Postgres.
  - **fn4:** a 9176-ME2 Max 82 with 16 AI units against "technologies commonly used at the end of 2025". No server count is given.
- **Workload:** containerised OLTP, "simulating core online banking", and AI-infused OLTP.
- **Headline:**
  - "up to 2,944 cores".
  - 65% less energy and 109 t CO2e a year.
  - Up to 83% less power for AI-OLTP.
  - Up to 65% lower 5-year TCO, with 93% lower software costs, 92% less power and 75% less space.
- **Quality / independence:** vendor. The 23-server and 72-server figures come from different scenarios, not one error. The page gives no utilisation, test duration or power-measurement method.
- **Maps to:** cost/energy, AI inference, mixed grid.
- **Useful to us:** MegaCard (github.com/IBM/megacard-standalone, per AN02) is a public OLTP app that could seed our online test.
- **Also in:** analysts C02.

**IB17. IBM LinuxONE Emperor 4 / Rockhopper 4 page**
- **Author, org, year:** IBM, 2023.
- **URL:** https://www.ibm.com/uk-en/products/linuxone-emperor-4 (fetched).
- **Also covered** (all not fetched; from search snippet; not verified):
  - Network World, https://www.networkworld.com/article/3673296/ibm-launches-fourth-gen-linuxone-servers.html
  - The Malaysian IBM page, https://www.ibm.com/my-en/products/linuxone-emperor-4
  - NAND Research, https://nand-research.com/tag/linuxone/
- **Linux on IBM Z FAQ,** https://ibm.com/downloads/cas/O8W6EJ0P: the May 2025 edition served there (ZSQ03011-USEN-51, fetched 2026-11-25) does not contain the consolidation figures a search snippet had attributed to it, so none is cited from it.
- **Compared:** one 3932 Max 68 (68 IFLs, 7 TB) against **36** two-socket Skylake Xeon Gold servers with 40 cores each, 1,440 cores in total.
- **Workload:** consolidation.
- **Headline:** 75% less energy. Press adds 50% less floor space (snippet only; not verified).
- **Quality / independence:** a vendor model with **no measured wall power on either side**.
  - x86 power uses February 2023 IDC QPI values "reduced to 55%" on the basis of field observation, with PUE 1.55.
  - Z power comes from the IBM Power Estimator.
  - IBM pages attach the 75% to different models.
- **Maps to:** cost/energy.
- **Useful to us:** if we measure the Ryzen at the wall, the result is not comparable unless the z17 side is modelled the same way.
- **Also in:** benchmarks_community items 22 and section 4.

**IB18. "MongoDB on IBM Z Performance Tuning Guide" v1.0**
- **Author, org, year:** Marc Beyerle, IBM, 2026-05-18.
- **URL:** https://www.ibm.com/docs/en/linuxonibm/pdf/lnz_MongoDB_on_IBM_Z_Tuning_Guide_V1_0.pdf (fetched).
- **Compared:** z17 tuning steps only.
  - A leader LPAR with 24 IFLs SMT and 128 GiB, plus 2 followers with 4 IFLs each, on FlashSystem 9200/9500 storage.
  - The Ice Lake Xeon Gold 6326 box is the YCSB **load driver**, not a comparison target. A search snippet had implied it was.
- **Workload:** RHEL 10, MongoDB Enterprise 8.2.5, YCSB 0.17.0, read-mostly.
- **Headline:**
  - WiredTiger cache at 75% of memory (the chart shows +70%).
  - Dirty-bytes tuning gave about +5% throughput and about -22% p95 update latency.
  - "None of the AI tools that I tried was able to give recommendations that had a measurable impact."
- **Quality / independence:** vendor, with transparent disclosure of the tuning.
- **Maps to:** RMW/IO, choke/limits.
- **Useful to us:** untuned runs understate capacity on either platform, so state the tuning on both sides.

**IB19. IBM z16 Proof Points decks**
- **Author, org, year:** Dominic Roehm (2022 Client Workshop EMEA), plus the 2023-04-26 webcast decks with E. Pasch.
- **URLs:**
  - https://community.ibm.com/community/user/ibmz-and-linuxone/viewdocument/2022-06-30-ibm-z16-proof-points?CommunityKey=c1293167-6d93-448e-8854-3068846d3dfe&tab=librarydocuments
  - https://community.ibm.com/community/user/ibmz-and-linuxone/viewdocument/2023-04-26-ibm-z16-linuxone-4-ha?CommunityKey=c1293167-6d93-448e-8854-3068846d3dfe&tab=librarydocuments
  - The landing pages were fetched. The PDFs sit behind a click-through and an S3 redirect and were **not fetched**.
- **Related:** the SHARE "Linux on IBM z16 and LinuxONE Emperor 4 Performance Update" decks (IBM Böblingen), https://www.share.org/Events/Past-Events/Proceedings/Proceeding-Details/35798-linux-on-ibm-z16-and-linuxone-emperor-4-performance-update (abstract pages only).
- **Compared:** unknown. Probably Linux x86 comparisons (pgbench, MongoDB, Java).
- **Workload:** unknown.
- **Headline:** unread.
- **Quality / independence:** vendor.
- **Maps to:** RMW/IO, cost/energy.
- **Useful to us:** these are the decks IBM's own performance team cites for x86 comparisons. They need a manual download.

**IB20. LSPR FAQ, plus the CPU MF Formulas and LSPR SC28-1187**
- **Author, org, year:** IBM Support.
- **URLs:**
  - https://www.ibm.com/support/pages/lspr-faq (fetched).
  - https://www.ibm.com/support/pages/system/files/inline-files/CPU%20MF%20Formulas%20April%202025_0.pdf (not fetched; from search snippet; not verified).
  - https://www.ibm.com/support/pages/system/files/inline-files/SC28118726.pdf (not fetched; from search snippet; not verified).
- **Compared:** IBM Z generations only.
- **Workload:** the LSPR workload categories.
- **Headline:**
  - "The LSPR ratios reflect the range of performance between IBM Z servers".
  - "The multi-image table is used as the basis for setting MIPS and MSUs".
  - SC28-1187 says IBM provides "capacity data in terms of work accomplished, or throughput".
  - The FAQ says nothing about x86.
- **Quality / independence:** vendor method.
- **Maps to:** choke/limits, and every category as a method.
- **Useful to us:** the LSPR is Z-only by its own statement, so there is no official MIPS-to-x86 equivalence. Its unit, work per CPU-busy time, is the one to borrow.

**IB21. "z16 Performance Considerations and Best Practices" (IBM Z Tech Bytes)**
- **Author, org, year:** John Burg, IBM WSC, 2023-12-07.
- **URL:** https://www.ibm.com/support/pages/system/files/inline-files/Tech%20Bytes%20z16%20Performance%20Considerations%20and%20Best%20Practices.pdf (fetched).
- **Compared:** Z generations. The largest z16 is 183,267 PCI, +17% over z15.
- **Workload:** none.
- **Headline:**
  - "Substantial frequency gains from generation to generation are no more".
  - SMF 30 carries 150+ crypto entry IDs (CPACF counters) and AIU counters 1-21, one per NNPA function.
- **Quality / independence:** vendor.
- **Maps to:** crypto, AI inference, as a method.
- **Useful to us:** the SMF 30 CPACF and NNPA counters can **prove** that a z/OS job used CPACF or the AIU, alongside ICE267I for sort.

**IB22. IBM REDP-5170, "IBM CICS Performance Series: Effective Monitoring for CICS Performance Benchmarks"**
- **Author, org, year:** J. Burgess and M. Mandelli, IBM, 2015.
- **URL:** https://www.redbooks.ibm.com/redpapers/pdfs/redp5170.pdf (fetched and text-extracted by the academic strand; the IBM strand saw it as a snippet).
- **Also covered:** SHARE session 8916, "Optimizing your Batch Window", https://share.confex.com/share/116/webprogram/Handout/Session8916/S8916%20Optimizing%20your%20Batch%20Window.pdf (not fetched; from search snippet; not verified). It is WLM guidance without footnoted numbers.
- **Compared:** the same zEC12 LPAR with and without "other batch-type work in separate address spaces".
- **Workload:** a CICS benchmark.
- **Headline:** CICS CPU per transaction rose by "up to 15% more CPU", through cache contention (higher CPI). This holds even with dedicated CPs, because the CPs share L3 per chip and L4 per book.
- **Quality / independence:** vendor; not peer-reviewed.
- **Maps to:** batch+online mix, choke/limits.
- **Useful to us:**
  - The reference value for our CICS-under-sort test.
  - Report both response time and CPU per transaction, because WLM can hold the first while the second inflates.
  - Also report run-to-run variance on the shared LPAR.

**IB23. IBM REDP-5461, "Scale up for Linux on IBM Z"**
- **Author, org, year:** IBM.
- **URL:** https://www.redbooks.ibm.com/redpapers/pdfs/redp5461.pdf (fetched 2026-11-25).
- **Compared:** LinuxONE Emperor II against "x86 alternatives". No test configuration is given.
- **Workload:** Java applications (the speed claim); a general statement on utilisation.
- **Headline:**
  - Emperor II "delivers industry-leading Java performance that runs applications up to 50% faster than on x86 alternatives".
  - x86 "typically runs at 40-60% utilization", against Z at "over 90% for sustained periods of time".
- **Quality / independence:** vendor.
- **Maps to:** choke/limits, cost/energy.
- **Useful to us:** the 90% against 40-60% figure is an assumption, and our choke tests could test it.

**IB24. IBM REDP-5032, "Understanding the Value of Your Mainframe"**
- **Author, org, year:** IBM.
- **URL:** https://www.redbooks.ibm.com/redpapers/pdfs/redp5032.pdf (fetched 2026-11-25); also https://public.dhe.ibm.com/software/uk/pdf/zEnterprise/redp5032.pdf (not fetched).
- **Compared:** CICS/COBOL against virtualised x86.
- **Workload:** core banking.
- **Headline:**
  - The CICS/COBOL platform "required 4.6 times fewer cycles per user interaction".
  - It also cites a migration that needed six 8-core x86 servers where one was assumed.
- **Quality / independence:** vendor.
- **Maps to:** batch+online mix, cost/energy.
- **Useful to us:** IBM's own core-banking cycles ratio (4.6x). It is the opposite of zRef's conclusion (VN01).

**IB25. Linux Journal "Geek Guide: IBM LinuxONE" and article (relaying IBM claims)**
- **Author, org, year:** Linux Journal, about 2015-2018.
- **URLs:** https://www.linuxjournal.com/sites/default/files/2018-11/GeekGuide-IBM-LinuxONE.pdf ; https://www.linuxjournal.com/content/ibm-linuxone-provides-new-options-linux-deployment (the Geek Guide PDF fetched 2026-11-25; the article not fetched).
- **Compared:** z13-era LinuxONE against x86, "when the systems each utilize four cores".
- **Workload:** Node.js web transactions; MongoDB 3.0.4.
- **Headline:** "2.1 times as many Node.js Web transactions as x86"; 1.9-2.1x on MongoDB.
- **Quality / independence:** IBM figures relayed by the press; about ten years old.
- **Maps to:** batch+online mix, RMW/IO.
- **Useful to us:** the only equal-core per-core claim found outside AI. It is an order-of-magnitude reference: about 2x per core, not 10-30x.

**IB26. "Accelerating big integer crypto" (Go crypto/big on z16)**
- **Author, org, year:** Kiran M. Vijay, IBM Community, 2026-04-20.
- **URL:** https://community.ibm.com/community/user/blogs/kiran-m-vijay/2026/04/20/accelerating-big-integer-crypto (not fetched; from search snippet; not verified).
- **Compared:** new vectorised Montgomery multiplication against older s390x Go code.
- **Workload:** big-integer crypto.
- **Headline:** "up to 62%", merged in Go 1.26.
- **Quality / independence:** IBM engineer.
- **Maps to:** crypto.
- **Useful to us:** the Go version matters for any Go crypto run on s390x. State it.

**IB27. "Mainframe TCO" (IBM Think)**
- **Author, org, year:** Elpida Tzortzatos, IBM.
- **URL:** https://www.ibm.com/think/insights/mainframe-tco (fetched).
- **Compared:** z16 against "a comparable x86 server".
- **Workload:** inference.
- **Headline:** z16 can "reduce energy consumption for running inference operations by 41x".
- **Quality / independence:** vendor; no footnote.
- **Maps to:** AI inference, cost/energy.
- **Useful to us:** an example of an energy ratio with no stated basis.

**IB28. IBM claim that System z can "save up to 60% compared to the cloud" (TechRadar)**
- **Author, org, year:** TechRadar, 2015-04-01, quoting IBM's Steven Dickens.
- **URL:** https://techradar.com/news/internet/cloud-services/running-apps-in-the-cloud-more-than-twice-as-expensive-as-mainframes-1289929 (fetched).
- **Compared:** System z against cloud and x86.
- **Workload:** none stated.
- **Headline:** "save up to 60% compared to the cloud" and "nearly a third" against x86.
- **Quality / independence:** vendor claim in the press; no method and no study named.
- **Maps to:** cost/energy.
- **Useful to us:** context only.

**IB29. IBM z10 launch claim, "equivalent to 1,500 x86 servers"**
- **Author, org, year:** The Register (Kelly Fiveash), 2008-02-26.
- **URL:** https://www.theregister.com/2008/02/26/ibm_z10_mainframe/ (fetched).
- **Compared:** z10 against 1,500 x86 servers.
- **Workload:** consolidation.
- **Headline:** the z10 "is equivalent to 1,500 x86 servers" and could "push down energy costs by up to 85 per cent", with "prices for the box do start at $1m".
- **Quality / independence:** vendor claim; the article gives no basis.
- **Maps to:** cost/energy.
- **Useful to us:** the claim behind the "Strange math" critique (CR02).

**IB30. "Transforming the Economics of IT" (z13-era TCO deck)**
- **Author, org, year:** Craig Bender, IBM.
- **URL:** https://public.dhe.ibm.com/software/os/systemz/pdf/12-Craig_Bender_-_Transforming_the_Economics_of_IT.pdf (not fetched; from search snippet; not verified).
- **Compared:** a mainframe private cloud against an x86 private cloud and public cloud.
- **Workload:** none measured.
- **Headline:** about 1/3 lower 3-year cost than x86 private cloud, and about 2/3 lower than public cloud, using estimated equivalence ratios.
- **Quality / independence:** vendor model.
- **Maps to:** cost/energy.
- **Useful to us:** context only.

**IB31. "OpenSSL on IBM Z" documentation, the OPENSSL_s390xcap man page, and the LKML s390/aes post**
- **Author, org, year:** IBM Docs and OpenSSL docs, 2026.
- **URLs:**
  - https://www.ibm.com/docs/en/linuxonibm/lxos/lxos_osl_lonz_support.html
  - https://docs.openssl.org/3.3/man3/OPENSSL_s390xcap/
  - https://lkml.iu.edu/2601.1/11712.html
  - All three not fetched; from search snippet; not verified.
- **Compared:** CPACF against software on the same box.
- **Workload:** AES-GCM through KMA (MSA 8, from z14); single-block AES.
- **Headline:**
  - `OPENSSL_s390xcap` can mask CPACF capabilities, with tokens up to z16/z17.
  - The LKML post says single-block CPACF AES is "up to ~2x faster" than software.
- **Quality / independence:** vendor and project docs.
- **Maps to:** crypto.
- **Useful to us:** masking CPACF with `OPENSSL_s390xcap` gives a same-box software baseline. That separates "CPACF against software" from "z17 against Ryzen". No published `openssl speed` pairing of s390x against AES-NI was found.


### 3.3 Vendors, ISVs, cloud providers and rehosters (20 items)

**VN01. zRef: HP's "z/OS Reference Application" and the 2013 "Mainframe Alternative Sizing" technical white paper (full entry)**

**Identity, owner, year**
- zRef is a reference *application*: a test kit with a manual. It is not a published benchmark specification.
- HP (HPE after the 2015 split) built it "from the TPC-E transaction set to be representative of a typical Mainframe workload with an online (CICS) and batch component". It was "originally deployed and implemented on an IBM Z System Mainframe to establish a baseline for MIPS".
  - Source: the Microsoft Azure Global post, syndicated at https://thewindowsupdate.com/2019/07/04/mips-equivalent-sizing-for-ibm-cics-cobol-applications-migrated-to-microsoft-azure-2/ (fetched).
- Didier Durand (2016-07-26): "In 2013, HPE published a very interesting paper about the results of its proprietary zRef benchmark which compares the same Cobol programs (transactional and batch) executed both on mainframes and on x86 servers."
  - Source: http://media-tech.blogspot.com/2016/07/computing-power-comparison-mainframe.html (fetched).
- TmaxSoft captions one diagram "zRef Overview from zRef Manual", so a manual exists. No copy was found.

**URLs and fetch status**
- HP original, "Mainframe Alternative Sizing -- Technical white paper", document 4AA4-2452ENW, at http://h20195.www2.hp.com/V2/GetPDF.aspx/4AA4-2452ENW.pdf: dead. It redirects to psnow.ext.hpe.com and returns 404; www.hpe.com/psnow/doc/4AA4-2452ENW timed out; the tool could not reach web.archive.org.
- Mirrors:
  - https://manualzz.com/doc/24646236/mainframe-alternative-sizing%E2%80%94technical-white-paper: dead (HTTP 403).
  - pdfslide.net mirror: dead; the URL now redirects to a different domain (checked 2026-11-25), so no link is given.
- Search snippets of the manualzz mirror (not verified):
  - "Sizing is done with zRef, a reference application suite of batch and CICS workloads".
  - The aim is to compare "relative throughput between various HP Mission-Critical Converged Infrastructure systems and an IBM System z mainframe running the same z/OS batch and CICS application".
  - The table of contents includes DL980 setup, "Cost of MIPS", a DR cost comparison, and a "zRefCICS" figure of "124 TPS at 93% CPU busy" on z/OS.
- Another meaning of the name: zref is also an unrelated LaTeX cross-referencing package by Heiko Oberdiek, https://ctan.org/pkg/zref?lang=de (not fetched; from search snippet; not verified). No IBM Z reference architecture, GitHub project or s390x tool called zRef was found.

**Workload design** (from the TmaxSoft study, fetched, unless stated otherwise)
- "zRef is based on the TPC-E benchmark, but it was not built to 'achieve the highest throughput possible' benchmark. zRef was programmed to be as much like a real-world 'mainframe' as possible". This quote was spot-checked by a second reader.
- Coding style: "SQL calls to Db2 are simple and many, often using a position cursor and retrieving a single table row at a time", with COBOL "coding habits that date back to the 1980s".
- Database: 33 tables taken from the TPC-E schema.
- Online: "10 CICS transactions processed by more than 30 executable CICS programs that read, update, and insert data into a DB2 database and optionally log audit information (write) to a VSAM ESDS file", submitted by TN3270 "users".
- Batch:
  - **zRefBR** (Batch Reporting) is read-only: extract steps from the database, then "classic sort/merge sequential file processing".
  - **zRefBU** (Transactional Batch Update) runs "the same business 'transactions' as in the CICS transactional workload" from an input file, simulating transactions that arrive by FTP.
- Concurrency: the jobs can run "one at a time with multiple copies concurrent in parallel and also concurrent with zRef CICS transaction processing".

**Results attributed to HP itself** (second-hand only)
- Per Durand (fetched): "1'000 MIPS of mainframe power can be replaced with better performances (on this benchmark) by a single server HP DL980 equipped with 8 Xeon E7-4870 (2.4 GHz, 10 cores / Xeon, 1 TB RAM)".
- Microsoft CAF, attributed to "a recent HPE zRef benchmark": 288 MIPS per Intel core online (CICS) and 170 for COBOL batch. This is from a search snippet (not verified); the page now redirects (see VN07).
- These two do not agree. 1,000 MIPS over 80 cores is about 12.5 MIPS per core if every core was busy (our arithmetic, from the zRef strand). The 288/170 figures probably come from a different, later ProLiant run, or from a per-core extrapolation. This cannot be confirmed.

**Every use of zRef found**
- HP, 2013 (above).
- Asysco on Azure, 2019 (VN03).
- TmaxSoft on Azure, about 2020-21 (VN02).
- LzLabs SDM, 2021 (VN04).
- No Micro Focus use of the zRef name was found. HP's 2009 Micro Focus paper (VN05) is a probable precursor that does not use the name.

**Methodology quality and independence**
- Every result was built, run and published by HP(E) or a migration vendor with a commercial stake.
- The kit is "proprietary". It is unaudited and has not been reproduced outside the vendors.
- Every MIPS-equivalent figure depends on one undisclosed IBM baseline run: model, LPAR weight, z/OS, CICS and DB2 levels, and measurement window are all unknown.
- Unknown: think time, transaction mix, database scale, zRefBR record counts, and the definition of a "zRef TPS".
- No critique of zRef by name was found, and no IBM response.
- The row-at-a-time DB2 access is deliberate. It models typical legacy code, not either platform's best case.

**Maps to:** batch+online mix (primary), sort (zRefBR sort/merge steps), RMW/IO (zRefBU updates, audit writes to ESDS), cost/energy.

**Useful to us**
- This is **the closest published analogue to our CICS-like online test under a concurrent batch sort**. zRefBR extracts and then sort/merges while the CICS load runs, and the vendors report online TPS, response time and batch elapsed time, both alone and together. Borrow that reporting shape:
  - online TPS and response time alone, then with N batch jobs;
  - batch elapsed time alone, then under online load;
  - CPU% on each tier.
- The gap our test fills: zRef never reports the z/OS side's behaviour under the same mix. Its only z/OS figures are the baseline columns (68.1 and 161.9 min) and the snippet's "124 TPS at 93% busy". It is also never run at equal core counts.
- Our 2-vs-2 design, with WLM on z/OS and the plain Linux scheduler or cgroups on the PC, measures what zRef leaves out.

**Also in:** zref.md (all sections); vendors C10 and C12; analysts Q3; academic B4 (HPE zRef via Durand's LinkedIn version).

**VN02. "TmaxSoft OpenFrame and Microsoft Azure Mainframe Migration Sizing Study"**
- **Who / when:** TmaxSoft with Microsoft. The vendors strand dates the study to 2020. The PDF footer reads "(c) TmaxSoft 2023" and its metadata was last modified 2023-10-10.
- **URLs:**
  - https://www.tmaxsoft.com/upload/eBookMgmt/20231219134111167015.pdf (fetched by the zRef strand; figures spot-checked by a second reader).
  - Blog: https://www.tmaxsoft.com/openframe-on-azure-vs-system-z-apps-and-infrastructure-who-wins (fetched; Kelly McClure).
  - Press page: https://www.tmaxsoft.com/en/press/view?seq=218&pageIndex=1&pageUnit=12&searchKeyword=nto+the+wide+blue+yonder%3A+Mainframe+modernization+on+Microsoft+Azur&searchUseYn=Y (fetched; Karthik Masilamani).
  - The older study URL, https://us.tmaxsoft.com/wp-content/uploads/TmaxSoft_Azure-Benchmark-Study.pdf, failed DNS. The vendors strand saw the TPS figures only as a snippet; the zRef strand later read them in the full PDF.
- **Compared:** TmaxSoft OpenFrame on Azure (Tibero 6, RHEL 7.7, premium SSD) against an "IBM Z mainframe" baseline whose model and size are not stated.
  - Study 1: F16s_v2 application server (16 vCPU, 32 GB) and E16s_v3 database (16 vCPU, 128 GB).
  - Study 2: F32s_v2 (32 vCPU, 64 GB) and E32s_v3 (32 vCPU, 256 GB).
  - The load came from py3270 on a B20ms client VM.
- **Workload:** zRef CICS, zRefBR and zRefBU (see VN01).
- **Headline** (spot-checked):

  | Scenario | Users | TPS | CPU, app / DB | Average response | MIPS-equivalent | Cost |
  | --- | --- | --- | --- | --- | --- | --- |
  | Study 1 | 2,000, 1 h | **1,672** | 79% / 92% | 207 ms | 13,676 | about $17,641/yr |
  | Study 2 | 4,500, 1 h 10 min | **2,600** | 76% / 84% | 242 ms | 21,268 | about $26,624/yr |
  | Study 1 VMs + 3 zRefBR jobs | 1,500 | **1,409** | 68% / 72% | 99 ms | 11,525 | — |

  - With CICS running, the "zRefBR jobs completed in 16 mins (i.e. 3 mins longer when compared to no CICS workload)".
  - Standalone batch, z/OS baseline against OpenFrame: zRefBR **68.1 against 6.21 min**; zRefBU **161.9 against 32.1 min**.
  - Concurrent 5x zRefBU + 3x zRefBR: zRefBR about 13 min, zRefBU about 32 min.
  - Vendor conclusion: "conclusive proof that OpenFrame on Azure can perform better than a IBM Z mainframe".
- **Ambiguities, stated rather than resolved:**
  - The 1,409 TPS run used 1,500 users, against 2,000 for the 1,672 TPS run. The drop is therefore not a like-for-like measure of batch interference.
  - The "no CICS" zRefBR reference of about 13 min matches the 5xBU+3xBR concurrent-batch scenario, not the standalone 6.21 min. The study does not say which baseline it meant.
- **Quality / independence:** vendor plus cloud provider. "Performance results may vary for each application". The IBM baseline is undisclosed.
  - All three MIPS figures equal TPS x about 8.18, which suggests one baseline scaled linearly (our arithmetic, zRef strand). 124 TPS x 8.18 is about 1,014, consistent with a 1,000 MIPS baseline but unconfirmed.
  - Per vCPU (our arithmetic, zRef strand): Study 1 is about 855 MIPS per application vCPU, or about 427 counting the database VM; Study 2 about 665, or 332. Whether these vCPUs are cores or hyperthreads was not checked.
- **Maps to:** batch+online mix, sort, RMW/IO, cost/energy.
- **Useful to us:** the only published pair of figures for online TPS with and without concurrent sort/merge batch, plus batch elapsed time with and without online load. It is the template for our results table.
- **Also in:** vendors C12; zref Q4.

**VN03. "MIPS Equivalent Sizing for IBM CICS COBOL Applications Migrated to Microsoft Azure" (Asysco AMT GO)**
- **Who / when:** Microsoft AzureCAT, with partner Asysco. Syndicated 4 July 2019.
- **URLs:**
  - Syndicated copy: https://thewindowsupdate.com/2019/07/04/mips-equivalent-sizing-for-ibm-cics-cobol-applications-migrated-to-microsoft-azure-2/ (fetched).
  - Original: https://techcommunity.microsoft.com/t5/AzureCAT/MIPS-Equivalent-Sizing-for-IBM-CICS-Cobol-Applications-Migrated/ba-p/731665 (dead: login redirect).
- **Compared:** AMT GO on Azure M-series VMs with SQL Server Enterprise, against a zRef MIPS baseline on "an IBM Z System mainframe". The VM sizes are not given.
- **Workload:** zRef online plus batch.
- **Headline:** "over 18,000 MIPS on a single system and over 28,000 on a multi-server system ... comparable to an IBM mid-level z14 processor". Batch "can run concurrently with online CICS COBOL work while still meeting online end-user SLA commitments". No TPS figures.
- **Quality / independence:** self-reported by the partner. The derivation is in a whitepaper (asysco.com/ibm/) that was not fetched.
- **Maps to:** batch+online mix, cost/energy.
- **Useful to us:** a second zRef use that claims batch and online coexist. It gives no numbers to check against.

**VN04. LzLabs / Microsoft Azure whitepaper (LzSDM)**
- **Who / when:** LzLabs, 2021.
- **URLs:**
  - https://www.lzlabs.com/wp-content/uploads/2021/10/LzLabs-Microsoft-Azure-whitepaper.pdf (fetched).
  - Linked zRef report: https://info.lzlabs.com/zref-performance-benchmark (dead: ENOTFOUND).
  - Microsoft's page "Download The latest zRef benchmark from LzLabs": https://acom-sandbox.azure.net/en-us/solutions/migration/mainframe/ (fetched mirror).
- **Compared:** LzSDM on "older x86 processors" on a single VM.
- **Workload:** zRef.
- **Headline:** "has been shown to easily support up to 4,000 transactions per second on older x86 processors and on a single VM instance".
- **Quality / independence:** vendor. The detailed report is unreachable.
- **Maps to:** batch+online mix.
- **Useful to us:** little until the report can be retrieved.

**VN05. "Sizing Mainframe Workloads for Intel Itanium and Intel Xeon-based Platforms" (HP 4AA2-7670ENW)**
- **Who / when:** HP with Micro Focus, August 2009.
- **URL:** https://www.intel.com/content/dam/doc/white-paper/risc-migration-itanium-xeon-hp-mainframe-workloads-paper.pdf (fetched).
- **Compared:** an HP rx8640 (16 Itanium cores, HP-UX, Oracle 10g) and an 8-core 3.0 GHz x86 blade (Windows Server 2008, SQL Server 2008), against z/OS MIPS.
- **Workload:** COBOL, CICS and IMS workloads run by Micro Focus.
- **Headline:**
  - The Itanium system rated 4,714 z/OS MIPS ("295 MIPS per core").
  - The x86 blade rated 2,165 MIPS ("271 MIPS per core").
  - The paper warns that MIPS per core depends on scaling: on a z10 EC the 64th CPU adds 462 MIPS against 890 for a uniprocessor. It also warns that tpm per MIPS ranges from 30 to 200 by workload.
- **Quality / independence:** vendor. The zRef name does not appear in it.
- **Maps to:** batch+online mix, cost/energy.
- **Useful to us:** a vendor's own admission that MIPS per core is not constant, even on Z.

**VN06. "Move mainframe compute to Azure Virtual Machines" (Microsoft Learn sizing rule)**
- **Who / when:** John Rayborn, Microsoft. ms.date 2019-04-02; now retired, with an archive copy updated 2025-08-05.
- **URL:** https://learn.microsoft.com/en-us/azure/virtual-machines/workloads/mainframe-rehosting/concepts/mainframe-compute-azure (fetched; redirects to the archive).
- **Compared:** an IBM z14 against Azure VMs. Nothing was measured.
- **Workload:** none.
- **Headline:**
  - "150 MIPS per vCPU" and "1,000 MIPS per processor".
  - "an F-series VM with 16 vCPUs could easily support an IBM Db2 workload coming from an LPAR with 2,400 MIPS".
  - "a virtual CPU (vCPU) roughly equates to a core on a mainframe".
- **Quality / independence:** vendor guidance with no benchmark, and it contradicts itself. A vCPU "equates to a core", yet 1,000 MIPS per processor against 150 per vCPU means about 6.7 vCPUs per processor. The page also says the z14 has "up to 240 cores" but sizes it as "24 processors ... about 24,000 MIPS".
- **Related rules** (all not fetched; from search snippet; not verified):
  - CMG 2018: "100 MIPS equates to a single core", https://www.cmg.org/2018/07/what-to-consider-when-evaluating-a-mainframe-migration/
  - Astadia, quoting Microsoft at about 150 MIPS per core: https://www.astadia.com/blog/can-you-really-save-80-of-your-annual-mainframe-operating-costs
- **Maps to:** cost/energy.
- **Useful to us:** the most-quoted conversion rule, and an unmeasured one. Our CPU-seconds-per-unit-of-work figures can be set beside it.
- **Also in:** vendors C3; analysts Q3 and Q4.

**VN07. Microsoft Cloud Adoption Framework, "Mainframe migration: migration strategies"**
- **Who / when:** Microsoft, date unknown. It cites "a recent HPE zRef benchmark".
- **URLs:**
  - https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/infrastructure/mainframe-migration/migration-strategies and https://learn.microsoft.com/id-id/azure/cloud-adoption-framework/infrastructure/mainframe-migration/migration-strategies (not fetched; from search snippet; not verified). Both now redirect to "Plan your migration", and the GitHub source returns 404.
  - A 2022 Microsoft-Intel whitepaper reportedly repeats the figures: https://www.prowesscorp.com/wp-content/uploads/2022/06/210178-MSFT-Intel-MainframeMigration-WP-Final_V2.pdf (not fetched; from search snippet; not verified).
- **Compared:** HPE ProLiant Intel cores against z/OS MIPS.
- **Workload:** zRef online and batch.
- **Headline:** 288 MIPS per Intel core online and 170 per core for COBOL batch. The planning figures are 200 MIPS per vCPU online and 100 for batch.
- **Quality / independence:** a second-hand reference to an HPE benchmark; HPE sold the x86 side.
- **Maps to:** batch+online mix, cost/energy.
- **Useful to us:** the target-platform vendor halves its batch rating. That is an admission, on the vendor's side, that batch moves worse than online, and it sits beside VN10. It contradicts VN02's batch result.

**VN08. "High-Performance Mainframe Workloads on AWS with Cloud-Native Heirloom PaaS"**
- **Who / when:** Gary Crook (Heirloom CEO), AWS APN Blog. The year was not confirmed.
- **URL:** https://aws.amazon.com/blogs/apn/high-performance-mainframe-workloads-on-aws-with-cloud-native-heirloom-paas (the fetch returned only navigation; not fetched; from search snippet; not verified).
- **Compared:** Heirloom on AWS against a mainframe MIPS-per-TPS ratio that is not published.
- **Workload:** a CICS/COBOL rehost.
- **Headline:** "equivalent MIPS rating of 15,200 at a sustained transaction throughput of 1,018 transactions per second"; about $350,000 a year; AWS "not tuned".
  - A "15,000+ TPS" figure was not found: 15,200 is a MIPS figure.
  - That is about 14.9 MIPS per TPS (our arithmetic, vendors strand).
- **Quality / independence:** vendor, published on the cloud provider's partner blog.
- **Maps to:** batch+online mix (online), cost/energy.
- **Useful to us:** it shows how a "MIPS-equivalent" is manufactured: TPS multiplied by a hidden ratio. Report CPU-seconds per transaction instead.

**VN09. "Heirloom Computing Says Its Cloud Surpasses Traditional Mainframe Performance"**
- **Who / when:** DBTA news, 13 May 2013.
- **URL:** https://dbta.com/Editorial/News-Flashes/Heirloom-Computing-Says-Its-Cloud-Surpasses-Traditional-Mainframe-Performance-89567.aspx (fetched).
- **Compared:** an HP Compute Cloud "Standard, Extra Large" instance (4 CPUs, 16 GB) against a "905 MIPS mainframe" equivalent.
- **Workload:** a CICS benchmark the vendor labels "TPC-C", with 1,500 terminals.
- **Headline:**
  - 12,105 CICS transactions per minute, about 202/s (our arithmetic, vendors strand).
  - "90% lower" cost.
  - About 226 MIPS per CPU (our arithmetic, analysts strand).
- **Related:** Heirloom's planning "rule of thumb" of about 105 MIPS per core, https://heirloomcomputing.com/?p=2704 (not fetched; from search snippet; not verified).
- **Quality / independence:** vendor press release. No mainframe run and no audit; the TPC-C label is unofficial.
- **Maps to:** batch+online mix, cost/energy.
- **Useful to us:** a historical example of a 4-CPU cloud claim. A 4-CPU instance is close to our 2-core scale.
- **Also in:** analysts C12.

**VN10. "Mainframe Migration - The Latency Challenge"**
- **Who / when:** Graham Cunningham, Heirloom Computing, 7 Aug 2025.
- **URL:** https://heirloomcomputing.com/the-latency-challenge/ (fetched; re-checked 2026-11-25).
- **Compared:** one nightly batch job on the mainframe against the same job after a cloud rehost. Hardware is not specified.
- **Workload:** a batch job making 2.3 million database reads.
- **Headline:**
  - "On the mainframe, with zero latency, this took 45 minutes."
  - "When we migrated to cloud, preserving all the business logic perfectly, the same job took 8 hours."
  - Illustrated with a hypothetical, not a measurement: "If each query takes 2 milliseconds", 2.3 million queries add "76 minutes of pure network latency".
  - That hypothetical (45 + 76 min) does not explain 8 hours, and no time after the fix is reported.
- **Quality / independence:** an anecdote from a rehoster, used to promote its own transpilers ("That's the challenge we're tackling at Heirloom"). No hardware, database or network specification.
- **Maps to:** RMW/IO, batch+online mix (batch).
- **Useful to us:** direct support for a record-at-a-time RMW test that reports per-record latency and records/s. Testing a local against a networked database on the PC would show the effect.
  - Contradiction: VN08 reports scaled-out online throughput from the same vendor, and VN02 reports batch faster after a rehost.

**VN11. "Scaling for performance with AWS Mainframe Modernization and Micro Focus"**
- **Who / when:** AWS Cloud Operations Blog.
- **URL:** https://aws.amazon.com/blogs/mt/scaling-for-performance-with-aws-mainframe-modernization-and-micro-focus (the fetch returned navigation only; not fetched; from search snippet; not verified).
- **Compared:** Micro Focus Enterprise Server on c5.large (2 vCPU), c5.4xlarge, and 9x c5.large.
- **Workload:** "tasks" driven by virtual users. The application is not confirmed.
- **Headline:**
  - One c5.large handled over 400 tasks/s with 3,000 virtual users.
  - Vertical scaling was "linear".
  - 9x c5.large (18 vCPU) handled about 27,000 users at over 4,000 tasks/s.
- **Quality / independence:** written by AWS. "Tasks" are not mapped to transactions or MIPS.
- **Maps to:** batch+online mix (online), choke/limits.
- **Useful to us:** the only rehost figure found per 2 vCPU, about 200 tasks/s per vCPU (vendors strand). That is our core budget, though the page body is needed to confirm the workload.

**VN12. EDS / Micro Focus / Microsoft / Unisys Enterprise Server scalability study**
- **Who / when:** Microsoft TechNet, 2003 era.
- **URL:** https://technet.microsoft.com/en-us/library/bb463147.aspx (not fetched; from search snippet; not verified).
- **Compared:** an 8-CPU Windows system against mainframe MIPS.
- **Workload:** not known.
- **Headline:** about "1,715 mainframe MIPS" for 8 CPUs.
- **Quality / independence:** vendor consortium.
- **Maps to:** cost/energy.
- **Useful to us:** a data point in the wide spread of MIPS-per-core figures.

**VN13. Microsoft-commissioned VeriTest benchmarks of Linux on zSeries against Windows Server 2003**
- **Who / when:** Microsoft and VeriTest (Lionbridge), 2003, as covered by Computerworld and InfoWorld (both 5 September 2003).
- **URLs:** https://www.computerworld.com/article/1586930/microsoft-benchmarks-step-up-linux-assault.html ; https://www.infoworld.com/article/2224366/update-microsoft-benchmarks-step-up-linux-assault.html (both fetched 2026-11-25). The original PDFs were not found.
- **Compared:** Linux on one zSeries CPU, which "performed at about the same as Windows NT 4 on one 900MHz Intel processor"; "Windows 2003 outperformed it". Microsoft ran the mainframe tests itself, and VeriTest the Intel side.
- **Workload:** file and web serving.
- **Headline:**
  - Microsoft claimed that Linux on the mainframe "lags behind Windows 2003 on Intel Corp. systems in terms of performance for the money" (InfoWorld).
  - IBM's response: "We typically don't run industry standard performance benchmarks for any software on the mainframe".
- **Quality / independence:** an adversarial vendor. The scripts are unpublished.
- **Maps to:** cost/energy, choke/limits.
- **Useful to us:** a historic single-CPU comparison run by an adversarial vendor. A small mainframe slice invites objections about how it was chosen, and our 2-CP slice will face them, so state the slice size and that the LPAR is shared, up front.
- **Also in:** benchmarks_community item 2.

**VN14. Temenos public-cloud benchmarks (2022 press release, 2024 blog, 2025 report)**
- **Who / when:** Temenos.
- **URLs:**
  - 2022 press release, 18 May 2022: https://www.temenos.com/press_release/temenos-banking-cloud-scales-to-record-high-transaction-volumes-and-achieves-world-class-efficiency/ (fetched).
  - 2024 blog: https://www.temenos.com/blog/achieving-maximum-scalability-with-a-minimum-footprint-temenos-highwater-benchmark-2024/ (fetched).
  - 2025 report: https://www.temenos.com/wp-content/uploads/2025/09/Temenos-Performance-Benchmark-2025.pdf (not fetched; from search results).
- **Compared:** Temenos Banking Cloud on AWS with MongoDB and Yugabyte. Instance types and core counts are not stated, and there is no mainframe run.
- **Workload:** 100 million customers and 200 million deposit accounts; Demand Deposit Accounts and Loans.
- **Headline:**
  - 100,000 TPS, "up to 61 TPS per core".
  - It "matches the performance and scale of mainframe-based systems".
  - 4.1x more efficient per TPS per core than before.
  - The 2024 blog gives "up to 2.3x TPS per core" year on year.
- **Quality / independence:** self-published, with no auditor and no configuration.
- **Maps to:** batch+online mix (online), cost/energy.
- **Contradiction:** IBM's 2020 paper (CS02) puts x86 public cloud at 8.6 TPS/core for Transact R19, against 61 here. The software generations differ, so per-core TPS here mostly reflects the software stack.
- **Useful to us:** hold the program constant across platforms.

**VN15. "Hazelcast Performance claims on IBM z16 compared to x86"**
- **Who / when:** an IBM performance study, published on hazelcast.com. The PDF was created 2026-06-26.
- **URL:** https://hazelcast.com/wp-content/uploads/2026/07/Hazelcast-Performance-claims-on-IBM-z16-compared-to-x86.pdf (fetched).
- **Compared:**

  | | z16 (3931 Max125) | x86 |
  | --- | --- | --- |
  | Cluster | 3 nodes of 5 dedicated IFLs (15) on z/VM 7.3, RHEL 9.5, 32 GB per node | 3 bare-metal nodes of 20 cores (60), Ubuntu 22.04, 460 GB per server |
  | Client | 4 IFLs (TPS test) or 10 IFLs (latency test) | 20 cores |
  | Network | 25 Gb OSA (TPS test) or HiperSockets (latency test) | 40 Gbps |
  | CPU model | z16 | not named |

- **Workload:** Hazelcast Enterprise 5.5.5 message streaming with 100 KB payloads.
- **Headline:** "16x more TPS per core"; p99 latency "2.75x" lower at a fixed 20,000 ops/s with 200 clients.
- **Quality / independence:** run by IBM; a claim sheet only, with no absolute TPS. The OS differs between sides, and the latency test partly measures in-box HiperSockets.
- **Maps to:** batch+online mix (online), RMW/IO (network), cost/energy.
- **Useful to us:** a lesson in normalisation. Name both CPUs exactly and keep the interconnect identical.
- **Also in:** benchmarks_community item 19.

**VN16. "FUJITSU Enterprise Postgres on IBM LinuxONE: Improving your organization's data performance"**
- **Who / when:** Fujitsu white paper, (c) 2022, stamped 2025-03-24.
- **URL:** https://www.postgresql.fastware.com/hubfs/Collateral/_IBM/WhitePaper-FujitsuEnterprisePostgresOnIbmLinuxOne-14.pdf (fetched).
- **Compared:** Fujitsu Enterprise Postgres (proprietary VCI columnar index and High-speed Data Load) on LinuxONE III with 32 IFLs (8 for the load test), against **open-source PostgreSQL 11** on an "equivalent" 32-core or 8-core x86. The x86 CPU is not named.
- **Workload:** HTAP on 120 GB with 512 clients; TPC-C-like at 128/256/512 clients; bulk load.
- **Headline:**
  - Analytical queries 3.9x faster.
  - TPC-C-like throughput 1.4x, 1.8x and 2.1x at 128, 256 and 512 clients.
  - Data load 8.4x.
- **Quality / independence:** confounded, because the software differs on each side.
- **Related caution:** EDB's "2.1x per core" pgbench claim is for IBM Power11 against x86 (15 against 40 cores), not LinuxONE. https://mktgsite.enterprisedb.com/sites/default/files/2026-06/EDB-IBM-Power-Benchmark.pdf (not fetched; from search snippet; not verified).
- **Maps to:** RMW/IO, batch+online mix.
- **Useful to us:** the ratio rising with client count is a "who chokes first" curve, so report ratio against concurrency rather than a single point. Never compare different software on the two sides.

**VN17. IBM LinuxONE MongoDB / YCSB decks (2015-2016)**
- **Who / when:** IBM.
- **URLs** (all not fetched; from search snippet; not verified):
  - https://www.slideshare.net/slideshow/mongodb-linux-porting-performance-measurements-and-and-scaling-advantage-using-the-z13-ibm-mainframe/49050079
  - https://www.slideshare.net/slideshow/linux-one-vs-x86/64284084
  - Marist-hosted Mitran deck, May 2016: https://ecc.marist.edu/documents/367507/568146/MitranLinuxONE+Marist+MMIT+20160505+V1.0.pdf/b1666d3f-35f6-404f-9f77-eb80af47f32f
- **Compared:** z13 / LinuxONE against a Xeon E5-2699 v3.
- **Workload:** YCSB 0.2.0 with MongoDB 3.0.4 WiredTiger on RHEL 7.1.
- **Headline:** "up to 2.4x" in one deck and "1.9x to 2.1x" in the other. The z13 slides give ITR ratios "based on measurements and projections", including 17 TB on one node at under 5 ms.
- **Quality / independence:** vendor, and the two decks disagree.
- **Maps to:** RMW/IO.
- **Useful to us:** YCSB is a candidate workload, and the disagreement itself is a teaching point.
- **Also in:** vendors Q5; benchmarks_community item 21.

**VN18. "Exploring IBM Z Sort Accelerator Support in Syncsort MFX"**
- **Who / when:** Precisely (formerly Syncsort), 28 Oct 2021.
- **URL:** https://www.precisely.com/blog/mainframe/ibm-z-mainframe-sort-accelerator-support-in-syncsort-mfx/ (fetched).
- **Related** (all not fetched; from search snippet; not verified):
  - PipeSort sheet: https://www.precisely.com/wp-content/uploads/2024/07/PS_Syncsort-PipeSort_240719_E_final.pdf
  - Mexican bank story: https://www.precisely.com/app/uploads/2020/06/CS_Mexican-Bank_Syncsort-MFX_E_200530_E_final.pdf
  - MFX datasheet: https://www.precisely.com/wp-content/uploads/2024/07/PS_Syncsort-MFX_240723_E_final.pdf
  - A Connect/DMExpress note: https://mainframelicensingexperts.pages.dev/blog/benchmarking-syncsort-precisely-mainframe-spend/
- **Compared:** MFX on z15 with and without the Z Sort accelerator. There is no x86 side and no DFSORT side.
- **Workload:** sort, copy and merge.
- **Headline:**
  - CPU and elapsed time improved "up to 35%". Results "depend on key length, record length, file size".
  - Separately, up to 45% CPU and 40% elapsed savings; up to 80% CPU saving with zIIP offload.
  - Copy with zIIP: 25% elapsed and up to 90% CPU (customer reports).
  - PipeSort: a 1 GB file sorted three ways went from 1 h 9 min to 31 min.
  - The Mexican bank saw about 40% CPU and 10-15% elapsed savings.
- **Quality / independence:** vendor lab, "up to" figures, no workload specification. Precisely competes with DFSORT.
- **Maps to:** sort.
- **Useful to us:** it confirms the sweep axes: key length, record length and file size.
  - Gap: no ISV publishes mainframe sort against Linux sort. Connect/DMExpress on Linux is positioned as an offload target with no head-to-head numbers.

**VN19. Google Cloud Dual Run (Santander "Gravity")**
- **Who / when:** Google, with Santander, 2022 onward.
- **URLs** (all not fetched; from search snippet; not verified):
  - https://docs.cloud.google.com/mainframe-dual-run/docs/dual-run-overview
  - https://services.google.com/fh/files/misc/google-cloud-dual-run-white-paper-mainframe-mod.pdf
  - https://www.santander.com/en/press-room/press-releases/2022/10/google-and-santander-to-help-other-companies-simplify-their-transition-from-legacy-tech-to-the-cloud
- **Compared:** a mainframe and Google Cloud running in parallel.
- **Workload:** production workloads, comparing functional and non-functional behaviour.
- **Headline:** no performance numbers published.
- **Related:** the Infosys/CNCF rehost case reports a 25% cut in development cycle time, not runtime. https://www.cncf.io/case-studies/infosysmainframe/ (not fetched; from search snippet; not verified).
- **Quality / independence:** vendor; method only.
- **Maps to:** batch+online mix (method).
- **Useful to us:** the same-input, compare-outputs-and-timings design our grid already uses.

**VN20. Linux-side interference claims from vendors (Multikernel; USPTO 9268702)**
- **URLs:** https://multikernel.io/ ; https://image-ppubs.uspto.gov/dirsearch-public/print/downloadPdf/9268702 (both not fetched; from search snippet; not verified).
- **Compared:** Linux cgroups against each vendor's own method. There is no Z side.
- **Workload:**
  - Multikernel: a latency-sensitive task beside a memory-hungry batch job.
  - USPTO: TPC-E under storage noise.
- **Headline:**
  - Multikernel: p99 of 232 ms against 35.6 s. It argues cgroups protect the victim only by starving the batch job.
  - USPTO: TPC-E degradation with cgroups below 10% at low and medium noise, 25% at high noise, against 7% for the patented method.
- **Quality / independence:** vendor and patent claims.
- **Maps to:** batch+online mix, choke/limits.
- **Useful to us:** on the PC side, measure both the victim's p99 and the batch job's slowdown, because isolation can trade one for the other.


### 3.4 Analysts and independent testers (12 items)

**AN01. "TCO Analysis: Workload Consolidation on IBM LinuxONE"**
- **Who / when:** Mitch Lewis, Signal65. Commissioned by IBM: the PDF says "in partnership with IBM" and the landing page says "Research commissioned by: IBM". The vendors strand dates the PDF July 2025; the URL path and landing page say October 2025.
- **URLs:** https://signal65.com/wp-content/uploads/2025/10/Signal65-TCO-Analysis_Workload-Consolidation-on-IBM-LinuxONE.pdf (fetched, read in full) ; https://signal65.com/research/workload-consolidation-on-ibm-linuxone/ (fetched).
- **Compared:** an IBM LinuxONE Emperor 5 Max 136 (3 CPC drawers, 136 PUs), tested as 1 LPAR with 16 IFLs, against two x86 estates.
  - Study 1: six older x86 servers with 256 cores in total. The mix was 2x Xeon Platinum 8260 24C, 2x E5-2690v3 12C, 2x Gold 5220R 24C, 2x Gold 6242 16C, 2x Gold 6252 24C and 2x Platinum 8276 28C.
  - Study 2: new servers with 2x Xeon Platinum 8592+ (64C each), bought in December 2024.
  - Both sides ran the same stack: WebSphere Liberty plus EDB Postgres on OpenShift, RHEL 9.5 with KVM, and the same storage.
- **Workload:** a web/DB transactional workload sized to about 10,000 req/s. Runs were 5 minutes: 1 minute ramping to 1,000 users, 3 minutes of test, 1 minute ramping down. Power was measured at several utilisation levels. The workload is not named in the PDF.
- **Measured headline:** 256 older x86 cores against 16 IFLs at about 10,000 req/s, or 16:1 (our arithmetic, vendors strand).
- **Modelled headline:**

  | | Production cores (LinuxONE vs x86) | Non-production cores | Total cores | 5-year TCO |
  | --- | --- | --- | --- | --- |
  | Study 1 | 39 vs 512 | 97 vs 4,864 | 136 vs 5,376 (126 servers) | $15.23M vs $64.79M (-76%) |
  | Study 2 | 20 vs 256 | 116 vs 2,688 | 136 vs 2,944 (23 servers) | $13.50M vs $24.17M (-44%) |

  - Software: -94% in both studies.
  - Energy: -91% against old x86 and -65% against new.
  - Hardware: higher on LinuxONE, +182% and +446%.
  - Study 2 lines: software $18,385,012 vs $1,124,120; hardware $1,581,172 vs $8,627,879; energy $87,922 vs $30,464.
- **Quality / independence:** IBM collected the data and Signal65 "validated" it. Nothing is reproducible: no load tool, application or tuning is published. The model assumptions favour consolidation:
  - x86 non-production is 3-4x production at 7-15% CPU;
  - x86 production averages under 50%;
  - LinuxONE runs at 80-95%;
  - LinuxONE labour is 50% of x86;
  - $0.15/kWh, GSA list prices, $150k per FTE.
- **Arithmetic** (our arithmetic, analysts and vendors strands):
  - Study 2 production cores are 256:20 = 12.8:1, but non-production is 2,688:116 = 23:1.
  - Software is about 76% of x86 TCO ($18.4M of $24.2M).
  - So the "2,944" headline (IB16) depends mostly on the non-production assumption.
- **Maps to:** cost/energy, batch+online mix (online only).
- **Useful to us:** the measured design is worth copying at small scale: fix the offered load, compare the cores needed, and meter watts.
  - Gap: no per-core figure for modern x86 at equal utilisation, and no batch, sort, crypto or AI work.
- **Also in:** vendors C1; analysts C01; consolidation section 3.7 (cross-reference).

**AN02. "IBM LinuxONE 5 Gets Huge AI Boost"**
- **Who / when:** Karl Freund, Cambrian-AI Research, 2025-07-21. The paper says "This white paper was sponsored by IBM", and Cambrian-AI lists IBM, Intel and NVIDIA as clients.
- **URL:** https://www.ibm.com/downloads/documents/id-id/137a1e23d8dba6c3 (fetched, PDF text extracted).
- **Compared:**
  - Z side: 1 LPAR on Ubuntu 24.04 with 7 dedicated cores (SMT) and 256 GB.
  - x86 side: "28 Emerald Rapids Intel Xeon Gold CPUs @ 2.20 GHz with Hyper-Threading turned on, 1 TB memory", running Triton with OpenVINO on CPU (AMX).
  - Inference runs locally on Z but **remotely** on x86, across a network hop.
- **Workload:** the public OLTP app github.com/IBM/megacard-standalone with PostgreSQL, plus a fraud ensemble of LSTM (github.com/IBM/ai-on-z-fraud-detection) and TabFormer (github.com/IBM/TabFormer). JMeter 5.6.3 drives it with 64 users.
- **Headline:**
  - One AI accelerator "matches the throughput of running inference on a compared remote x86 server with 13 cores".
  - Energy 81% lower against remote x86 with a GPU.
  - "up to $24 million" in 5-year TCO savings.
  - 450 billion inferences/day (batch 160, 6 IFLs); LinuxOne 4 reaches 300 billion (48 IFLs, batch 128, 8 threads pinned one per chip).
- **Quality / independence:** sponsored; the comparison is lopsided because of the remote hop.
- **Maps to:** AI inference, mixed grid, cost/energy.
- **Useful to us:**
  - All three code bases are public, so our mixed grid could run MegaCard plus the LSTM model on both sides, in-process, at the same batch size.
  - Our same-box design removes the network asymmetry.
- **Also in:** IB04 fn1.

**AN03. "Optimizing DFSORT Use of Z Sort Accelerator"**
- **Who / when:** Watson & Walker, Cheryl Watson's Tuning Letter 2022 No. 2. IBM hosts the reprint.
- **URL:** https://ibm.com/support/pages/system/files/inline-files/Optimizing%20DFSORT%20Use%20of%20Z%20Sort%20Accelerator.pdf (fetched).
- **Compared:** DFSORT with Z Sort against DFSORT without it. The article quotes IBM's figures and runs no tests of its own.
- **Workload:** sort.
- **Headline:**
  - IBM's percentages are for the whole step, but only the sort phase speeds up, not the read and write phases.
  - Z Sort is not used for any of these reasons (ICE267I; SMF 16 field ICEZSRNU):
    - COPY, MERGE, INREC, OUTREC, OUTFIL, SUM, JOINKEYS or MODS/E15;
    - VSAM input or output;
    - **a file under 32 MB (reason 231)**;
    - a variable-length LRECL of 24 or less;
    - zHPF off for SORTWK;
    - no memory objects of **75% of the file size** (reason 240).
  - The ideal memory is twice the input size.
  - SMF 16 has a Z Sort section with phase-1 elapsed and TCB times.
- **Quality / independence:** an independent consultancy, IBM-hosted. It relies on IBM's numbers.
- **Maps to:** sort, choke/limits.
- **Useful to us:** a checklist for every ZSORT run:
  - input of at least 32 MB;
  - no INREC or OUTREC;
  - MEMLIMIT of at least 75% of the input;
  - ICE267I and SMF 16 kept as proof that Z Sort ran.
- **Also in:** ibm_material C3.

**AN04. "Mainframe vs server farm" five-year cost comparison**
- **Who / when:** Michael Benson (Enterprise Executive, 2016), relayed by Allan Zander (DataKinetics CEO) on Planet Mainframe, 2021-09-07.
- **URLs:** https://planetmainframe.com/?p=3116 (fetched) ; repeated in Arcati 2024, https://planetmainframe.com/?p=6581 (fetched).
- **Compared:** a z13 against "a bank of HP servers", described as "two similar performing platforms". The spec tables are images, so core counts could not be recovered. Benson's own comment says it was a Linux web server.
- **Workload:** a Linux web server.
- **Headline** (5 years, HP server farm against z13):

  | Line | HP servers | z13 |
  | --- | --- | --- |
  | Hardware | $2,299,451 | $2,793,371 |
  | Software | $1,807,406 | $416,883 |
  | Maintenance | $390,327 | $269,767 |
  | Operating cost | $9,052,749 | $6,979,693 |

- **Quality / independence:** the article's author sells mainframe tools. Benson says the article "misrepresented" his Linux-only study by generalising it to z/OS, and a commenter notes that the Linux-only scope favours the mainframe.
- **Maps to:** cost/energy.
- **Useful to us:** an example of scope drift between a study and its retelling.

**AN05. ITIC 2023 Global Server Hardware / Server OS Reliability Survey (IBM Z results)**
- **Who / when:** Laura DiDio, ITIC, 2023-08-22.
- **URLs:** https://itic-corp.com/?p=3107 (fetched) ; TechChannel coverage of 2024, https://techchannel.com/industry-news/server-reliability-survey-2024/ (not fetched; from search snippet; not verified).
- **Compared:** self-reported availability by platform, from nearly 1,900 organisations. "ITIC accepted no vendor sponsorship".
- **Workload:** none.
- **Headline:**
  - 96% of 200+ z16 respondents reported nine nines, which ITIC puts at "31.56 milliseconds of per-server annual downtime".
  - Power10 came in at eight nines.
  - Another year (snippet; not verified): Z/LinuxONE III at "0.0043 minutes" of monthly unplanned outage, against 1.10 minutes for the best x86.
- **Quality / independence:** an unsponsored survey, but self-reported.
- **Maps to:** choke/limits (availability only).
- **Useful to us:** none for performance. It shows the "reliability" axis is survey-based.

**AN06. "The Surprising Technology Economics of Mainframe vs. Distributed Servers"**
- **Who / when:** Dr Howard A. Rubin, Rubin Worldwide, about 2011, using Gartner 2010 unit costs. A 2023 update is linked to Broadcom.
- **URLs:**
  - Arcati 2024, which relays it: https://planetmainframe.com/?p=6581 (fetched).
  - Deck: https://www.slideshare.net/slideshow/the-suprising-technology-economics-of-mainframes-vs-distributed-servers/9096887 (not fetched).
  - IBM-hosted copy: https://public.dhe.ibm.com/software/uk/system-z/pdf/Mainframe_Economics.pdf (not fetched).
  - 2023 Broadcom report: https://www.mainframe.broadcom.com/hubfs/01.%20Mainframe%20Software/02.%20Assets/Cross-Segment/TACO/MSD_Technology-Economics-Report-May2023.pdf (redirects).
  - Arcati 2025: https://planetmainframe.com/arcati-2025/follow-the-leaders/ (not fetched; from search snippet; not verified).
- **Compared:** "mainframe-heavy" against "server-heavy" organisations, by correlation across companies.
- **Workload:** none measured.
- **Headline:**
  - A "29% cost advantage"; computational costs "may be 30% lower".
  - An insurance claim costs about $56 in IT, $36 less than server-heavy.
  - Under $600 per ATM against over $1,000.
  - The 2023 update (snippet; not verified): about 2,400 organisations; doubling scale cuts unit cost by ">60%" on mainframe, against about 20% for cloud and 10% for servers. Arcati 2025 says 67%.
- **Quality / independence:** correlational. No workload was metered.
- **Maps to:** cost/energy.
- **Useful to us:** cost per unit of work as a framing, but not comparable with measured runs.

**AN07. IDC business-value studies of IBM Z (2019-2020)**
- **Who / when:** IDC, sponsored by Broadcom and IBM, and by Rocket Software.
- **URLs:** https://mainframe.broadcom.com/hubfs/01.%20Mainframe%20Software/02.%20Assets/Cross-Segment/MF_IDC_Infographic_The%20Business%20Value%20of%20the%20Transformative%20Mainframe-1.pdf ; https://itjungle.com/2020/09/23/modernization-trumps-migration-for-ibm-i-and-mainframe-idc-says (not fetched; from search snippet; not verified).
- **Compared:** interviews with customers the sponsors selected.
- **Workload:** none.
- **Headline:**
  - A ">6:1 ratio of benefits to costs", and "$194 million per year" of revenue protected.
  - Modernisers save 12.5% on hardware capex, 5.8% on software opex and 4.6% on staffing opex.
- **Quality / independence:** sponsored, with no measurement. No IDC study from 2024-2026 was found.
- **Maps to:** cost/energy.
- **Useful to us:** none for measurement.

**AN08. Forrester Total Economic Impact (TEI) studies touching IBM Z**
- **Who / when:** Forrester, commissioned by vendors: IntelliMagic (July 2024), an older IBM System z study, and BMC (June 2025).
- **URLs:** https://tei.forrester.com/go/IBM/ZIntelliMagicforzOS ; https://www.ibm.com/support/pages/more-details-ibm-zenterprise-ec12-mainframes ; https://www.bmc.com/forms/the-roi-of-mainframe-modernization-analyst-report.html (all not fetched; from search snippet; not verified).
- **Compared:** composite-organisation financial models.
- **Workload:** none.
- **Headline:** ROI of 161%, 199% and 217%.
- **Quality / independence:** commissioned. A TEI holds no performance data. No TEI of LinuxONE or z16/z17 hardware against x86 was found.
- **Maps to:** cost/energy.
- **Useful to us:** none for measurement.

**AN09. Kyndryl "State of Mainframe Modernization" 2025 and the BMC Mainframe Survey 2024/2025**
- **Who / when:** Kyndryl and BMC.
- **URLs** (all not fetched; from search snippet; not verified):
  - https://www.kyndryl.com/ie/en/about-us/news/2025/09/mainframe-modernization-delivers-big-roi
  - https://www.itjungle.com/2025/09/15/positive-news-from-the-kyndryl-mainframe-modernization-report
  - https://planetmainframe.com/?p=6764
  - https://www.bmc.com/blogs/mainframe-survey-financial-services-key-takeaways/
- **Compared:** respondents' expectations.
- **Workload:** none.
- **Headline:**
  - Kyndryl, 500 leaders: expected ROI of 288% (modernise on the mainframe), 297% (integrate with cloud) and 362% (migrate off).
  - BMC: 90% still investing in 2024; in 2025, 72% reported increased general-purpose capacity.
- **Quality / independence:** vendor surveys.
- **Maps to:** none.
- **Useful to us:** none.

**AN10. "IBM brings Z to the rack: the real story is the data"**
- **Who / when:** Matt Kimball, Moor Insights & Strategy, 2026-07-13.
- **URL:** https://moorinsightsstrategy.com/field-notes/ibm-brings-z-to-the-rack-the-real-story-is-the-data/ (fetched).
- **Compared:** commentary on IBM's z17 claims.
- **Workload:** none.
- **Headline:**
  - "An IBM Z core is not an x86 core."
  - "The counts don't translate one-to-one."
  - Z "drives those cores at high utilization for long stretches, where x86 fleets are usually sized for peak load and average utilization sits considerably lower".
  - It repeats "up to 83 percent less energy" without critique.
- **Quality / independence:** a sympathetic analyst.
- **Maps to:** cost/energy, choke/limits.
- **Useful to us:** a quotable statement that core counts are not comparable without a utilisation model.

**AN11. "IBM targets energy-saving mainframes at Linux users"**
- **Who / when:** Ed Scannell, TechTarget, 2022-09-13.
- **URL:** https://www.techtarget.com/it-infrastructure/news/252524891/IBM-targets-energy-saving-mainframes-at-Linux-users (fetched).
- **Compared:** IBM's claim of "up to 75% lower energy" for LinuxONE Emperor 4 against "hundreds of x86-based servers".
- **Workload:** none.
- **Headline:** analyst scepticism about adoption, not about the numbers.
  - Frank Dzubeck: "not an opportunity I expect a lot of users to jump at".
  - Bob O'Donnell: "It may not become a mainstream approach".
- **Quality / independence:** trade press.
- **Maps to:** cost/energy.
- **Useful to us:** context only. No re-measurement.

**AN12. International Technology Group survey of mainframe and LAN cost (via EDUCAUSE CAUSE 1996)**
- **Who / when:** ITG, a consultancy, reported second-hand in 1996.
- **URL:** https://www.educause.edu/ir/library/html/cem/cem96/cem9638.html (not fetched; from search snippet; not verified).
- **Compared:** 250 sites, mainframe against LAN.
- **Workload:** none measured.
- **Headline:** $2,282 against $6,445 per user per year; 3 cents against 43 cents per transaction.
- **Quality / independence:** a consultancy survey, second-hand.
- **Maps to:** cost/energy.
- **Useful to us:** the historical origin of "cost per transaction" framing.
- **Also in:** academic C3.

### 3.5 Standard benchmarks (9 items)

**SB01. TPC-C on mainframes: "IBM has run the TPC-C test on MVS systems, but never released them"**
- **Who / when:** Tech Monitor / Computer Business Review, column credited to Timothy Prickett Morgan, 5 May 1998, quoting Ross Mauri (IBM VP, S/390 hardware).
- **URL:** https://www.techmonitor.ai/?p=92089 (fetched by the analysts and benchmarks strands; the academic strand saw a snippet).
- **Related:** the first TPC-C result ever was an IBM AS/400, not a mainframe, at 54 tpmC in September 1992. https://en.wikipedia.org/wiki/TPC-C (not fetched; from search snippet; not verified).
- **Compared:** the columnist's own estimate of a G4 9672 at "30,000 or so" tpmC, against more than 52,000 for a 16-way HP V2250.
- **Workload:** TPC-C.
- **Headline:** the quote above. The columnist infers: "That's why IBM focuses on the relationship instead of the hardware" and "By not running, and more importantly, not winning... IBM concedes defeat".
- **Quality / independence:** trade press. The figures are estimates, not results.
- **Maps to:** RMW/IO, cost/energy.
- **Useful to us:** citable evidence that the absence of TPC results is IBM's choice, not a lack of measurement.

**SB02. "The Truth About the TPC"**
- **Who / when:** ITPro Today.
- **URL:** https://itprotoday.com/early-versions/the-truth-about-the-tpc (not fetched; from search snippet; not verified).
- **Compared:** none.
- **Workload:** TPC in general.
- **Headline:**
  - "you probably won't see a TPC-C score published unless the vendor thinks it's in the company's best interest".
  - Database licences carry "DeWitt clauses" that restrict publishing benchmarks.
- **Quality / independence:** trade press.
- **Maps to:** cost/energy.
- **Useful to us:** check DB2 and z/OS licence terms before publishing any database results.

**SB03. SPECjbb2015 and SPEC CPU searches**
- **Who / when:** spec.org results database.
- **URL:** https://www.spec.org/jbb2015/results/res2022q3/jbb2015-20220624-00827.html. This is the Power10 result the search returned. It came from a search, not an exhaustive database query, and the note does not mark the page as fetched.
- **Compared:** IBM Power10 (753,796 max-jOPS, 96 cores) and x86. No IBM Z entry surfaced.
- **Workload:** Java server.
- **Headline:** no Z result found.
- **Quality / independence:** audited, but the absence finding rests on a search.
- **Maps to:** mixed grid.
- **Useful to us:** a SPECjbb-like Java run on both sides would have no audited Z comparator.

**SB04. "Robust power management in the IBM z13"**
- **Who / when:** IBM, about 2015.
- **URL:** https://www.researchgate.net/publication/282538028_Robust_power_management_in_the_IBM_z13 (not fetched; from search snippet; not verified).
- **Compared:** z13 power projection against measurement.
- **Workload:** SPEC CPU2006 used as a workload, not as a published score.
- **Headline:** the power-projection error was within about 2.3%.
- **Quality / independence:** IBM.
- **Maps to:** cost/energy.
- **Useful to us:** the nearest thing to a SPEC CPU number on Z. It shows IBM runs SPEC internally.

**SB05. Sort Benchmark records**
- **Who / when:** sortbenchmark.org, 1987-2023.
- **URL:** https://sortbenchmark.org/ (fetched; the full records page was checked).
- **Compared:** clusters, clouds and PCs. No entry in any category (Gray, Minute, Cloud, Joule, Penny, Datamation) names a mainframe, z/OS or s390. IBM Almaden's entries were Itanium2 Linux (2005 MinuteSort) and an SP cluster (2000).
- **Workload:** sort.
- **Headline:** the JouleSort 10^10-record records are held by desktop and laptop AMD machines running Nsort on NVMe:
  - AMR5Sort, Ryzen 5800U: 52.7 kJ, or 190K records/J (2023);
  - MendSort, Ryzen 7900: 59.3 kJ (2023).
- **Quality / independence:** audited, with published rules.
- **Maps to:** sort, cost/energy.
- **Useful to us:**
  - An audited sort-per-joule figure for a Ryzen-family PC. The Zen 3 5800U is the same generation as our 5800X.
  - There is no mainframe counterpart.
  - Use 100-byte records with 10-byte keys.

**SB06. MLPerf / MLCommons**
- **Who / when:** MLCommons.
- **URL:** https://mlcommons.org/?p=396 (search result; the inference datacenter table was not filtered).
- **Compared:** Red Hat, NVIDIA and Intel entries were found. No IBM Telum submission was found.
- **Workload:** AI inference.
- **Headline:** no Telum entry found.
- **Quality / independence:** audited, but the absence finding rests on a search.
- **Maps to:** AI inference.
- **Useful to us:** no third-party inference comparator exists for the AIU.

**SB07. HammerDB and pgbench on LinuxONE**
- **Who / when:** various.
- **URLs** (all not fetched; from search snippet; not verified):
  - https://developer.ibm.com/tutorials/postgresql-experiences-tuning-recomendations-linux-on-ibm-z/
  - https://hammerdb.com/blog
  - https://www.linkedin.com/pulse/linuxone-truly-engineered-system-oracle-workloads-benjamin-kwan
- **Compared:** no published LinuxONE against x86 TPROC-C comparison was found. IBM's PostgreSQL tuning tutorial has no x86 head-to-head.
- **Workload:** TPROC-C, pgbench.
- **Headline:**
  - HammerDB says NOPM is "the only one that should be used for a cross database comparison", and that the CPU governor (powersave) changes results.
  - IBM claims pgbench on PostgreSQL 9.6.1 gives "up to 2x more throughput per core than x86".
- **Quality / independence:** vendor claim (pgbench); tool guidance (HammerDB).
- **Maps to:** RMW/IO, choke/limits.
- **Useful to us:** HammerDB's note on the CPU governor applies to any x86 run: set it to performance.

**SB08. OpenBenchmarking.org (Phoronix Test Suite) uploads from LinuxONE Community Cloud**
- **Who / when:** community uploads, 2020-2022.
- **URLs:** https://openbenchmarking.org/result/2004234-NI-Z15ZVMSLE42&export=html ; https://openbenchmarking.org/result/2204081-JHAM-SERVERZ39 ; https://openbenchmarking.org/result/2109228-JHAM-PTSCOMP02&grr (not fetched; from search snippet; not verified; pages return 403).
- **Compared:**
  - "IBM z15 (8561) 2 VCPUs on LinuxONE Community Cloud", running SLES 12 SP5 or openSUSE Tumbleweed on z/VM 6.4;
  - z14 on the IBM Cloud KVM free tier;
  - z15 with Ubuntu 22.04.
- **Workload:** pgbench, SQLite, Redis/KeyDB, John the Ripper, nginx and others.
- **Headline** (pgbench, PostgreSQL 12.0, 2 vCPU):
  - Mostly-RAM read-only: 3,207.37 TPS;
  - read-write: 889.84 TPS;
  - Buffer read-write: 4,549.94 TPS;
  - Buffer read-only on another run (openSUSE, Kyber): about 69,400 TPS.
- **Quality / independence:** community uploads, reproducible with PTS. Figures not verified.
- **Maps to:** RMW/IO.
- **Useful to us:** the only same-size (2 CPU) public s390x standard-suite numbers. PTS could run the same tests on the Ryzen capped to 2 cores.

**SB09. Phoronix coverage of IBM Z**
- **Who / when:** Phoronix / Michael Larabel.
- **URL:** https://openbenchmarking.org/result/1603237-GA-1410225LI59&rdt&grr&export=html. This is the only Larabel-attributed hit, and it is a POWER8 system (search result).
- **Compared:** no Phoronix article benchmarking IBM Z hardware was found.
- **Workload:** none.
- **Headline:** absence.
- **Quality / independence:** absence based on search.
- **Maps to:** none.
- **Useful to us:** confirms that no mainstream independent benchmarking site has covered IBM Z.


### 3.6 Community and practitioner (14 items)

**CM01. golang/go#82520: "use CPACF PCC for X25519, P-384 and P-521"**
- **Who / when:** ashokpariya0 on GitHub; opened 26 Oct 2026.
- **URL:** https://github.com/golang/go/issues/82520 (fetched; the P-256 figures were spot-checked by a second reader).
- **Compared:** z15 (8561), z16 (3931) and z17 (9175). On each, the normal build is compared with a `purego` build. On z17, Go is also compared with OpenSSL 3.2.2 with PCC disabled.
- **Workload:** BenchmarkECDH (ns/op) and the cost of one scalar multiplication. Per the issue, the benchmark is key generation plus ECDH, median of 5 runs (a second reader fetched the issue on 2026-11-19).
- **Headline:**
  - P-256 ECDH, accelerated against pure Go (spot-checked):

    | Machine | Accelerated (ns/op) | Pure Go (ns/op) |
    | --- | --- | --- |
    | z17 | 100,468 | 319,715 |
    | z16 | 139,740 | 638,626 |
    | z15 | 123,998 | 440,240 |

    That is about 3-4.6x from acceleration.
  - X25519, P-384 and P-521 ECDH show no gain, because Go does not use PCC for them yet.
  - Cost per scalar multiplication on z17, Go against OpenSSL with PCC disabled: X25519 about 95 against 17 us; P-256 about 50 against 33; P-384 about 528 against 63; P-521 about 1,408 against 102.
  - OpenSSL with PCC is "~5-7x faster" for X25519.
  - P-384 ECDSA verify (BenchmarkVerify/P384) on z17 is 94.7 us accelerated against 1,145 us pure Go; the reporter notes that Go already uses KDSA for ECDSA (a second reader fetched the issue on 2026-11-19).
- **Quality / independence:** an open issue that anyone can reproduce with `go test -bench`. The contributor is IBM-adjacent. There are no x86 numbers.
- **Maps to:** crypto.
- **Useful to us:** the best public per-operation cross-check for our CPACF signature runs. The same benchmark can run on the Ryzen to give the x86 column that the issue lacks. Its z15/z16/z17 ordering also shows that generation-to-generation differences are not monotonic.

**CM02. Chen100E/zcrypt-s390x**
- **Who / when:** GitHub; undated (2026?).
- **URL:** https://github.com/Chen100E/zcrypt-s390x (fetched).
- **Compared:** claimed to be an "IBM LinuxONE Community Cloud instance (machine type 9175...)" with 2 cores, Ubuntu 22.04 and kernel 5.15. Results are compared against portable C (OpenSSL) and software Go.
- **Workload:** AES-GCM, SHA-2/3, ECDSA, Ed25519.
- **Headline:**
  - AES-256-GCM: 13,869 MB/s with CPACF KMA, against 138.78 MB/s for portable C.
  - SHA-256: 4,384 against 274 MB/s.
  - SHA-512: 6,877 against 218 MB/s.
  - SHA3-256: 8,776 against 66 MB/s.
  - ECDSA P-256: 46,425 signs/s and 20,774 verifies/s.
  - Ed25519 (software): 17,985 signs/s and 6,550 verifies/s.
- **Quality / independence:** LOW TRUST. The README calls a 9175 "LinuxONE III", repeats the same metric with different values, and has no date. It does publish `scripts/compare-openssl.sh`, which masks CPACF to give a software baseline.
- **Maps to:** crypto.
- **Useful to us:** the masking method is worth borrowing. Use its numbers only after reproducing them. Its P-256 sign rate (46K/s) is close to IBM's z15 figure (IB13, 48.7K/s), which is consistent but proves nothing.

**CM03. goboltbench**
- **Who / when:** Jan Kammerath; runs dated 2025-07-07.
- **URL:** https://github.com/jankammerath/goboltbench (fetched).
- **Compared:**

  | Machine | Storage | Time | Monthly cost |
  | --- | --- | --- | --- |
  | Marist LinuxONE 4 GB guest, RHEL 9.6 | | 2.826 s | $78.36 |
  | AWS c7g.large Graviton3 | io2, 8,000 IOPS | 1.850 s | $67.67 |
  | AWS c7a.large EPYC 9R14 | | 1.852 s | $92.95 |
  | Apple M1 Pro | | 12.04 s | |

  vCPU counts and Go versions are not stated.
- **Workload:** bbolt key-value inserts of 114,634 Shakespeare lines, with random users and SHA-256 hashes, on 16 workers.
- **Headline:** the times in the table. The author states no conclusion.
- **Quality / independence:** independent, with scripts published (Apache-2.0). It is a single run.
- **Maps to:** RMW/IO, crypto, cost/energy.
- **Useful to us:** the closest community analogue to our commit-throughput test. The M1 result shows the benchmark is bound by storage and fsync, so record the storage path and fsync policy on both sides.

**CM04. "z15 on-board compression"**
- **Who / when:** Michael MacIsaac on the LINUX-390 list, 10 Jun 2020.
- **URL:** https://www.mail-archive.com/linux-390@vm.marist.edu/msg72967.html (fetched).
- **Compared:** z14 (3906) against z15 (8561).
- **Workload:** gzip on a 1 GiB file of zeroes.
- **Headline:**
  - `gzip -c`: 7.603 s against 5.722 s.
  - `gzip -d`: 24.833 s against 6.150 s.
  - `dd` write: 49 against 675 MB/s, which is not a compression effect.
  - No gzip or zlib version and no DFLTCC enablement are stated. The author says he is "not a performance expert".
- **Quality / independence:** a practitioner's single run.
- **Maps to:** compression.
- **Useful to us:** a warning about method. Zeroes are DFLTCC's worst case (deflate 1.46x in IB08), and GNU gzip has its own deflate code with a separate DFLTCC patch. A gzip test and a zlib test on the same box can therefore disagree.
- **Related:** openSUSE `gzip-1.10-ibm_dfltcc_support.patch`, https://build.opensuse.org/projects/openSUSE/packages/gzip.19903/files/gzip-1.10-ibm_dfltcc_support.patch?expand=0

**CM05. zlib-ng s390 documentation**
- **Who / when:** the zlib-ng project.
- **URLs:** https://github.com/zlib-ng/zlib-ng ; https://fuchsia.googlesource.com/third_party/github.com/zlib-ng/zlib-ng/+/dc3b5de8cd7c7605f73f253b61fed7d39a33c036/arch/s390 (not fetched; from search snippet; not verified).
- **Compared:** DFLTCC against software, citing IBM figures.
- **Workload:** compression and decompression.
- **Headline:**
  - Compression speedup "as high as 110x"; decompression "15x".
  - DFLTCC is off by default (`WITH_DFLTCC_DEFLATE`, `WITH_DFLTCC_INFLATE`) and needs z15 or later. QEMU may never emulate it.
- **Quality / independence:** project documentation.
- **Maps to:** compression.
- **Useful to us:** zlib-ng builds need DFLTCC switched on explicitly. Without it, a "zlib-ng on z17" run is a software run.

**CM06. OpenJDK jdk-dev thread on DFLTCC output reproducibility**
- **Who / when:** OpenJDK jdk-dev, October 2025.
- **URL:** https://mail.openjdk.org/pipermail/jdk-dev/2025-October/010561.html (not fetched; from search snippet; not verified).
- **Compared:** DFLTCC output against software output.
- **Workload:** Java Deflater tests.
- **Headline:** the DFLTCC path "does not generate reproducible output" and writes output regardless of the flush parameter, which broke OpenJDK tests.
- **Quality / independence:** project mailing list.
- **Maps to:** compression.
- **Useful to us:** compare round-trip correctness and compression ratios, never byte-for-byte compressed output across platforms.

**CM07. torch-spyre#4799**
- **Who / when:** R3hankhan123, 23 Sep 2026.
- **URL:** https://github.com/torch-spyre/torch-spyre/issues/4799 (fetched).
- **Compared:** an s390x host against an x86 host, each with the same Spyre card.
- **Workload:** encoder attention.
- **Headline:**
  - CPU-side dispatch (launch H2D/D2H/HostCallback) was slower on s390x, while kernel time was equal.
  - The maintainer pointed to x86 PCIe gen5 against s390x gen4, and to the s390x box being shared at the time. The issue was closed as "working as designed".
  - The snippet's "~27x" was not visible on the fetched page.
- **Quality / independence:** an issue report.
- **Maps to:** AI inference, choke/limits.
- **Useful to us:** host-side overhead can swamp an accelerator's gains, so time the whole call path, not just the kernel.

**CM08. Hercules emulator performance folklore**
- **Who / when:** Wikipedia and mailing lists, 2002-2009.
- **URLs:** https://en.wikipedia.org/wiki/Hercules_(emulator) ; https://classiccmp.org/mailman3/hyperkitty/list/test-drb@ccmp.vtda.org/thread/OON4LSJPMJHYV2AQHM6LWWI4C2ILKHCY ; https://forums.theregister.com/forum/all/2009/09/25/turbohercules_goes_commercial/ (not fetched; from search snippet; not verified).
- **Compared:** x86 PCs emulating S/370 through z/Architecture, against real hardware.
- **Workload:** MIPS.
- **Headline:**
  - About 50-60 MIPS sustained on a fast dual x86, about 300 MIPS on an early-2009 PC, and over 1,200 MIPS in tight loops.
  - About 5 MIPS on a dual Athlon 1700+ in 2002.
  - About 20x slower than real hardware, per a 2009 comment.
  - Wikipedia notes a "lack of concrete, controlled performance comparisons".
- **Quality / independence:** anecdotal.
- **Maps to:** choke/limits.
- **Useful to us:** it answers the question "can my PC be a mainframe?".

**CM09. Hacker News threads on mainframe throughput and cost claims**
- **Who / when:** Hacker News, 2015-2025.
- **URLs:**
  - https://news.ycombinator.com/item?id=8945080 (2015; fetched).
  - https://news.ycombinator.com/item?id=28456469 and https://news.ycombinator.com/item?id=43620257 (not fetched; from search snippet; not verified).
- **Compared:** none.
- **Workload:** none.
- **Headline:**
  - The 2015 thread reacts to IBM's "CICS runs 1.1m transactions per second" with "That equivalence is bogus" and "pure PR mess".
  - A later thread says "mainframe is 1 30 core chip and HP is 12x 24 core chips".
  - Another notes that "the software is strictly licensed and you pay per MIPS".
- **Quality / independence:** opinion, with no measurements.
- **Maps to:** cost/energy.
- **Useful to us:** shows how outsiders react to vendor per-core claims. A per-core comparison needs its denominator spelled out.

**CM10. "Benchmarks for Linux Applications on S/390"**
- **Who / when:** Slashdot, 2002.
- **URL:** https://slashdot.org/story/02/01/20/202205/benchmarks-for-linux-applications-on-s390-zseries (not fetched; from search snippet; not verified).
- **Compared:** none.
- **Workload:** none.
- **Headline:** mainframe benchmarks try to remove I/O limits, and "MHz became very meaningless". No numbers.
- **Quality / independence:** forum.
- **Maps to:** choke/limits.
- **Useful to us:** context only.

**CM11. Martin Packer on overloaded zIIPs (IBM-MAIN)**
- **Who / when:** Martin Packer, IBM-MAIN, 2018.
- **URL:** https://groups.google.com/g/bit.listserv.ibm-main/c/q7zWPXa4ysc (not fetched; from search snippet; not verified).
- **Compared:** none.
- **Workload:** zIIP offload.
- **Headline:** an overloaded zIIP "could lead to performance problems... worse (!) than having no zIIP at all".
- **Quality / independence:** a practitioner (an IBMer posting to the list).
- **Maps to:** choke/limits.
- **Useful to us:** a Z-side choke mode to look for. If Java or zCX work on our slice is zIIP-eligible, record zIIP utilisation.

**CM12. "Mainframe On Cloud" thread (Wheeler, IBM-MAIN)**
- **Who / when:** Anne and Lynn Wheeler, IBM-MAIN, 15 Sep 2013.
- **URL:** https://go.mail-archive.com/ibm-main@listserv.ua.edu/msg18808.html (fetched).
- **Compared:** MIPS per processor across generations, Dhrystone-style.
- **Workload:** none.
- **Headline:**
  - z900 156 (2000), z990 281, z9 333, z10 469, z196 625, EC12 743 (2012).
  - "even IBM publishes results for hundreds of such benchmarks (although not for mainframes)".
- **Quality / independence:** practitioner.
- **Maps to:** choke/limits, cost/energy.
- **Useful to us:** gives the scale of per-processor growth across generations, and a practitioner's confirmation that IBM publishes no standard benchmarks for its mainframes.

**CM13. "Computing power comparison: mainframe versus Intel Xeon"**
- **Who / when:** Didier Durand, Media & Tech blog, 2016-07-26; also on LinkedIn.
- **URLs:** http://media-tech.blogspot.com/2016/07/computing-power-comparison-mainframe.html (fetched) ; https://www.linkedin.com/pulse/computing-power-comparison-mainframe-versus-intel-xeon-didier-durand (not fetched; from search snippet; not verified).
- **Compared:** relays HP's 2013 zRef result: 1,000 MIPS against one DL980 with 8x E7-4870.
- **Workload:** zRef.
- **Headline:** "1'000 MIPS of mainframe power can be replaced with better performances (on this benchmark) by a single server HP DL980".
- **Quality / independence:** practitioner blog, written by someone with a migration interest (his commercial position is not stated in the notes). It is the only surviving pointer to HP's paper and its document number.
- **Maps to:** batch+online mix, cost/energy.
- **Useful to us:** the route to VN01.
- **Also in:** zref Q1/Q3; academic B4.

**CM14. LinuxONE Community Cloud operational talks**
- **Who / when:** Marist and IBM, ECC 2017 and SHARE 2018.
- **URLs:** https://ecc.marist.edu/documents/745175/860856/LinuxONE+Community+Cloud+-+ECC2017.pdf/96b8ef5f-3c79-4ede-8e40-7d7f4de1a24f ; https://www.share.org/Events/Past-Events/Proceedings/Proceeding-Details/linuxone-community-cloud-the-adventure-continues-2 (not fetched; from search snippet; not verified).
- **Compared:** none. They report operational capacity: a two-way z/VM SSI with OpenStack, over 3,600 registrations, and over 800 virtual servers at peak. The SHARE talk covers z/VM HiperDispatch problems across 10 systems hosting about 1,000 Linux servers.
- **Workload:** none.
- **Headline:** none against x86.
- **Quality / independence:** operator talks.
- **Maps to:** choke/limits.
- **Useful to us:** background on the shared academic-cloud environments behind CM03 and SB08. HiperDispatch behaviour is one source of noisy-neighbour variance.

### 3.7 Consolidation case studies (7 items, plus cross-references)

Cross-references in other sections:
- AN01: the Signal65 estate model, the best-documented one.
- IB15, IB16, IB17: IBM's 23-server (2,944-core, IB16), 72-server (IB16) and 36-server (IB17) footprints.
- IB29: the z10 "1,500 x86 servers" claim.

**CS01. "Leveraging workload consolidation for lower IT costs"**
- **Who / when:** IBM IT Economics Consulting & Research, (c) 2020.
- **URL:** https://ibm.com/downloads/cas/G3QK9D8V (fetched, PDF text extracted).
- **Compared (fn1, the one measured test):** a LinuxONE III with 50 cores and 2,048 GB on z/VM and RHEL, running a "competitive database", against sixteen x86 systems, each with 28 Broadwell cores and 768 GB.
- **Workload:** a "materially identical order fulfillment database workload" at 48,974 tps.
- **Headline:**
  - 448 against 50 cores, about 9:1 (our arithmetic, analysts strand).
  - fn4: "core consolidation ratios ranging from 10 to 32.5 distributed cores to one IFL", taken from "20 IT Economics assessments". These are estimates.
  - 50% less power and 75% less floor space.
  - Not fetched; from search snippet; not verified: a Chinese-language IBM document gives ratios "from 4.6:1 to 54:1, based on 29 customer analyses". https://ibm.com/downloads/cas/MJDPLD24
- **Quality / independence:** vendor. "IBM's internal workload studies are not benchmark applications".
  - Energy: LinuxONE power was *estimated* with IBM's tool, while x86 power was *measured* under load. x86 utilisations come from 15 customer surveys, at $0.10/kWh and PUE 1.66.
  - LinuxONE III figures were projected from Emperor II data plus 10%.
- **Maps to:** cost/energy, RMW/IO (database OLTP).
- **Useful to us:** this is the clearest three-layer example: a measured run (about 9:1), consulting estimates (10-32.5:1) and estate models. Energy measured on one side and estimated on the other is a trap to avoid.

**CS02. "Choosing a platform for the Temenos Transact banking solution: LinuxONE III vs x86"**
- **Who / when:** Dr J C Yao and Susan Proietti Conti, IBM IT Economics, with Temenos's Simon Henman acknowledged. September 2020, document 86035686-USEN-00.
- **URL:** https://ibm.com/downloads/cas/MJDPLD24 (fetched by the vendors strand and read in full; benchmarks strand saw a snippet).
- **Compared:**
  - Measured: a LinuxONE III LT1 with 52 IFLs (over 3,000 TPS) against x86 measured on **Sandy Bridge** servers, on-premises and in public cloud. "x86 results were extrapolated for newer x86 chipsets".
  - TCO: LT2 against 16- and 32-core Skylake and 80-core Cascade Lake.
- **Workload:** Temenos Transact R19.08 (Java), a retail-bank online mix driven by JMeter via MQ.
- **Headline:**
  - Over 80 TPS per core on LinuxONE (over 100 at 500 TPS), against 28 for x86 on-premises and 8.6 for x86 public cloud.
  - Response times 58 ms against 160 ms against 240 ms.
  - 3-year TCO savings of 7% at 100 TPS (9 IFLs against 42 x86 cores), rising to 56-60% at 1,000-2,000 TPS.
  - Footnote 17: 1,476 x86 cores at 10-20% CPU against 120 IFLs at 89% for the same 130,400 TPS, with LinuxONE III "projected from actual LinuxONE Emperor II test results".
- **Quality / independence:** IBM-written. The x86 baseline is 2011-era and extrapolated. IBM gets a 40% discount, and non-production plus DR each add 100%.
- **Arithmetic** (our arithmetic, vendors strand):
  - 1,476/120 is 12.3 x86 cores per IFL.
  - The utilisation difference explains a factor of 4.5-8.9.
  - That leaves roughly 1.4-2.8x per busy core.
- **Maps to:** batch+online mix (online), cost/energy.
- **Useful to us:**
  - Copy the presentation: TPS per core and response time at several offered loads.
  - Footnote 17 shows where consolidation ratios come from.
  - Our 2-vs-2 runs at full load measure the residual per-busy-core factor, not the 12x.
- **Contradiction:** Temenos's own figure is 61 TPS per core on AWS (VN14).
- **Also in:** benchmarks_community item 20.

**CS03. "Nationwide Virtualizes With A Vengeance: 700 Servers Gone"**
- **Who / when:** Charles Babcock, InformationWeek, 15 Dec 2006.
- **URL:** https://www.informationweek.com/software-services/nationwide-virtualizes-with-a-vengeance-700-servers-gone (fetched).
- **Compared:** more than 700 small Intel and Unix servers moved onto two z9 machines with 20 IFLs each (40 in total), running SUSE guests.
- **Workload:** consolidation.
- **Headline:**
  - About 40 Linux guests per IFL; about 17.5 servers per IFL (our arithmetic, vendors strand).
  - About $30M saved over 3-4 years (an executive estimate).
  - IFLs at $95,000 list against about $10,000 per small server.
  - WebSphere licensing was a stated driver.
- **Quality / independence:** a journalist interviewing the customer, 15 months into the programme. No performance measurement.
- **Maps to:** cost/energy.
- **Useful to us:** the canonical "N servers per IFL" story. The ratio counts idle-heavy servers, not cores at equal load.

**CS04. LzLabs Software Defined Mainframe at Swisscom**
- **Who / when:** LzLabs and Swisscom, 2019.
- **URLs** (all not fetched; from search snippet; not verified):
  - https://www.theregister.com/2019/05/16/lzlabs_kills_swisscoms_mainframes/
  - https://www.computerweekly.com/de/feature/Swisscom-Mainframe-Abschied-ist-Kulturwandel
  - https://www.businesswire.com/news/home/20190516005366/en/
  - https://swisscom.ch/en/business/enterprise/themen/cloud/mainframe-cloud.html
  - https://www.fintechfutures.com/saas-paas/lzlabs-unveils-world-s-first-software-defined-mainframe
- **Compared:** about 2,500 MIPS and about 20 TB of mostly DB2 data, moved to x86 RHEL in one weekend night.
- **Workload:** production.
- **Headline:**
  - "comparable performance and stability" and "about half" the operating cost.
  - One performance problem was traced to a missing patch.
  - LzLabs' general claim is "thousands of transactions per second".
- **Quality / independence:** vendor and customer PR, with no figures.
- **Maps to:** cost/energy.
- **Useful to us:** proof that a move of this size happened. There are no numbers to compare against.

**CS05. "z/VM vs KVM" (Velocity Software)**
- **Who / when:** Velocity Software, a z/VM tooling vendor.
- **URL:** https://www.velocitysoftware.com/ZVMVSKVM.pdf (not fetched; from search snippet; not verified).
- **Compared:** z/VM against KVM density.
- **Workload:** none measured.
- **Headline:** KVM needs about 50% more engines, based on assumed peak-utilisation targets of 90% against 60%.
- **Quality / independence:** vendor. It rests on an assumption, not a measurement.
- **Maps to:** cost/energy, choke/limits.
- **Useful to us:** another place where the 90% figure is assumed. Our choke curves can show where each side actually degrades.

**CS06. "Centralized architecture beats distributed systems: the energy efficiency of IBM Z and zLinux"**
- **Who / when:** Bytes for Business blog, relaying IBM.
- **URL:** https://bytesforbusiness.com/en/centralized-architecture-beats-distributed-systems-the-energy-efficiency-of-ibm-z-and-zlinux/ (not fetched; from search snippet; not verified).
- **Compared:** "39 x86 servers with a total of 2,072 cores" against one LinuxONE Emperor 4 with 125 cores.
- **Workload:** consolidation.
- **Headline:** energy falls from 566,448 to 143,962 kWh a year.
- **Quality / independence:** an IBM internal study, reported second-hand.
- **Maps to:** cost/energy.
- **Useful to us:** another energy model. No metered re-test exists.

**CS07. "Global bank achieves 90% cost savings with mainframe offloading"**
- **Who / when:** Ververica.
- **URL:** https://www.ververica.com/blog/global-bank-achieves-90-cost-savings-with-mainframe-offloading (not fetched; from search snippet; not verified).
- **Compared:** an unnamed bank's batch, before and after a streaming offload.
- **Workload:** batch moved to streaming.
- **Headline:** from 8+ hours to about 3 hours.
- **Quality / independence:** vendor case study.
- **Maps to:** batch+online mix (batch), cost/energy.
- **Useful to us:** it contrasts with VN10. Redesigning batch for a new platform changes the result, so keep our programs constant across platforms.

### 3.8 Critiques and debunks (10 items)

**CR01. "Mainframe Rehosting: The Ugly Truth about Cost and Quality of Service"**
- **Who / when:** IBM Competitive Project Office / Eagle team, (c) 2014.
- **URL:** https://public.dhe.ibm.com/software/os/systemz/pdf/2015summit/Mainframe_Rehosting_Ugly_Truths.pdf (fetched, PDF text extracted).
- **Compared:** 97 Eagle TCO studies from 2007-2011, plus case studies:

  | Distributed side | Mainframe side | IBM's verdict |
  | --- | --- | --- |
  | 4 HP DL980 G7 (256 cores) | 2,800 MIPS Sysplex | "65% less cost!" |
  | 3 HP DL580 (2011), 60 cores | z800 (2002), 499 MIPS on 2.1 processors | "Despite a 9-year technology gap, the Intel platform still required 29x more processors" |
  | not stated | not stated | 41x more cores |
  | 8 POWER p550 cores | 88 MIPS on 0.24 processors | "33x" |

- **Workload:** CICS/COBOL estates.
- **Headline:**
  - "in 96% of mainframe rehosting cases, clients ultimately end up spending more".
  - It rebuts HP's 2012 "Mainframe Alternative Sizing" guide: "Can a 2-chip, quad-core x86-based Blade server really replace 3,000+ MIPS?" and "Real world use cases suggest this number is off by a factor of 10-20 times".
- **Quality / independence:** IBM rebutting a competitor. The comparisons are estate cases sampled by IBM.
- **Maps to:** cost/energy, batch+online mix.
- **Useful to us:** the direct counter to zRef (VN01). Two vendors reach opposite conclusions because one quotes a peak per-core performance ratio and the other an estate consolidation ratio (see CR03). Neither is independent.

**CR02. "Strange math": the critique of the z10's "1,500 x86 servers"**
- **Who / when:** Joerg Moellenkamp (a Sun employee), 2008-06-12.
- **URLs:**
  - Original: https://c0t0d0s0.org/oracle/english/2008/06/12/strange-math.c0t0d0s0.html (HTTP 410; archive unreachable).
  - Register forum: https://forums.theregister.com/forum/all/2008/02/26/ibm_z10_mainframe/ (not fetched; from search snippet; not verified).
  - Phoronix forum: https://www.phoronix.com/forums/forum/hardware/processors-memory/963498-ibm-z14-announced-support-added-to-llvm-clang?p=964732 (not fetched; from search snippet; not verified).
  - ESJ counter-view: https://esj.com/articles/2008/03/18/analysis-system-z10-drastically-alters-the-mainframe-market.aspx (not fetched; from search snippet; not verified).
- **Compared:** the arithmetic behind IBM's claim (IB29).
- **Workload:** consolidation.
- **Headline (from snippets; not verified):**
  - The 1,500 servers broke down as "125 idle failover backups plus 1,250 test, development and QA machines running at 5 percent average utilization".
  - The claim assumed "every x86 system is only loaded at 10%".
  - It implied "a $30 million hardware investment to substitute $1 million worth of x86 servers".
  - A forum post asked: "what would happen if 10 of the x86 servers started to do real work?"
  - An ESJ analyst called IBM's figure conservative.
- **Quality / independence:** a competitor's blog, read only through snippets.
- **Maps to:** cost/energy, choke/limits.
- **Useful to us:** the original version of the utilisation-model critique. Our choke tests ask its question directly.

**CR03. SHARE 2025 session 435, "Decoding Performance vs Consolidation Ratios: x86 vs IBM IFLs"**
- **Who / when:** SHARE Cleveland 2025; the speaker is not named on the page.
- **URL:** https://www.share.org/Events/Past-Events/Proceedings/Proceeding-Details/435-decoding-performance-vs-consolidation-ratios-x86-vs-ibm-ifls (fetched; abstract only).
- **Compared:** the method only.
- **Workload:** none.
- **Headline:** it separates "core performance ratios" in a "controlled, functionally equivalent environment" from "core consolidation ratios" across production, dev/test, HA and DR. "Benchmarks (performance ratios) describe what a system is capable of but not how it will be implemented or deployed."
- **Quality / independence:** conference abstract; no numbers.
- **Maps to:** cost/energy, and every workload as a method.
- **Useful to us:** the vocabulary for our report. Ours are performance ratios on a 2-core budget, never consolidation ratios. The content was not retrieved.
- **Also in:** analysts Q3; benchmarks section 3; vendors Q4.

**CR04. "Why Your CPU Capacity May Not Match Your Vendor Estimate"**
- **Who / when:** Cheryl Watson's Tuning Letter; author and date not shown on the page.
- **URL:** https://tuningletter.planetmainframe.com/?p=31088 (fetched).
- **Compared:** published MIPS ratings against what a given workload actually achieves, within Z.
- **Workload:** LSPR workloads.
- **Headline:**
  - "Most MIPS ratings today are simply based on the vendor's claims".
  - LSPR throughput is "units of work (jobs or transactions) divided by the processor busy time".
  - The #MIX rating "is not confirmed by running 20% of each workload".
  - "there could be a 40% or more variance between the published MIPS and what your workload would see".
  - LPAR overhead ranges from 2% to 25%.
- **Quality / independence:** independent consultancy.
- **Maps to:** every workload, as a method.
- **Useful to us:**
  - Report work per CPU-busy-second for each workload separately.
  - Never average them into a composite, because the spread is over 30% even on one architecture.
  - The 2-25% LPAR overhead applies to our shared slice.

**CR05. LSPR card goes public (1991)**
- **Who / when:** Tech Monitor (CBR staff), 25 Jun 1991.
- **URL:** https://www.techmonitor.ai/?p=181246 (fetched).
- **Compared:** an "Internal Use Only" IBM LSPR card, ZZ19-8333-4.
- **Workload:** none.
- **Headline:** critics called it "A very misleading document". It does not say which processors were measured and which were extrapolated, its data are undated, and it is output from a PC planning tool.
- **Quality / independence:** trade press.
- **Maps to:** cost/energy.
- **Useful to us:** background on why IBM's own capacity ratings are not independent evidence.

**CR06. "To MIPS or not to MIPS, that is the CP question"**
- **Who / when:** a SHARE session.
- **URL:** https://www.share.org/Events/Past-Events/Proceedings/Proceeding-Details/to-mips-or-not-to-mips-that-is-the-cp-question-4 (not fetched; from search snippet; not verified).
- **Compared:** MIPS as a metric.
- **Workload:** none.
- **Headline:** it presents MIPS, in some uses, as a "Misleading Indicator of Processor Speed".
- **Quality / independence:** practitioner conference.
- **Maps to:** cost/energy.
- **Useful to us:** supports avoiding MIPS in our write-up.

**CR07. "Break the Myth of MIPS per TPS of CICS"**
- **Who / when:** Ming Lu, LinkedIn and Enterprise Tech Journal, 2019.
- **URLs:** https://www.linkedin.com/pulse/break-myth-mips-per-tps-cics-new-approach-mainframe-ming-lu (dead: 403) ; https://mydigitalpublication.com/publication/?i=604583&article_id=3437642&view=articleBrowser (not fetched; from search snippet; not verified).
- **Compared:** MIPS-per-TPS conversions for CICS.
- **Workload:** CICS.
- **Headline:** unread. It is not known whether it mentions zRef.
- **Quality / independence:** practitioner.
- **Maps to:** batch+online mix.
- **Useful to us:** the only named critique of the MIPS-per-TPS step that zRef-based claims rely on (VN02's 8.18 factor). It needs retrieving.

**CR08. Omdia on the z16 and LinuxONE rack-mount configurations**
- **Who / when:** Omdia.
- **URL:** https://omdia.tech.informa.com/om029183/ibm-takes-mainframe-to-the-forefront-of-cloud-with-ibm-z16-and-linuxone-rack-mount-configurations (fetch returned 403; from search snippet; not verified).
- **Compared:** commentary on IBM's claims.
- **Workload:** none.
- **Headline:** IBM "has not provided the details of the exact server configuration used and has extrapolated the per core performance benchmarks to rack scale".
- **Quality / independence:** analyst.
- **Maps to:** cost/energy.
- **Useful to us:** an analyst naming the extrapolation trap.

**CR09. IBM Community thread comparing IBM Power processors with IBM Z / LinuxONE IFLs**
- **Who / when:** IBM Community.
- **URL:** https://community.ibm.com/community/user/discussion/comparison-ibm-power-processor-to-ibm-z-linuxone-ifl?hlmlt=VT (not fetched; from search snippet; not verified).
- **Compared:** none.
- **Workload:** none.
- **Headline:** an IFL-against-Power benchmark won't be found because "there's no sense in running it". Raw numbers are "Apples to Oranges".
- **Quality / independence:** community.
- **Maps to:** none.
- **Useful to us:** an example of the stance our measurements push against. Comparing at the level of a workload, not raw speed, answers it.

**CR10. Vaskevitch, client/server text (chapter 8)**
- **Who / when:** David Vaskevitch, in a 1990s client/server book.
- **URL:** http://home.ubalt.edu/tdarling/puad626_share/Vaskevitch/chap08.html (not fetched; from search snippet; not verified).
- **Compared:** a client/server application against a mainframe.
- **Workload:** not known.
- **Headline:** the client/server application ran "more than 12 times faster than the mainframe".
- **Quality / independence:** advocacy book; a single case.
- **Maps to:** cost/energy.
- **Useful to us:** the 1990s mirror image of IBM's consolidation claims. Single cases point both ways.


## 4. Mapping: our workload, prior work, gap

| Our workload | Closest prior work (catalogue IDs) | What the prior work gives | Gap our test fills |
| --- | --- | --- | --- |
| **Sort** (DFSORT with zSort against GNU sort and Java) | IB12, AN03, VN18, AC05, AC17, SB05, VN01/VN02 (zRefBR sort/merge) | Z Sort against z14 DFSORT: up to 20-30% elapsed, 40% CPU. MFX "up to 35%". An eligibility checklist (32 MB minimum, 75% memory objects, no INREC/OUTREC). Audited JouleSort records on Ryzen 5800U/7900 (190K records/J). zRefBR 68.1 min on z/OS against 6.21 min rehosted (vendor). | No published Z Sort against GNU sort or Java on x86. No Z Sort on/off comparison on the same machine. No mainframe Sort Benchmark entry. No 2-CP figure. |
| **Compression** (DFLTCC/zEDC against software zlib) | IB07, IB08, IB09, IB10, IB06, IB11, AC02, AC03, AC06, CM04, CM05, CM06 | Hardware against software on the same Z, at 1.46x-42x by corpus, level and buffer. A "110x" project claim. Compression ratios of 0.94-1.02. Elapsed and CPU at 2 CPs for compress-plus-encrypt (IB11). A reproducibility caveat (CM06). | No DFLTCC against x86 software zlib (or zlib-ng, or Intel QAT). No z16 or z17 DFLTCC GB/s figures. No decompression figures with a stated configuration. No per-core figure from a shared-chip slice. |
| **AI inference per transaction** (Telum AIU against CPU) | AC07, AC09, AC10, IB01-IB05, AN02, CM07, SB06 | 22x against a SIMD core on the same chip (z16). Batch 128/160 throughput headlines. "13 x86 cores" per accelerator (remote x86). Snap ML 3.5x at 6 against 6 cores. 7.5x at batch 1 between z16 and z17 (2 GCPs + 4 zIIPs). The public models: LSTM fraud, TabFormer, MegaCard. | No absolute batch-1 latency on z16/z17. No in-process AIU against x86 CPU (AMX/AVX-512) on the same model at the same batch size. No independent Telum benchmark and no MLPerf entry. |
| **Crypto signatures per payment** (CPACF against software) | IB13, CM01, CM02, AC06, AC11, IB31, IB26 | z15 CPACF P-256 at 48,660 signs/s and 19,365 verifies/s per CPACF. Go P-256 ns/op on z15/z16/z17. Cycle-normalised SIKE against Skylake (from the literature). A CPACF-masking method (`OPENSSL_s390xcap`). | No CPACF against x86 OpenSSL (AES-NI / AVX-512 IFMA) at matched OpenSSL versions. No z16 or z17 edition of IBM's crypto performance report. No signatures-per-transaction figure inside an OLTP path. |
| **Mixed grid** (document size x AI checks x signatures, throughput per core) | AC10, IB04, AN02, IB14, CS02, AC01 | CICS with in-transaction inference (extrapolated from 6 CPs). TPS per core as a unit (IB14, CS02). MegaCard plus LSTM/TabFormer as a public stack. | No grid study varying payload size, AI-check count and signature count. No per-core throughput surface on either platform. Nothing at 2 against 2 cores. |
| **CICS-like online under concurrent batch sort** | **VN01/VN02 (zRef)**, IB22, AC14, VN03, VN07, VN10, AC21, VN20 | zRef online TPS with and without zRefBR (1,672 against 1,409 TPS, at different user counts), and batch elapsed time with and without online load (16 against 13 min), rehosted side only. +15% CICS CPU per transaction from co-resident batch on zEC12. A batch rating half the online rating (Microsoft). The Linux noisy-neighbour literature. | No study measures both platforms under the same batch interference. No WLM against cgroups or plain-scheduler comparison. No z/OS-side zRef mixed figures. No response-time and CPU-per-transaction pair at equal cores. |
| **Random read-modify-write; commit/write throughput** | CM03, VN10, SB08, IB18, VN16, VN17, AC01, AC20, AC03 | bbolt inserts: LinuxONE guest 2.83 s against EPYC 1.85 s (storage-bound). 45 min against 8 h per-query latency anecdote. 2-vCPU pgbench on z15 (3,207 read-only, 890 read-write TPS). YCSB claims (1.9-2.4x, vendor). FitzGerald's MySQL and memory-write results. | No large-file random RMW benchmark on z/OS (VSAM or BDAM) against x86. No commit-rate (fsync/log-write) comparison with stated storage on both sides. |
| **Choke/limits** (memory, swap, GC) | AC01 (nesting collapse), IB23 and CS05 (assumed 90% against 40-60%), VN16 (ratio against clients), CM11, CR02, AN01 (assumed 80-95%) | Assumed operating points rather than measured ones. One measured collapse, MySQL from L1 to L2 under z/VM. A ratio that grows with concurrency. | No published stress-to-failure curve on either platform. No measurement of x86 degradation under memory pressure, swap or GC against Z at equal cores. The 90%-utilisation premise behind every consolidation model is untested. |
| **Cost/energy** | AN01, CS01, IB15-IB17, IB27, CS06, SB05, AN06, CR01 | Estate models built on utilisation and licences. Energy estimated on Z and measured or modelled (IDC QPI at 55%) on x86. Audited JouleSort records/J on Ryzen. | No metered energy for a fractional Z LPAR. No like-for-like energy per unit of work. Shared Z partitions generally offer no power metering, so CPU-seconds per unit of work is the practical common measure. |

## 5. Methods worth borrowing, and traps to avoid

### How others normalised across architectures

- **Equal cores, SMT/HT off, same kernel build** (AC01). This is the only independent precedent, and it matches our 2-against-2 design. State SMT state, clock and cache on both sides. Note any cap that disadvantages one side: WSL2 limited to 2 GB hurts x86 on memory-bound work (analysts strand).
- **Work per CPU-busy-second for a named workload** (IB20, CR04). This is LSPR's own unit, an ITR.
  - Report it per workload, never as a #MIX-style composite: Watson shows a spread over 30% between workloads on one architecture.
  - Examples: records sorted per CPU-second, MB/s deflated per core, signatures/s per core, inferences/s at a stated batch size, commits/s per core.
- **TPS per core across several offered loads, with response time** (CS02, IB14). A scatter of TPS per core against offered TPS shows where each side bends.
- **Fixed offered load, compare cores needed** (AN01's measured run). At small scale this means: hold the request rate constant and report the CPU% each side needs.
- **"x86 cores equivalent to one accelerator"** (IB04 fn1, AN02). A unit for the AI grid, but only if both sides run in-process.
- **Cycles and wall time reported separately** (AC11). The z17's higher clock and the Ryzen's boost behaviour otherwise get conflated.
- **Per dollar** (CM03 gives monthly cloud prices; AN01 and CS02 give TCO). Use only with stated assumptions: utilisation, licence model, production/non-production split, $/kWh and labour.
- **Per watt** (SB05 JouleSort records/J). An audited PC-side figure exists; no Z-side counterpart does.
- **MIPS/LSPR equivalence: do not use it.**
  - The LSPR is Z-only by IBM's own statement (IB20).
  - MIPS-per-x86-core rules vary widely by source (VN05-VN07, VN09, VN12), and every "MIPS-equivalent" is TPS times an undisclosed ratio: TmaxSoft about 8.18 per TPS, Heirloom about 14.9 (our arithmetic in VN02 and VN08).

### Evidence that an accelerator actually ran

- **Z Sort:** ICE267I and SMF 16 ICEZSRNU (AN03, IB12).
- **CPACF and NNPA:** SMF 30 counters (IB21).
- **DFLTCC:** the `dflt` flag in /proc/cpuinfo, and `DFLTCC_LEVEL_MASK` (IB07, IB09). zlib-ng also needs `WITH_DFLTCC_*` (CM05). GNU gzip has its own patch (CM04).
- **Same-box software baseline for crypto:** mask CPACF with `OPENSSL_s390xcap` (IB31, CM02).

### Reusable inputs

- Compression corpora: the Canterbury corpus files (IB07); zeroes, random ASCII, Dickens and vmlinux (IB08).
- Sort record format: 100-byte records with 10-byte keys (AC17, SB05). The IBM grid adds 80 B-27 KB records, 8-500 B keys and 100K-100M records (IB12).
- Public AI and OLTP code: github.com/IBM/ai-on-z-fraud-detection, github.com/IBM/TabFormer and github.com/IBM/megacard-standalone (AC09, AN02).
- Cross-checks: `go test -bench` (CM01) and the Phoronix Test Suite (SB08).

### Statistics and reporting

- Use repeats, confidence intervals and t-tests (AC01).
- Report variance, because a shared LPAR has noisy neighbours, RNI shifts and HiperDispatch effects (AC12, IB22, CM14). Public measured runs are short (AN01: 3 minutes of steady state) and none reports variance.
- Report the compression ratio alongside speed (AC06, IB08).
- Report both response time and CPU per transaction for online-under-batch (IB22).
- Report the victim's p99 and the batch job's slowdown together (AC21, VN20).
- State tuning on both sides (IB18), and set the Ryzen CPU governor (SB07).
- Name both CPUs exactly (VN15).

### Traps to avoid

- **Remote x86 plus network delay.**
  - IB01 added 60 ms with `tc-netem`.
  - AN02 and IB04 fn1 run inference remotely on the x86 side.
  - IB14's x86 side paid an SSL hop to Db2 on z/OS.
  - VN15's latency test used in-box HiperSockets against Ethernet.
  - Keep both sides co-located, or state the hop.
- **Estate utilisation models.**
  - AN01: 2,688 of 2,944 cores are modelled non-production at 7-15%.
  - CR02: idle failover and test servers at 5-10%.
  - CS02 fn17: 10-20% against 89%.
  - CS03: idle-heavy servers per IFL.
  - Our results are performance ratios, never consolidation ratios (CR03).
- **Batch-size choices.** IBM's AI headlines use batch 128-160 with threads pinned one per chip (AC09, IB02, IB04). Batch 1 is a different regime, so report the batch size with every inference figure.
- **Cross-generation baselines.**
  - Z Sort against z14 (IB12); compression against z14 zEDC Express (IB06); ECC against a z14 Crypto Express6 card (AC06).
  - x86 baselines that are old or extrapolated: Sandy Bridge in CS02, Haswell in IB14, "Xeon Gen 2 and older" in IB15, Broadwell in CS01.
  - Compare on the same machine (on/off) and against a current x86 part.
- **Linear extrapolation from small LPARs:** 6 CPs to 200 CPs (AC10), 1 chip to 32 chips (AC07), 1 thread to 450 billion/day (IB02). A measured 2-CP figure must not be scaled the same way.
- **Different software on the two sides:** Fujitsu Enterprise Postgres against PG11 (VN16); Temenos releases (VN14, CS02). Hold the program constant.
- **Headline against footnote.** The zCX paper's "44%/54% more" for Z matches the x86 shortfall in its own TCO-model settings, which give Z 1.79x and 2.16x (IB14): "X% more" and "X% less" are not the same ratio, so recompute from the footnote. "450 billion/day" has two different footnotes (IB02-IB04). PUE varies from 1.55 to 1.67 across IBM pages. Cite the footnote.
- **Energy estimated on one side, measured on the other** (CS01, IB17).
- **Slice size set by one side.** Microsoft's 2003 comparison used Linux on one zSeries CPU (VN13). State our slice size and that the LPAR is shared, and expect the objection.
- **Corpus and tool choice.** Zeroes understate DFLTCC (CM04, IB08). gzip and zlib take different DFLTCC paths. Small Python buffers throttle the hardware (IB09).
- **Shared accelerators.** One AIU per chip on z16, eight per drawer on z17, and one DFLTCC unit per chip (IB05, AC02). A 2-CP slice does not own them, so contention from other tenants is possible.
- **Host-side overhead.** Time the whole call path, not just the accelerator kernel (CM07).

## 6. Open gaps no one seems to have published

The searches found no published work on any of the following. That is a search-based absence, worded as "none found", not "none exist".

1. **Accelerator against x86 software, same workload.**
   - DFLTCC against x86 zlib or zlib-ng.
   - CPACF ECDSA/EdDSA/AES against x86 OpenSSL with AES-NI or AVX-512 at matched OpenSSL versions.
   - Telum AIU against x86 CPU inference (AMX/AVX-512) in-process at batch 1.
   - Z Sort against GNU sort and Java sort.
   - No published accelerator figure found runs the same code in-process, at the same batch size, on Z and on x86.
2. **Absolute per-transaction (batch 1) inference latency on z16/z17.** IBM gives only relative figures (7.5x, -48%).
3. **Z Sort on and off on the same machine.** IBM's baseline is z14.
4. **Online response time and CPU per transaction under concurrent batch, measured on both z/OS (WLM) and Linux (scheduler/cgroups) at equal cores.** zRef measured only the rehosted side; REDP-5170 measured only the Z side.
5. **A mixed grid** (payload size x AI checks x signatures) giving a per-core throughput surface on either platform.
6. **Random read-modify-write on large files and commit throughput,** on z/OS data sets against x86 files or databases, with storage stated on both sides. The only numbers are a storage-bound community run (CM03) and an anecdote (VN10).
7. **A measured choke curve**: throughput and latency against utilisation, memory pressure, swap and GC, on both sides. This would test the 80-95% against 7-50% utilisation premise that every consolidation model depends on.
8. **Any repeated, variance-reported, raw-output-preserving Z-against-x86 run since 2014.** FitzGerald (AC01) is the last with CIs. No public study reports repeat counts on a shared LPAR.
9. **Measurements at 2 CPs on current Z hardware.** The nearest are IB11 (2 CPs, z14/z15, compression and encryption), IB03 (2 GCPs + 4 zIIPs, relative only) and SB08 (2 vCPU z15 under z/VM, community).
10. **A z16 or z17 edition of IBM's cryptographic performance report.** Only the z15 edition (IB13) was found.
11. **Any independent reproduction of an IBM x86 comparison, or of zRef.** None was found in any strand.

## 7. Search log (merged from the six strands)

All strands used WebSearch and WebFetch only. PDFs that WebFetch could not parse were downloaded and read with pdftotext. No host connections were made, nothing was run, and nothing was posted.

### Absence findings and how each was checked

| Claim | How it was checked | Strength |
| --- | --- | --- |
| No audited **TPC** (C/E/H/DS) result for an IBM mainframe | Targeted searches: "TPC-C result IBM mainframe ES/9000 history", "TPC-C Unisys ClearPath/Fujitsu mainframe", "why IBM does not publish TPC/SPEC". The Unisys and Fujitsu hits were Windows NT/Intel or open servers. Backed by IBM's 1998 statement (SB01) and its 2003 statement (VN13). | Search-based, not an exhaustive tpc.org query. |
| No **SPEC** (jbb/jvm/CPU) submission for IBM Z | spec.org-targeted searches for SPECjbb2015 and SPEC CPU on z196/zEC12/z13 (extended). Only Power10 and x86 surfaced (SB03); SPEC CPU2006 appears only as an internal workload (SB04). | Search-based. The analysts strand did not query spec.org directly. |
| No **MLPerf** Telum entry | Search "MLPerf inference IBM Telum z16 results". Only Red Hat, NVIDIA and Intel entries were found (SB06). | Search-based. The MLCommons table was not filtered. |
| No **Sort Benchmark** mainframe entry (1987-2023) | Fetched sortbenchmark.org and checked the full records page across the Gray, Minute, Joule, Cloud, Penny and Datamation categories (SB05). The academic strand also searched for AlphaSort/Datamation and DFSORT. | Checked against the primary page. |
| No **HammerDB** LinuxONE against x86 comparison | Search "HammerDB LinuxONE x86 comparison PostgreSQL TPROC-C" (SB07). | Search-based. |
| No **Phoronix** article on IBM Z | Searches "Phoronix IBM Z s390x benchmark", "phoronix z15/z16/LinuxONE Larabel" and "phoronix s390x DFLTCC" (SB09). | Search-based. |
| No peer-reviewed Z against x86 same-lab paper since 2014 | Academic queries 4, 6, 14, 16, 21, 25, 29 and 30 (below), plus analysts query 25. | Search-based. arXiv full text was not searchable with these tools. |
| No independent Telum against x86 inference benchmark | Benchmarks strand queries on zDLC, ONNX and fraud latency, and on the z16 19x/20x footnote. IBM strand query 23. | Search-based. |
| No WLM against cgroups study | Benchmarks strand "noisy neighbour cgroups vs WLM" (extended). IBM strand query 17. Academic queries 32 and section 5. | Search-based. |

### Queries by strand (condensed)

**Academic (32 queries)**
- **Accelerator design papers:** DFLTCC/ISCA, Telum/ISSCC, JRD SORTL, ISCA DOI, Telum II Hot Chips, ICS 2025, IEEE Micro Telum, ISSCC 2022 and 2020.
  - Dead ends: no SORTL paper; no z15 ISSCC 2020 paper; no IEEE Micro Telum paper.
- **Linux on Z against x86 in IEEE/ICPE/CMG:** vendor Redpapers only. CMG proceedings sit behind s2member links.
- **arXiv s390x:** dead end apart from 2608.21393.
- **LinuxONE Community Cloud:** operations talks only.
- **Theses:** Binghamton found. A Java/DB2 thesis search was a dead end (low-quality journals). Hadoop/Spark on Z and Hyperledger on LinuxONE were dead ends.
- **Crypto:** a JRD paper on z14 CPACF was a dead end; the s390x PQC search found AC11.
- **Other:** independent DFLTCC benchmarks (none); 1990s rightsizing; z/VM against KVM density; AlphaSort; Aman WLM; TPC/SPEC absence; CMG batch/online.
- **Fetch failures:** researcher.ibm.com (307 to 404); dl.acm.org (403); aminer (header only); publikationen.uni-tuebingen.de (ECONNREFUSED).

**IBM material (26 queries)**
- **Searches:** z17 footnotes; zEDC numbers (qualitative only); Z Sort; z16 300B footnote; LinuxONE 5 footnotes; the z16 fraud PDF; Emperor 4 energy; the "13 cores" claim (found later on the AI Toolkit page); LSPR cross-architecture (no such statement exists); CPACF GB/s; LinuxONE III OpenSSL/pgbench/Java footnotes; Roehm proof points; Snap ML; zDLC latency; z15 SMF compression; pervasive-encryption overhead; Telum 116k.
- **Dead ends:**
  - No z16 or z17 edition of the crypto performance report.
  - "Practical Migration from x86 to LinuxONE" ratios not retrieved.
  - No quantitative mixed-workload study.
  - No absolute batch-1 latency.
  - The z17 newsroom URL returned 404.
  - The HigherLogic/S3 downloads (z15 compression deck, Roehm PDFs) would not follow.

**Vendors, rehosting and consolidation (21 queries)**
- **Searches:** Heirloom 15,000 TPS (exists only as MIPS); LzLabs TPS (none); Azure MIPS per vCPU; LinuxONE cores per IFL; MFX/DMExpress against Linux sort (none); AWS Micro Focus; TmaxSoft; Temenos; MongoDB LinuxONE; MFX zSort.
- **Dead ends:** Citi, Banco and Sparkassen case studies; Blu Age batch numbers; LzLabs at Northern Trust; Dual Run numbers; Oracle per-core; EDB (Power, not Z); SI before/after numbers.
- **Fetch failures:** AWS blog bodies render client-side; the CAF page redirects; the AzureCAT original needs a login; us.tmaxsoft.com failed DNS; the SHARE linklings PDF returned 403.
- **Not searched:** Raincode, NTT Data, Astadia and Modern Systems individually; Broadcom, BMC and Rocket; Db2 z/OS against Db2 LUW; Kyndryl.
- **Excluded:** a Dutch-government example ("47 blades to 11 IFLs, 80% energy"), because only a Chinese-language PDF snippet with no URL was found.

**Analysts, TCO and critiques (29 queries)**
- **Searches:** Edison Group (no LinuxONE study; only Oracle-funded SPARC work, https://www.oracle.com/jp/a/tech/docs/technical-resources/edison-sparc-s7-wp.pdf, not fetched); Solitaire Interglobal (no original report; only second-hand IBM citations); Forrester TEI; Wikibon/Floyer (no mainframe TCO); Watson on MIPS; Register critiques; TPC absence; Rubin/Arcati; IDC 2024-25 (none newer than 2020); ITIC; MIPS rules of thumb; Heirloom TPC-C; Clabby (only old IBM-hosted papers); Phoronix (dead end); LSPR caveat; sceptical analysts; HN; the 32.5 cores/IFL ratio; the z10 critique; BMC and Kyndryl surveys; Principled Technologies, ESG and Quark+Lepton (dead end); academic SPEC (dead end); Telum critique; Gartner (second-hand only); Rubin critique (none).
- **Fetch failures:** c0t0d0s0 (410); web.archive.org (not reachable by the tool); Omdia (403); the share.confex.com "TCO Lessons Learned" PDF (DNS); the Broadcom Rubin PDF (redirect).

**Standard benchmarks and community (about 45 calls)**
- **Searches:**
  - Standard suites: TPC-C history, SPECjbb/SPEC CPU, MLPerf, HammerDB, Sort Benchmark, TPC-C on Unisys/Fujitsu, why no TPC/SPEC, LSPR.
  - Phoronix and OpenBenchmarking.
  - Compression: zlib PR 410, zlib-ng GB/s, gzip DFLTCC.
  - Crypto: openssl speed CPACF against AES-NI, KDSA P-256, OPENSSL_s390xcap.
  - Runtimes: Telum zDLC, OpenJ9 against HotSpot, s390x ns/op on GitHub, 2ndQuadrant pgbench, LinuxONE pgbench 2023-25.
  - Community and press: HN, Hercules, Martin Packer, moshix, SHARE z16 performance update, LinuxONE MongoDB YCSB, Temenos, the Microsoft 2003 benchmark, noisy neighbours against WLM, the Emperor 4 75% critique, Planet Mainframe and TechChannel numbers.
- **Dead ends:** OpenBenchmarking pages (403); researcher.ibm.com Telum page (404); ibmsystemsmag.com (DNS); Kacprzak and Kolasinski PDF (403).
- **Nothing found for:** a Phoronix Z article; a Reddit thread with numbers; a moshix video; a MLPerf, SPEC or Sort Benchmark Z entry; a WLM against cgroups study; OpenJ9 against HotSpot on s390x; an independent Telum benchmark.

**zRef (14 queries)**
- **Searches:** zRef HPE; zRef TPC-E CICS COBOL MIPS; zRef GitHub s390x; "4AA4-2452ENW" (spam hits only); "Mainframe Alternative Sizing" DL980; Microsoft 288/170; TmaxSoft zRef; zRef Superdome/Micro Focus; Asysco 18,000 MIPS; LzLabs zRef (x2); other meanings of zRef; MIPS-per-core critiques; zRefCICS/BL860c.
- **Dead ends:**
  - The HP PDF (404) and hpe.com psnow (timeout).
  - Wayback (not reachable by the tool).
  - manualzz (403) and pdfslide (now redirects to a different domain).
  - The CAF page (redirect) and its GitHub source (404).
  - techcommunity (login); info.lzlabs.com (ENOTFOUND); LinkedIn articles by Ming Lu and Dale Vecchio (403).
- **Not tried:** the "zRefBU" exact phrase, HPE community blogs, the asysco.com/ibm/ whitepaper, and the Micro Focus/OpenText archives.
- **Best remaining route to HP's original:** a browser Wayback lookup of `h20195.www2.hp.com/V2/GetPDF.aspx/4AA4-2452ENW.pdf`, or the manualzz mirror opened in a browser.

### Follow-ups the strands suggested but did not run

- Exhaustive tpc.org, spec.org and MLCommons queries by vendor and architecture.
- IEEE Xplore and ACM DL author searches (Jacobi, Lichtenau, Abali, Bradbury), and the JRD vol. 64 table of contents.
- ProQuest, DiVA and EThOS thesis searches.
- The CMG MeasureIT archive.
- The Hot Chips 2021 Telum slides.
- A manual download of the IBM z16 Proof Points and SHARE Böblingen decks (IB19).
- Retrieval of the Kacprzak and Kolasinski paper (AC20) and Ming Lu's MIPS-per-TPS article (CR07).
- Retrieval of HP's 2013 zRef paper (VN01) and the Asysco whitepaper (VN03).

## 8. Item count by section, and choices

| Section | Items |
| --- | --- |
| 3.1 Academic | 22 |
| 3.2 IBM material | 31 |
| 3.3 Vendors, ISVs, cloud, rehosters | 20 |
| 3.4 Analysts and independent testers | 12 |
| 3.5 Standard benchmarks | 9 |
| 3.6 Community and practitioner | 14 |
| 3.7 Consolidation case studies | 7 |
| 3.8 Critiques and debunks | 10 |
| **Total (de-duplicated)** | **125** |

Method notes:
- The research was split into five source-type strands plus a dedicated zRef strand.
- Section bodies use bullets and tables, because each catalogue entry carries the same fields.
- Items that fit two groups are placed in one group, with cross-references to the other:
  - Signal65 in analysts (AN01), cross-referenced from consolidation;
  - the Watson and Walker Z Sort reprint in analysts (AN03), as an independent consultancy even though IBM hosts it;
  - IBM's "Ugly Truths" in critiques (CR01);
  - IBM's z10 claim in IBM material (IB29), with its critique in CR02;
  - the Microsoft/VeriTest 2003 benchmark in vendors (VN13);
  - REDP-5170 in IBM material (IB22), although the academic strand found it;
  - the Ververica offload in consolidation (CS07).
- Near-duplicates and minor related items are merged into one entry each:
  - the zDLC blog and Software AG reprint into IB03;
  - the Joy Deng blog into IB04;
  - the IBM Docs zEDC pages into IB07;
  - the DFSORT support page into IB12;
  - the IMS encryption blog into IB13;
  - the Linux on Z FAQ into IB17;
  - SHARE 8916 into IB22;
  - ISSCC 2015 into AC08;
  - the Telum II announcement into AC09;
  - EDB Power11 into VN16;
  - the PipeSort, Mexican bank and Connect items into VN18;
  - Infosys into VN19;
  - Kwan's pgbench claim into SB07;
  - the CMG 2018 and Astadia MIPS rules into VN06.
- Where strands disagreed on fetch status, the strongest status any strand achieved is recorded, with that strand named. This applies to Tech Monitor 1998, the zlib PR and LWN, REDP-5170, the TmaxSoft study and IBM/Temenos. No item that no strand fetched was upgraded on the strands' evidence alone.
- On 2026-11-25 a sample of 44 entries was re-checked against their cited sources (one source could not be retrieved), and the eight that differed were corrected: AC11, IB14, IB15, IB17, IB23, IB24, VN10 and VN13. Figures not found in the cited source were removed, and sources read then are marked "fetched 2026-11-25" (also IB25). Entries still marked "from search snippet" are not verified.
- Two Go-issue details (the "key generation plus ECDH, median of 5" wording and the P-384 ECDSA verify figures) are not in the strand notes; they were read from the issue on 2026-11-19.
- Both dates are recorded where the notes conflict: Signal65 (July against October 2025) and the TmaxSoft study (about 2020-21 against the ©2023 PDF footer).
- Two zRef ambiguities are flagged, not resolved: 1,409 TPS ran with a different user count from 1,672 TPS, and the 13-min no-CICS zRefBR reference matches the concurrent-batch scenario rather than the standalone 6.21 min.
- zRef is ranked second, behind FitzGerald. zRef is the closest analogue to our online-under-batch test, but it is vendor-run and unreproducible, while FitzGerald is the only independent equal-core head-to-head.
- Items with unknown contents (AC20, CR07, IB19) are included as catalogue entries flagged for retrieval rather than dropped. Left out are the Dutch-government example (no URL recovered) and the Belgian Ministry of Finance "400K IOPS" case-study mention on the LinuxONE 5 page (the IBM strand called it uncontrolled; it appears only under Open gaps context in the notes).
- SB05's Ryzen 5800U is mapped to the 5800X by name and generation only. Their performance is not equated.
