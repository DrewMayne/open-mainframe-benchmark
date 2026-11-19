# Roadmap

| Milestone | Content | Status |
| --- | --- | --- |
| M0 | Plan, spec v0.1/v0.1.1 (two tracks), pre-registered hypotheses, prior-art review, platform facts, FDR template | done |
| M1 | **Java track** skeleton on x86 Linux: data generator, 2 online transactions, B2 sort, load driver, run manifest, golden-output test; 5-run repeatability shown | next |
| M1b | Java track complete: all online transactions, batch suite, accelerated extension; spec v0.2 with the transaction mix table | planned |
| M2 | **COBOL track:** CardDemo port to GnuCOBOL (online + batch) and accelerated extension on x86, bare metal and VM; track equivalence passes | planned |
| M3 | s390x leg (both tracks) on a LinuxONE / Linux on Z platform whose terms permit benchmarking; H1-H5 phase 1 results | access being arranged |
| M4 | Technical report (preprint) with all raw data | planned |
| M5 | Phase 2: x86 with QAT/AMX; z/OS with CICS, VSAM, DFSORT and WLM (COBOL track on Enterprise COBOL; Java track with and without zIIPs); MIPS-to-x86-core calibration | access being arranged |
| M6 | Peer-reviewed reproducibility: submit the report and kit for artifact evaluation (e.g. ACM ICPE), so that independent reviewers re-run it | planned |
| M7 | Open cost model: a separate report in which every cost assumption is a visible, editable input, fed by the measured ratios. No hidden utilisation or licence assumptions | later |
| A1 | Analytics track (TPC-H-derived, compliance-ready) on PostgreSQL x86: SF 1 and 10, power and throughput tests, refresh streams, answer validation, ACID tests (with power-loss durability on our own hardware) | planned |
| A2 | Analytics on Linux on Z, then Db2 for z/OS (phase 2); queries as batch underneath the online track | access being arranged |
| T1 | TPC-Pricing / TPC-Energy methods applied to the FDR pricing appendix and energy section (x86) | planned |
| T2 | TPC-DI-derived batch ETL module on PostgreSQL x86, then Linux on Z / z/OS | planned |
| T3 | TPC-E-derived OLTP module beside the online track (x86 first) | planned |
| T4 | TPCx-V-derived consolidation module (KVM on x86; KVM or z/VM on Z) | access being arranged |
| Stretch | An audited run of an actual TPC benchmark (e.g. TPC-E or TPCx-AI) on Z and x86, as a separately sponsored effort with a TPC-certified auditor | needs a sponsor |

## Other TPC benchmarks, ranked (derived, labelled, never vendored)
The high-priority rows are now adopted as optional modules (SPEC 1.7; milestones T1-T4). The others stay candidates.
Revisions as on tpc.org, 2026-11-19: TPC-H 3.0.1, TPC-E 1.14.0, TPC-DI 1.1.0 (obsolete), TPCx-V 2.1.10, TPC-Energy 1.5.0, TPC-Pricing 2.9.0.

| Benchmark | What it is | Why it matters here | Priority |
| --- | --- | --- | --- |
| TPC-E (adopted) | Brokerage OLTP: complex, read-heavy transactions | The closest standard analogue to zRef and to CICS-style work; a familiar OLTP yardstick beside our CardDemo track | high |
| TPC-DI (adopted; obsolete at TPC) | Data integration: batch ETL from multiple source formats, including fixed-width files, into a warehouse | The nightly batch load, in its most mainframe-like form | high |
| TPCx-V (adopted) | Virtualization: many database VMs with shifting load on one host | Tests the consolidation claims (N VMs per LinuxONE vs x86) directly | high |
| TPC-Pricing, TPC-Energy (adopted) | Common methodologies for price and energy metrics | Method for the open cost model (M7) and for energy where it can be measured | high (method only) |
| TPC-C | Classic order-entry OLTP (spec only; HammerDB's TPROC-C is the common derived kit) | A widely recognised OLTP baseline on PostgreSQL and Db2 | medium |
| TPC-DS | Modern decision support: 99 queries plus data maintenance | A broader analytics check beside TPC-H | medium |
| TPCx-AI | End-to-end AI pipelines (data prep, training, serving) | Relevant to the Telum AI accelerator; stack portability to s390x unknown | medium |
| TPCx-HS | Big-data sort (TeraSort-style, Hadoop/Spark) | Relates to our sort tests, but the Hadoop stack is far from mainframe practice | low |
| TPCx-BB, TPCx-IoT, TPCx-HCI | Big-data analytics, IoT ingest, hyperconverged infrastructure | Little overlap with mainframe workloads | low / skip |

## System access: what we're pursuing
We need s390x and z/OS time on platforms whose terms allow benchmarking **and** publishing results. Routes in progress:
- the IBM Z Academic Initiative (a research allocation);
- the LinuxONE Community Cloud project (asking whether a written exception is possible, since its terms prohibit benchmarking);
- the Open Mainframe Project's mainframe access program.

**Advice welcome:** other routes, programmes or people to ask. Comment on
[#1](../../issues/1) (s390x) or [#2](../../issues/2) (z/OS).

## Open questions (comment welcome)
- **Base application:** CardDemo (proposed) or a smaller purpose-built schema?
- **Data layer for the Linux port:** GnuCOBOL indexed files (proposed; mirrors VSAM) or a SQL variant (PostgreSQL)?
- **The Java track's keyed store:** what keeps it portable and fair across x86, Linux on Z and z/OS (e.g. an embedded key-value store, or JDBC to PostgreSQL / Db2)?
- **Transaction mix weights** for the online set.
- **The AI model for the scoring step:** a public fraud model and dataset (candidates: the IBM LSTM fraud and TabFormer models).
- **Fair x86 accelerator coverage** on consumer hardware without QAT or AMX.
- **Order of the TPC modules** (T1-T4), and whether any medium-priority ones are worth adding.
