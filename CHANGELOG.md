# Changelog

All notable changes to the specification and the project's documents are recorded here, by spec version, so that a
result can be matched to the rules it ran under. The format is based on Keep a Changelog
(https://keepachangelog.com/en/1.1.0/), and versions follow semantic versioning. For the specification itself, the
change note at the top of [SPEC.md](SPEC.md) is authoritative; this file repeats it and adds changes to the
project's other documents. The registered text of [HYPOTHESES.md](HYPOTHESES.md) is never edited; entries of
[docs/HYPOTHESES-ADDENDUM.md](docs/HYPOTHESES-ADDENDUM.md) are listed here when they are added, with the tag that
registers them.

## [Unreleased]
Spec v0.4.0 (draft, unreleased).

### Added
- Six preconditions before any headline result: a published kit, a sanctioned platform, checked licence terms, an
  independent reproduction with an equivalence criterion, external pre-registration, and a pilot with a power
  analysis before the registration tag (SPEC 10; README).
- External pre-registration at signed tags, deposited in an archive that issues a DOI, with the archive's creation
  time as the timestamp; a machine registry, in which every machine is named before any kit run on it; a public run
  registry; a tuning log for every series (SPEC 4.7, 4.13, 4.14; docs/REGISTRY.md).
- An equal-service comparison, exploratory, with no registered prediction and no headline result; reported after
  the registered verdicts, with a rule on how it may be summarised (SPEC 2, 8, 4.12).
- A load model: open-loop arrivals, a persistent connection pool sized by Little's law with a cap, no TLS handshake
  inside the measured window, one timeout per series, throughput counted in committed requests, refusals counted as
  timeouts, decision and past-saturation points, a calibration bound fixed in advance, a per-repeat knee with a
  lower-median rule, equivalence tests for the driver checks, and a table of default parameters (SPEC 7).
- A scale-out tier, single-writer, with a declared availability design at the same layer on both sides, and one
  scaling-limit test for added nodes, engines and cores (SPEC 9).
- A request profile with per-request bookkeeping steps and TLS, and data variants with a live database in two
  labelled forms, with declared durability classes for the file-based variant; an invariant check against an
  application-level audit stream (SPEC 1.8).
- Run rules for the load generator, refusals, equal guarantees, the order of reporting, the run registry, the
  tuning log, storage class by durability and commit latency, acceptance tests, the statistics plan, accounting
  validation against a known load, and tuning parity (SPEC 4.9-4.18).
- Scale factors with a cache-residency rule, a defined request stream and loaded data, workload characterisation
  from hardware counters, transport equivalence, a declared CICS topology, a build policy, and accelerator evidence
  with size-matched compression output (SPEC 1.0, 1.1, 1.4, 1.5, 3).
- A fair x86 baseline: a current general-purpose server part named in advance, held at its sustained all-core
  frequency, the same neighbour load on both sides, vendor-optimised libraries in accelerated mode, a HotSpot
  variant of the Java track, NUMA and confinement rules (SPEC 1.0, 2, 8.2).
- Z partition rules: capping with entitlement, logical processors equal to the cap, no System Recovery Boost period,
  Intelligent Resource Director management off, and the partition running only the system under test (SPEC 4.4,
  8.2).
- CPU per transaction counted on one basis on both sides: the PR/SM view (effective dispatch time) for every Z
  partition, z/OS and Linux alike, with a zero-load check; the cpuset plus the confinement residue on x86; coupling
  facility busy from SMF type 74; hypervisor time counted for guests; SMT in thread-seconds; and the kernel
  accounting difference between s390 and x86 (SPEC 3; docs/CPU-ACCOUNTING.md).
- Required FDR fields for shared processors, other load, the Z and x86 platform state, and accelerator evidence
  (docs/FDR-TEMPLATE.md).
- New pages: docs/ARCHITECTURE.md, docs/PLAYBOOK.md, docs/CPU-ACCOUNTING.md, docs/REGISTRY.md; project files
  CODE_OF_CONDUCT.md, CITATION.cff and this changelog.
- Hypotheses: docs/HYPOTHESES-ADDENDUM.md, entry 1, to be registered at tag omb-v0.4.0-prereg (status in
  docs/REGISTRY.md): the change of comparison basis, the measurands, the scoring configurations, the choices they
  make, and the statistics and decision rules. No registered hypothesis is changed.

### Changed
- Equal core counts stay for the registered per-core hypotheses; per-core figures are a labelled normalisation
  (SPEC 2, 3).
- Confinement replaces pinning (SPEC 4.4); repeats are fixed, at least 6, spread over at least three boots or IPLs
  (SPEC 4.1); pre-registration needs an external timestamp (SPEC 4.7).
- README: the comparison tiers in "What it measures" and "Fair comparison", with GnuCOBOL named for the COBOL track;
  the not-affiliated notice; the preconditions section; the scope of the "no audited results" finding; the
  documents list, the contact routes and the citation file; what a counted result needs from an IBM Z system.
- CONTRIBUTING.md: hardware wanted for the scale-out tier and for load drivers, what a counted result needs on IBM
  Z, a first-contribution route, a conduct and contact section, and the FDR and registry entries with results.
  SECURITY.md: the private route is for security matters only.
- GOVERNANCE.md: competing interests, who counts as a vendor, platform permission obtained before the runs and never
  conditional on the outcome, tuning parity, public routes for vendor replies and conduct reports, and the licence
  check for every licensed program on either side.
- Issue templates: private details of access offers go through the maintainer's reply; results include the FDR
  and the run registry entries.
- docs/FDR-TEMPLATE.md: registered verdicts first, the preliminary status, results by tier, the CPU busy-time
  columns, the hardware not counted as cores, and the licence metric of each product.
- docs/PORTING-zOS.md: Db2 for Linux, UNIX, and Windows as a data adapter; which Java work is zIIP-eligible, shown by
  measurement; threadsafe definitions only after a code review; the LSR pool's buffers; no MIPS calibration.
- ROADMAP.md: M1 repeatability over the run plan's repeats; M3 in the addendum's scoring configurations; M5 with
  Db2 for z/OS and without the MIPS calibration; the data-layer question answered by two variants.
- docs/prior-art.md: passages that describe this project's design (not the review's findings) now name the
  single-box equal-core runs, the equal-service comparison and the z17 accelerator reach.
