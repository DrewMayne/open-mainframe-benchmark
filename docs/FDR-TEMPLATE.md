# Full Disclosure Report (FDR) template

Every published result comes with one FDR. The structure follows the discipline of audited industry benchmarks, so a
reader can review a result, and someone else can reproduce it, without asking us anything. **It is not a TPC Full
Disclosure Report, and results from this kit are not TPC results.**

Copy this file to `results/<date>-<platform>-<short-name>/FDR.md` and fill in every section. Write "n/a" with a
reason rather than leaving a section out.

## 1. Summary
- **Registered verdicts, first:** the hypotheses this result scores, each with its verdict (VERIFIED / DIFFERS), the scoring configuration named in docs/HYPOTHESES-ADDENDUM.md, its confidence interval and its Holm-adjusted level; or "scores no registered hypothesis" (SPEC 4.12)
- **Preliminary or not:** "Preliminary: the preconditions of SPEC section 10 are not all met", with the list of those not met; or the evidence that each is met
- Spec version, kit commit (git SHA), result ID, and the run registry entries of the series (SPEC 4.13)
- Platform, one line: machine and model, cores/IFLs used, OS
- The comparison tier: equal core counts (single box), equal service (single box, exploratory), or the scale-out tier (single-writer, exploratory)
- The request profile; the data variant, its label and durability class; for a live database, its form: same product (platform comparison) or whole stack (whole-stack comparison)
- Results, by tier:
  - equal core counts: the core or IFL count on each side, and closed-loop capacity per core with its confidence interval;
  - equal service and the scale-out tier: the service target (throughput, transaction mix, request profile, and the p99 bound fixed before the runs) and, for each side, the configuration that sustained it: cores or engines by type, CPU time per transaction by engine type, memory, accelerators, power where measurable and cost where priced, with the sentence of SPEC 8.7
- At the reported point (the capacity point for equal core counts, the sustaining configuration otherwise): p99, CPU per transaction, and the interference ratio (where measured)
- Normalisations, each labelled as one: per core or engine, per watt and per dollar (per watt and per dollar only where both sides are on the same basis)

## 2. System under test
- **Hardware:**
  - machine type and model, CPU model, clock;
  - cores or IFLs used, dedicated or shared. For shared processors these fields are **required**: the partition's weight; the total weight of the active partitions sharing the pool and the number of physical processors of the type in it, and from them the partition's entitlement; the capping (none, initial, absolute with its value, defined capacity or group capacity limit); the number of logical processors of each type online; HiperDispatch on or off, and the polarisation of each logical processor at the start and end of each run; whether Intelligent Resource Director management is off; the System Recovery Boost setting and the IPL time; and the utilisation of the pool by the other partitions during each run. The partitions sharing the same chip or drawer caches, and the drawer and chip of each logical processor, are listed where known. A counted result needs dedicated engines, or shared engines under the capping and entitlement rule (SPEC 4.4 and 8.2);
  - SMT on or off;
  - memory, storage (type, interface, size) and network, if used;
  - processors and adapters not counted as cores: on Z, SAPs, IFPs, the on-chip DPU (z17), channels, network adapters and their offloads, and cryptographic adapters; on x86, network adapters and their offloads (and any DPU or SmartNIC), storage controllers, and QAT, IAA or DSA where present; on both, the storage subsystem, and any load-balancer appliance that terminates no TLS and runs no step of the request pipeline;
  - the hardware generation: machine type and model, or CPU model, with its general-availability date.
- **Virtualisation:** hypervisor(s) and layers (e.g. LPAR > z/VM > guest; Hyper-V > WSL2), and steal time observed.
- **Accelerators present, and whether each was enabled:** DFLTCC (the on-chip compression accelerator, zEDC), CMPSC (the compression call instruction), CPACF/KDSA, the Z Sort accelerator (SORTL), the IBM Integrated Accelerator for AI (NNPA), with the number of AI accelerators the partition's cores can reach (on z17, every accelerator in the drawers that hold the partition's cores: IBM, "New Telum II Processor and IBM Spyre Accelerator", https://www.ibm.com/blog/announcement/telum-ii), IBM Spyre Accelerator adapters, Crypto Express adapters, AES-NI/SHA-NI, QAT, IAA, DSA, AMX, AVX-512.
- **Plain and accelerated, defined per platform:** for each step and platform, the library build, the capability mask (for example `OPENSSL_s390xcap`, `OPENSSL_ia32cap`) and the device the accelerated mode uses (SPEC 1.4).
- **Evidence that each ran, or did not:** counters, not configuration (SPEC 1.4).
- **Output quality:** compressed sizes per side; the key protection class of the signing key; the numeric format of the scoring model and its accuracy on the fixed held-out set (SPEC 1.4).
- **Other load on the system during runs** (required), and how it was established: steal time on Linux guests, and on shared Z capacity the other partitions' processor consumption from the RMF partition data report or the hypervisor. A result whose other load cannot be established is labelled "other load unknown" and scores nothing.

## 3. Software
- OS and kernel version, and the package versions of the compiler, libraries and runtime (GnuCOBOL, zlib, OpenSSL, zDNN or ONNX Runtime, JVM), and the database product, version and fix level for a live-database result (Db2 for z/OS, Db2 for Linux, UNIX, and Windows, or PostgreSQL), with its durability settings.
- On z/OS: release, CICS, DFSORT, the WLM policy excerpt for the service classes used.
- Every non-default setting: compiler flags, environment variables (e.g. `DFLTCC`), cgroups weights, buffer sizes.

## 4. Workload
- The scale factor and data generator seed, with checksums of the generated data.
- The transaction mix table actually used, with the batch jobs and their order.
- Which accelerated-extension steps were on, and in which mode (plain / accelerated).

## 5. Run rules followed
- Repeats per point, randomisation, warm-up, window length and batch phase alignment (spec section 4).
- **Deviations from the run rules:** every one, with its reason. A result with an undisclosed deviation is invalid.

## 6. Results
- Per point: mean, 95% CI, median, min/max, and N.
- Latency: p50/p90/p99/p99.9.
- CPU busy time by column (user, nice, system, irq, softirq), with iowait and steal beside it, and the confinement residue (docs/CPU-ACCOUNTING.md); on IBM Z, the partition's effective dispatch time and LPAR management time; memory and swap peaks.
- Failed runs and why they failed (OOM, timeout and so on). These are results, not omissions.

## 7. Raw data
- Links to the raw output, manifests and monitor samples, with their checksums.
- The exact command lines used.

## 8. Pricing appendix (optional)
- Published list prices only, with source and date, for the hardware, software and support terms used.
- The licence metric of each software product (for example per core, per Processor Value Unit, or MSU capacity or consumption), and the quantity of it the configuration needs.
- Assumptions stated explicitly.
- No negotiated or confidential prices. A metric per dollar is labelled "list-price estimate".

## 9. Platform terms and support
- Confirmation that the platform's terms allow performance testing and publication, or that written permission was obtained (who and when, if it can be disclosed).
- Who provided access or support, and any constraints they set.
- Any factual review of a draft, and who did it. Nobody approves results.

## 10. Attestation
- Who ran it, and who independently re-ran or reviewed it (if anyone).
- The date.
