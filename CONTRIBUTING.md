# Contributing

Contributions are welcome. Two kinds help most: **methodology critique** (if a design choice favours one platform,
open an issue and say why) and **access to systems** (below).

## Ways to contribute
- **Critique and spec changes:** open a [methodology issue](../../issues/new?template=methodology.md). Point at the spec section and say what would be fairer.
- **Platform access:** open a [platform access issue](../../issues/new?template=platform-access.md) (see below).
- **Platform facts:** a correction or addition to docs/platforms.md, always with a source URL.
- **Code:** kit code (once M1 lands), ports and documentation, as pull requests. Keep them focused, and explain the reason in the description. Until kit code exists, a good place to start is a correction to docs/platforms.md or docs/prior-art.md with a source, or a methodology issue. Issues will be labelled `good first issue` once there are some.
- **Results:** open a [results issue](../../issues/new?template=results.md), with the Full Disclosure Report, the run manifest, the raw output, the machine and run registry entries of the series and the spec version (GOVERNANCE.md, "Results").

## Offering access to a system
These help us most:
- LinuxONE or Linux on Z guests (2-4 IFLs, dedicated preferred);
- z/OS sandboxes with CICS, VSAM, DFSORT, Db2 for z/OS and an adjustable WLM service class;
- x86 clusters of identical servers for the scale-out tier, and separate machines to act as load-driver hosts;
- for counted results on IBM Z: Linux directly in an LPAR or a z/OS partition, dedicated or entitled engines,
  permission for IPLs and an abrupt stop, and counter and partition-data authorisation (docs/PLAYBOOK.md, section
  2.1); guests suit development and labelled variants;
- x86 servers with QAT/AMX, or EPYC Zen 4+.

What we commit to:
- **Terms first.** We run only where the platform's terms allow performance testing and publishing results, or with the owner's written permission. Please say which applies when you offer access.
- **Agreed load.** Scheduled windows, a fixed maximum duration per run, capped CPU, and an immediate stop on request. No overrides of site limits.
- **Disclosure.** The configuration (core counts, SMT, sharing and capping, other load) and any constraint you set are published with the results. Your support is acknowledged, unless you ask us not to name you.
- **Independence.** You may review drafts for factual accuracy, but results are not subject to approval.
- **Hygiene.** No credentials, internal hostnames or private data ever enter this repository.
- **Functional checks are not measurements.** On a system without benchmark permission, we only compile and run small pass/fail checks, and we never record or publish performance data from it (docs/PORTING-zOS.md).

## Third-party benchmark materials
Follow SPEC.md section 6:
- never commit third-party benchmark kits or specification text;
- modules derived from benchmark material, and their results, are published only once the benchmark owner's
  written permission is in place;
- never use another benchmark's name or metrics for our own workloads;
- no marketing claims.

## Licence
Contributions are under the Apache License 2.0 (inbound = outbound).

## Conduct and contact
- Everyone taking part follows the [code of conduct](CODE_OF_CONDUCT.md).
- Questions and general discussion: GitHub Discussions on this repository, or an issue.
- Conduct reports: GitHub's content reporting, from the "Report content" menu on the issue, pull request, discussion
  or comment: "Report to repository admins" (to the maintainer) or "Report abuse" (to GitHub Support). Questions
  about the code of conduct, and concerns not tied to a piece of GitHub content, go to GitHub Discussions
  ([CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md); [GOVERNANCE.md](GOVERNANCE.md), "Contact").
- A vendor's reply to a result: an issue with the label `vendor-reply` (GOVERNANCE.md).
- Platform access offers: the platform-access issue template, with no private details.
- Security issues only: GitHub's private vulnerability reporting ([SECURITY.md](SECURITY.md)).
