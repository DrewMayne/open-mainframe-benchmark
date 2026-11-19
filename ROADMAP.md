# Roadmap

| Milestone | Content | Status |
| --- | --- | --- |
| M0 | Plan, spec v0.1, pre-registered hypotheses, prior-art review, platform facts | **this commit** |
| M1 | Kit skeleton on x86 Linux: data generator, 2 online transactions, B2 sort, load driver, run manifest; 5-run repeatability shown | next |
| M2 | Full CardDemo port (online + batch) and accelerated extension on x86, bare metal and VM; spec v0.2 with transaction mix table | planned |
| M3 | s390x leg on a LinuxONE / Linux on Z platform whose terms permit benchmarking; H1-H5 phase 1 results | access being arranged |
| M4 | Technical report (preprint) with all raw data | planned |
| M5 | Phase 2: x86 with QAT/AMX; z/OS with CICS, VSAM, DFSORT and WLM; MIPS-to-x86-core calibration | access being arranged |

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
- **Transaction mix weights** for the online set.
- **The AI model for the scoring step:** a public fraud model and dataset (candidates: the IBM LSTM fraud and TabFormer models).
- **Fair x86 accelerator coverage** on consumer hardware without QAT or AMX.
