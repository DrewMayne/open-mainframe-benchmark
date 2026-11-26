# Roadmap

| Milestone | Content | Status |
| --- | --- | --- |
| M0 | Plan, spec v0.1/v0.1.1 (two tracks), pre-registered hypotheses, prior-art review, platform facts, FDR template | done |
| M1 | **Java track** skeleton on x86 Linux: data generator, 2 online transactions, B2 sort, load driver, run manifest, golden-output test; repeatability shown over the run plan's repeats (at least 6 over at least 3 boots, SPEC 4.1) | planned (next) |
| M1b | Java track complete: all online transactions, batch suite, accelerated extension; a later spec version with the transaction mix table | planned |
| M2 | **COBOL track:** CardDemo port to GnuCOBOL (online + batch) and accelerated extension on x86, bare metal and VM; track equivalence passes | planned |
| M3 | s390x leg (both tracks) on a LinuxONE / Linux on Z platform whose terms permit benchmarking; H1-H5 and H1j phase 1 results, in the scoring configurations of docs/HYPOTHESES-ADDENDUM.md (cross-platform ratios against a current x86 server, SPEC 2) | planned; access sought (#1) |
| M4 | Technical report (preprint) with all raw data | planned |
| M5 | Phase 2: x86 with QAT/AMX; z/OS with CICS, VSAM, DFSORT and WLM, and Db2 for z/OS for the live-database variant (COBOL track on Enterprise COBOL; Java track with and without zIIPs). No MIPS-to-x86-core calibration (SPEC 3) | planned; access sought (#2, #3) |
| M6 | Peer-reviewed reproducibility: submit the report and kit for artifact evaluation (e.g. ACM ICPE), so that independent reviewers re-run it | planned |
| M7 | Open cost model: a separate report in which every cost assumption is a visible, editable input, fed by the measured ratios. No hidden utilisation or licence assumptions | planned (later) |
| Modules | Optional modules derived from industry-standard benchmark material: in preparation, published only once the benchmark owners' permission is in place | withdrawn |
| T1 | Pricing appendix and energy section of the FDR, applied on x86 | notes (docs/FDR-TEMPLATE.md section 8; docs/methods/energy.md) |
| Stretch | An audited run of an actual TPC benchmark on Z and x86, as a separately sponsored effort with a TPC-certified auditor | planned (needs a sponsor) |

Status words are used as defined in the README ("Status at a glance").

## System access: what we're pursuing
We need s390x and z/OS time on platforms whose terms allow benchmarking **and** publishing results. Routes in progress:
- the IBM Z Academic Initiative (a research allocation);
- the LinuxONE Community Cloud project (asking whether a written exception is possible, since its terms prohibit benchmarking);
- the Open Mainframe Project's mainframe access program.

**Advice welcome:** other routes, programmes or people to ask. Comment on
[#1](../../issues/1) (s390x) or [#2](../../issues/2) (z/OS).

## Open questions (comment welcome)
- **Base application:** CardDemo (proposed) or a smaller purpose-built schema?
- **Data layer for the Linux port:** both, as separate labelled variants: GnuCOBOL indexed files (mirrors VSAM), and a live database (SPEC 1.8.4) with Db2 and PostgreSQL in two forms. Open: which SQL interface the COBOL track uses on Linux.
- **The Java track's keyed store:** what keeps it portable and fair across x86, Linux on Z and z/OS (e.g. an embedded key-value store, or JDBC to PostgreSQL / Db2)?
- **Transaction mix weights** for the online set.
- **The AI model for the scoring step:** a public fraud model and dataset (candidates: the IBM LSTM fraud and TabFormer models).
- **Fair x86 accelerator coverage** on consumer hardware without QAT or AMX.
