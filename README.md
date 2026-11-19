# Open Mainframe Benchmark

An open, reproducible benchmark specification and kit for comparing IBM Z / LinuxONE with x86 servers. It runs COBOL
online transactions and COBOL batch together, from the same source on every platform.

**Status: early design (v0.1). No kit code and no results yet.** This repository publishes the plan, the specification
draft and the pre-registered hypotheses *before* any measurement. Anyone can then check that the results answer the
questions as they were asked.

## Why
Published mainframe-against-server comparisons are almost all vendor-run. The best-known workload for sizing mainframe
replacements (HP's "zRef", z/OS Reference Application) is proprietary, its original paper is no longer online, and its
results can't be reproduced. IBM's own accelerator figures compare one generation of Z with another, and its x86
comparisons mostly rest on estate models or added network delay. No audited TPC, SPEC, MLPerf or Sort Benchmark result
exists for any IBM mainframe. See [docs/prior-art.md](docs/prior-art.md): 125 sources reviewed, each marked by
independence and by whether we read the primary text.

This project aims to fill that gap in the open: same code, equal cores, repeated runs with confidence intervals, every
script and raw file published, and predictions written down before the data exists.

## What it measures
- **Per-core throughput:** the same COBOL workload on a LinuxONE IFL and an x86 core.
- **Accelerators on both sides:** on-chip compression, crypto and AI inference on Z, against the x86 equivalents (AES-NI/SHA, and QAT/AMX where available), each also switched off.
- **Batch and online together:** online capacity and tail latency while batch runs on the same machine, under default scheduling and under tuned priorities (Linux cgroups; z/OS WLM in a later phase).
- **Behaviour past saturation:** throughput and p99/p99.9 latency from 50% to 120% of capacity, i.e. how each machine degrades, not just its peak.
- **Batch elapsed time,** alone and alongside online work.

## Documents
- [SPEC.md](SPEC.md): workload, platforms, metrics and run rules (draft v0.1)
- [HYPOTHESES.md](HYPOTHESES.md): the pre-registered predictions H1 to H5
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

## License
Apache License 2.0, see [LICENSE](LICENSE). Any third-party code adopted later (for example AWS CardDemo, also Apache
2.0) keeps its own licence and notices; see [NOTICE](NOTICE).

Maintainer: Andrew Mayne. Contributions and critique are welcome, especially methodological critique.
