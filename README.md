# Open Mainframe Benchmark

> **Not affiliated with the Open Mainframe Project or The Linux Foundation.** Open Mainframe Benchmark is an
> independent project. It is not a project of the Open Mainframe Project or of The Linux Foundation, and it is not
> endorsed or sponsored by either of them, by IBM, or by any x86 vendor. "Open Mainframe Project" is a trademark of
> The Linux Foundation; see [NOTICE](NOTICE) for trademarks.

An open, reproducible benchmark specification and kit for comparing IBM Z / LinuxONE with x86 servers. It runs online
transactions and batch together, from the same source on every platform, in two tracks: **Java** (the same JVM
everywhere, for the hardware question) and **COBOL** (based on AWS CardDemo, for the migration question).

**Status: early design (spec draft v0.4.0). No kit code and no results yet.** This repository publishes the plan,
the specification draft and the pre-registered hypotheses *before* any kit run. Anyone can then check that the
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
| Specification, hypotheses, FDR template | [SPEC.md](SPEC.md), [HYPOTHESES.md](HYPOTHESES.md), [docs/HYPOTHESES-ADDENDUM.md](docs/HYPOTHESES-ADDENDUM.md), [docs/FDR-TEMPLATE.md](docs/FDR-TEMPLATE.md) | notes: spec draft v0.4.0; hypotheses registered against v0.1 and v0.1.1, unchanged; addendum entry 1 (tag omb-v0.4.0-prereg; registration status and DOI in docs/REGISTRY.md) on the comparison basis and the scoring configurations | none |
| Registry of registrations, machines, runs and tuning | [docs/REGISTRY.md](docs/REGISTRY.md) | notes; no machine named and no kit run made | none |
| Architecture model, playbook and CPU accounting | [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md), [docs/PLAYBOOK.md](docs/PLAYBOOK.md), [docs/CPU-ACCOUNTING.md](docs/CPU-ACCOUNTING.md) | notes (draft) | none |
| Java track (M1, M1b) | not started | planned (M1 is next) | none |
| COBOL track (M2) | not started | planned | none |
| s390x and z/OS legs (M3, M5) | [docs/platforms.md](docs/platforms.md), [docs/PORTING-zOS.md](docs/PORTING-zOS.md) | planned; access sought ([#1](../../issues/1), [#2](../../issues/2)) | none |
| Optional modules derived from benchmark material | not in this repository | withdrawn | private |
| Pricing and energy methods (T1) | [docs/FDR-TEMPLATE.md](docs/FDR-TEMPLATE.md) section 8, [docs/methods/energy.md](docs/methods/energy.md) | notes (draft) | none |
| Prior-art review | [docs/prior-art.md](docs/prior-art.md) | notes | none |

## Preconditions before any headline result
No result of this benchmark is a headline result yet, and none can be until all six of these hold (SPEC section
10):
1. **A published kit.** Today no kit code is published. A result needs a tagged kit release with its data
   generator, load driver, Linux port, Java reimplementation and golden script.
2. **A sanctioned platform.** Today the project has no IBM Z or LinuxONE system whose terms allow benchmarking and
   publishing results. Every machine in a result needs its owner's written permission.
3. **Licence terms checked.** Today the terms of the programs a run would use have not been checked. Every licensed
   program in a run, on both sides, is checked for conditions on benchmarking and publishing results first.
4. **An independent reproduction.** Today no one has re-run anything. A team independent of the maintainer must
   reproduce a result from the published kit before it is called a headline.
5. **External pre-registration.** Today the hypotheses are timestamped by git history only. The addendum is
   registered at the signed tag `omb-v0.4.0-prereg` and deposited in an archive that issues a DOI, recorded in
   [docs/REGISTRY.md](docs/REGISTRY.md), before the first counted kit run; every machine is named in the machine
   registry before any kit run on it; and every kit run is entered in the run registry before it starts.
6. **A pilot and a power analysis.** Today neither exists. Before the registration tag, a pilot over at least three
   boots or IPLs on each side, and a power analysis from it, show which predictions the planned repeats can resolve.

Until then, every result is labelled "preliminary", and even a preliminary result is published only with the
platform owner's permission and after the licence check (preconditions 2 and 3). The equal-service comparison is
exploratory, with no registered prediction, and none of its results is a headline result until a prediction about
it is registered.

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
  This finding does not cover SAP's Standard Application Benchmarks, whose results SAP certifies rather than
  audits; docs/prior-art.md does not yet review them.

This project aims to fill that gap in the open, with numbers that IBM, x86 vendors and customers can all re-run:
same code, equal cores for the registered per-core predictions, an equal-service comparison (what each side needs
to sustain the same throughput within the same latency bound), repeated runs with confidence intervals, every
script and raw file published, and predictions committed before the kit runs that test them.

## What it measures
- **Per-core throughput (COBOL track):** the same CardDemo-based COBOL, compiled by GnuCOBOL through a C compiler on
  both sides, on Linux on IBM Z IFLs and on x86 cores, at equal core counts (the registered hypothesis H1). It
  compares the hardware together with each platform's C code generation and GMP build. It does not measure IBM
  Enterprise COBOL, z/OS or IBM Z's decimal hardware, which production COBOL on IBM Z uses (SPEC 1.0).
- **Equal service (exploratory):** for a stated throughput within a p99 bound fixed before the runs, what each side
  needs: cores or engines by type, memory and accelerators, and power and cost where they can be stated.
- **A live database:** the same work over Db2 or PostgreSQL, as a same-product platform comparison and as a
  labelled whole-stack comparison.
- **Accelerators on both sides:** on-chip compression, crypto and AI inference on Z, against the x86 equivalents (AES-NI/SHA, and QAT/AMX where available), each also switched off.
- **Batch and online together:** online capacity and tail latency while batch runs on the same machine, under default scheduling and under tuned priorities (Linux cgroups; z/OS WLM in a later phase).
- **Behaviour past saturation:** throughput and p99/p99.9 latency from 50% to 120% of capacity, i.e. how each machine degrades, not just its peak.
- **Batch elapsed time,** alone and alongside online work.

## Documents
- [SPEC.md](SPEC.md): workload, platforms, metrics and run rules (draft v0.4.0)
- [HYPOTHESES.md](HYPOTHESES.md): the pre-registered predictions H1 to H6, plus H1j
- [docs/HYPOTHESES-ADDENDUM.md](docs/HYPOTHESES-ADDENDUM.md): addendum entries, each registered at a signed tag: the change of comparison basis, the measurands and scoring configurations of the registered hypotheses, the choices they make, and the decision rules; no hypothesis is changed
- [docs/REGISTRY.md](docs/REGISTRY.md): the registrations with their DOIs, the machine registry, the run registry and the tuning logs
- [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md): the deployments compared, what is counted as a core, and proposed FDR additions
- [docs/PLAYBOOK.md](docs/PLAYBOOK.md): running the benchmark end to end, step by step
- [docs/CPU-ACCOUNTING.md](docs/CPU-ACCOUNTING.md): CPU time per transaction on one basis on both sides
- [CHANGELOG.md](CHANGELOG.md): changes by spec version
- [ROADMAP.md](ROADMAP.md): milestones and open questions
- [docs/prior-art.md](docs/prior-art.md): review of prior mainframe-against-server measurements
- [docs/platforms.md](docs/platforms.md): s390x and x86 platform and toolchain facts, with sources
- [GOVERNANCE.md](GOVERNANCE.md), [CONTRIBUTING.md](CONTRIBUTING.md), [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md), [SECURITY.md](SECURITY.md), [CITATION.cff](CITATION.cff)

## Principles
1. **Independence.** No vendor funds or approves results. Any help (system access, hardware) is acknowledged in full,
   and drafts may be shared for factual review only.
2. **Reproducibility.** One source tree, pinned versions, a manifest with every run, and raw data published.
3. **Pre-registration.** Hypotheses are committed before the runs that test them. Each result is scored VERIFIED or
   DIFFERS against its prediction, and misses are published too.
4. **Fair comparison.**
   - The registered per-core predictions are tested at equal core counts, and their verdicts are reported first.
   - Beside them, an exploratory equal-service comparison asks what each side needs to sustain a stated throughput
     within a p99 bound fixed before the runs. Cores or engines by type, memory, accelerators, power and cost are
     disclosed, and the processors that do work outside the counted cores are listed on both sides.
   - Accelerators are used on both sides or neither, tuned and untuned configurations are both reported, and every
     limitation is stated. Where a licence term prevents publishing a configuration, the FDR names the term and the
     configuration withheld.
5. **Permitted platforms only.** Runs happen only where the platform's terms allow performance testing and publishing
   results, with written permission where needed. See [docs/platforms.md](docs/platforms.md).

## Help wanted
The project is open for contribution: code, critique, results and, most of all, **access to systems**.
- **LinuxONE / Linux on Z:** 2-4 IFLs (dedicated preferred), for a few weeks of scheduled runs.
- **z/OS:** a sandbox with CICS, VSAM, DFSORT, Db2 for z/OS and a WLM service class we may adjust, in agreed windows.
- **x86 clusters and load-driver hosts:** several identical servers for the scale-out tier, and separate machines to
  generate load.
- **What a counted result needs from an IBM Z system:** Linux directly in an LPAR, or a z/OS partition, on dedicated
  or entitled engines; permission for IPLs or re-activations between repeats and for an abrupt stop in the
  durability test; and authorisation for the CPU-measurement facility counters and the partition data the CPU
  basis reads (docs/PLAYBOOK.md, section 2.1). z/VM and KVM guests are welcome for development and labelled
  variants, but score nothing.
- **x86 servers with accelerators:** Intel QAT and AMX (4th Gen Xeon or later), or AMD EPYC Zen 4+.

**We only run where the platform's terms allow performance testing and publishing results**, or with the owner's
written permission. Runs are capped, scheduled and stoppable on request, and every configuration and constraint is
published with the results. Offers of access: open a
[platform access issue](../../issues/new?template=platform-access.md). See [CONTRIBUTING.md](CONTRIBUTING.md) for the rest.

## License
Apache License 2.0, see [LICENSE](LICENSE). Any third-party code adopted later (for example AWS CardDemo, also Apache
2.0) keeps its own licence and notices; see [NOTICE](NOTICE).

## About this project
Maintained by Andrew Mayne, Durham College. An independent project, not a Durham College publication.

AI tools assisted with research compilation and drafting. Each prior-art entry marks how its sources were seen:
fetched and read, not fetched, or seen only in a search-result snippet (marked "not verified"). On 2026-11-25 a
sample of entries was re-checked against their sources, and the eight that differed were corrected. The maintainer
is responsible for the content; corrections are welcome as issues.

Contributions and critique are welcome, especially methodological critique. Everyone taking part follows the
[code of conduct](CODE_OF_CONDUCT.md). Questions go to GitHub Discussions or the issues; security issues go through
GitHub's private vulnerability reporting ([SECURITY.md](SECURITY.md)); the other routes are in
[GOVERNANCE.md](GOVERNANCE.md), "Contact". To cite the project, see [CITATION.cff](CITATION.cff).
