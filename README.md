# Open Mainframe Benchmark

*Not a project of the Open Mainframe Project or The Linux Foundation, and not affiliated with either. See
[NOTICE](NOTICE) for trademarks.*

An open, reproducible benchmark specification and kit for comparing IBM Z / LinuxONE with x86 servers. It runs online
transactions and batch together, from the same source on every platform, in two tracks: **Java** (the same JVM
everywhere, for the hardware question) and **COBOL** (based on AWS CardDemo, for the migration question).

**Status: early design (spec draft v0.3.2). No kit code and no results yet.** This repository publishes the plan,
the specification draft and the pre-registered hypotheses *before* any measurement. Anyone can then check that the
results answer the questions as they were asked. The hypotheses were registered against spec v0.1 (H1-H5) and
v0.1.1 (H1j, H6).

## Status at a glance
> **Optional modules withdrawn for now.** Optional modules derived from industry-standard benchmark material are
> in preparation and kept private until the benchmark owners' permission is in place. No kit code and no results
> are published yet.

Status words, used the same way on every page: **planned** (on the roadmap; nothing written yet), **notes** (a plan,
design or method in this repository; no code), **withdrawn** (in preparation, but not described here until the
benchmark owners' permission is in place), and **done**.

| Part | Where | Status | Code |
| --- | --- | --- | --- |
| Specification, hypotheses, FDR template | [SPEC.md](SPEC.md), [HYPOTHESES.md](HYPOTHESES.md), [docs/FDR-TEMPLATE.md](docs/FDR-TEMPLATE.md) | notes: spec draft v0.3.2; hypotheses registered against v0.1 and v0.1.1 | none |
| Java track (M1, M1b) | not started | planned (M1 is next) | none |
| COBOL track (M2) | not started | planned | none |
| s390x and z/OS legs (M3, M5) | [docs/platforms.md](docs/platforms.md), [docs/PORTING-zOS.md](docs/PORTING-zOS.md) | planned; access sought ([#1](../../issues/1), [#2](../../issues/2)) | none |
| Optional modules derived from benchmark material | not in this repository | withdrawn | private |
| Pricing and energy methods (T1) | [docs/FDR-TEMPLATE.md](docs/FDR-TEMPLATE.md) section 8, [docs/methods/energy.md](docs/methods/energy.md) | notes (draft) | none |
| Prior-art review | [docs/prior-art.md](docs/prior-art.md) | notes | none |

## Why
Public measurements of IBM Z and x86 servers running the same work are scarce. Most published comparisons come from
a vendor of one side, IBM or an x86 or rehosting vendor, and the last independent head-to-head with confidence
intervals that we found is from 2014. See [docs/prior-art.md](docs/prior-art.md): 125 sources reviewed, each marked by
independence and by whether we read the primary text. Among its findings:
- HP's "zRef" (z/OS Reference Application), a workload used to size mainframe replacements, is proprietary. We could
  not retrieve its original paper, and its results can't be reproduced independently.
- IBM's published accelerator figures mostly compare Z with Z (an older generation, a general-purpose core, or
  software on the same machine). Its x86 comparisons include estate models, remote x86 servers with added network
  delay, and a few core-matched tests.
- We found no audited TPC, SPEC or MLPerf result for any IBM mainframe, and no Sort Benchmark entry. These are
  searches, not exhaustive queries of each results database; the Sort Benchmark records page was checked in full.

This project aims to fill that gap in the open, with numbers that IBM, x86 vendors and customers can all re-run:
same code, equal cores, repeated runs with confidence intervals, every script and raw file published, and
predictions written down before the data exists.

## What it measures
- **Per-core throughput:** the same COBOL workload on a LinuxONE IFL and an x86 core.
- **Accelerators on both sides:** on-chip compression, crypto and AI inference on Z, against the x86 equivalents (AES-NI/SHA, and QAT/AMX where available), each also switched off.
- **Batch and online together:** online capacity and tail latency while batch runs on the same machine, under default scheduling and under tuned priorities (Linux cgroups; z/OS WLM in a later phase).
- **Behaviour past saturation:** throughput and p99/p99.9 latency from 50% to 120% of capacity, i.e. how each machine degrades, not just its peak.
- **Batch elapsed time,** alone and alongside online work.

## Documents
- [SPEC.md](SPEC.md): workload, platforms, metrics and run rules (draft v0.3.2)
- [HYPOTHESES.md](HYPOTHESES.md): the pre-registered predictions H1 to H6, plus H1j
- [ROADMAP.md](ROADMAP.md): milestones and open questions
- [docs/prior-art.md](docs/prior-art.md): review of prior mainframe-against-server measurements
- [docs/platforms.md](docs/platforms.md): s390x and x86 platform and toolchain facts, with sources
- [GOVERNANCE.md](GOVERNANCE.md), [CONTRIBUTING.md](CONTRIBUTING.md)

## Principles
1. **Independence.** No vendor funds or approves results. Any help (system access, hardware) is acknowledged in full,
   and drafts may be shared for factual review only.
2. **Reproducibility.** One source tree, pinned versions, a manifest with every run, and raw data published.
3. **Pre-registration.** Hypotheses are committed before the runs that test them. Each result is scored VERIFIED or
   DIFFERS against its prediction, and misses are published too.
4. **Fair comparison.** Equal core counts, accelerators on both sides or neither, tuned and untuned configurations both
   reported, and every limitation stated.
5. **Permitted platforms only.** Runs happen only where the platform's terms allow performance testing and publishing
   results, with written permission where needed. See [docs/platforms.md](docs/platforms.md).

## Help wanted
The project is open for contribution: code, critique, results and, most of all, **access to systems**.
- **LinuxONE / Linux on Z:** 2-4 IFLs (dedicated preferred), for a few weeks of scheduled runs.
- **z/OS:** a sandbox with CICS, VSAM, DFSORT and a WLM service class we may adjust, in agreed windows.
- **x86 servers with accelerators:** Intel QAT and AMX (4th Gen Xeon or later), or AMD EPYC Zen 4+.

**We only run where the platform's terms allow performance testing and publishing results**, or with the owner's
written permission. Runs are capped, scheduled and stoppable on request, and every configuration and constraint is
published with the results. Offers of access: open a
[platform access issue](../../issues/new?template=platform-access.md). See [CONTRIBUTING.md](CONTRIBUTING.md) for the rest.

## License
Apache License 2.0, see [LICENSE](LICENSE). Any third-party code adopted later (for example AWS CardDemo, also Apache
2.0) keeps its own licence and notices; see [NOTICE](NOTICE).

Maintainer: Andrew Mayne. Contributions and critique are welcome, especially methodological critique.
