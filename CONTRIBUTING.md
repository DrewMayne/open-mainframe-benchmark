# Contributing

Contributions are welcome. Two kinds help most: **methodology critique** (if a design choice favours one platform,
open an issue and say why) and **access to systems** (below).

## Ways to contribute
- **Critique and spec changes:** open a [methodology issue](../../issues/new?template=methodology.md). Point at the spec section and say what would be fairer.
- **Platform access:** open a [platform access issue](../../issues/new?template=platform-access.md) (see below).
- **Platform facts:** a correction or addition to docs/platforms.md, always with a source URL.
- **Code:** kit code (once M1 lands), ports and documentation, as pull requests. Keep them focused, and explain the reason in the description. Issues labelled `good first issue` are a place to start.
- **Results:** open a [results issue](../../issues/new?template=results.md), with the run manifest, the raw output and the spec version.

## Offering access to a system
These help us most:
- LinuxONE or Linux on Z guests (2-4 IFLs, dedicated preferred);
- z/OS sandboxes with CICS, VSAM, DFSORT and an adjustable WLM service class;
- x86 servers with QAT/AMX, or EPYC Zen 4+.

What we commit to:
- **Terms first.** We run only where the platform's terms allow performance testing and publishing results, or with the owner's written permission. Please say which applies when you offer access.
- **Agreed load.** Scheduled windows, a fixed maximum duration per run, capped CPU, and an immediate stop on request. No overrides of site limits.
- **Disclosure.** The configuration (core counts, SMT, sharing and capping, other load) and any constraint you set are published with the results. Your support is acknowledged, unless you ask us not to name you.
- **Independence.** You may review drafts for factual accuracy, but results are not subject to approval.
- **Hygiene.** No credentials, internal hostnames or private data ever enter this repository.
- **Functional checks are not measurements.** On a system without benchmark permission, we only compile and run small pass/fail checks, and we never record or publish performance data from it (docs/PORTING-zOS.md).

## TPC materials
If you work on a TPC-derived part of the benchmark, follow SPEC.md section 6:
- never commit TPC kits or spec text;
- label every derived result "not comparable to published TPC results";
- never use TPC names or metrics for our own workloads;
- no marketing claims.

## Licence
Contributions are under the Apache License 2.0 (inbound = outbound).