- NOTICE: trademark lines for PostgreSQL, UNIX, Windows, Eclipse Temurin, SAP, OpenJDK, Berkeley DB and GitHub.
- The Db2 variant comes into scope; IMS and MQ stay later (SPEC 5).

### Removed
- The planned MIPS calibration; no MIPS equivalence is computed (SPEC 3).

## [0.3.2] - 2026-11-25
### Fixed
- CardDemo on z/OS needs configuration changes to run batch and online together (SPEC 1.1, 4.3); CPC power is shown
  on the HMC for older Z machines (1.7); IBM's capacity ratings are per processor model (3); "Db2" (5); version
  labels made consistent.

### Removed
- Withdrawn until the benchmark owners' permission is in place: the optional modules derived from benchmark
  material, and the rules specific to them (1.6, 1.7, 6).

## [0.3.1] - 2026-11-19
### Changed
- Labelling of derived work aligned with the benchmark owner's published policies; written permission requested
  before publishing derived results (6).

## [0.3.0] - 2026-11-19
### Added
- Further optional modules derived from benchmark material, plus pricing and energy methods (1.7).

## [0.2.0] - 2026-11-19
### Added
- An optional analytics module derived from industry-standard benchmark material (1.6), and rules for using
  third-party benchmark materials (6).

## [0.1.1] - 2026-11-19
### Added
- Two implementation tracks (Java first, then COBOL) under one spec (1.0, 1.5).
- Hypotheses H1j and H6 added to HYPOTHESES.md.

## [0.1] - 2026-11-19
### Added
- First draft of the specification; hypotheses H1-H5 registered in HYPOTHESES.md.
