# Full Disclosure Report (FDR) template

Every published result comes with one FDR. The structure follows the discipline of audited industry benchmarks, so a
reader can review a result, and someone else can reproduce it, without asking us anything. **It is not a TPC Full
Disclosure Report, and results from this kit are not TPC results.**

Copy this file to `results/<date>-<platform>-<short-name>/FDR.md` and fill in every section. Write "n/a" with a
reason rather than leaving a section out.

## 1. Summary
- Spec version, kit commit (git SHA) and result ID
- Platform, one line: machine and model, cores/IFLs used, OS
- Headline metrics: throughput per core, p99, CPU per transaction, interference ratio (where measured)
- The hypotheses this result scores, and each verdict (VERIFIED / DIFFERS)

## 2. System under test
- **Hardware:**
  - machine type and model, CPU model, clock;
  - cores or IFLs used, dedicated or shared (with weights and capping, if known);
  - SMT on or off;
  - memory, storage (type, interface, size) and network, if used.
- **Virtualisation:** hypervisor(s) and layers (e.g. LPAR > z/VM > guest; Hyper-V > WSL2), and steal time observed.
- **Accelerators present, and whether each was enabled:** DFLTCC, CPACF/KDSA, AIU, AES-NI/SHA-NI, QAT, AMX, AVX-512.
- **Other load on the system during runs,** if known, and how it was established.

## 3. Software
- OS and kernel version, and the package versions of the compiler, libraries and runtime (GnuCOBOL, zlib, OpenSSL, zDNN or ONNX Runtime, JVM, PostgreSQL).
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
- CPU split (user/sys/iowait/steal) and memory and swap peaks.
- Failed runs and why they failed (OOM, timeout and so on). These are results, not omissions.

## 7. Raw data
- Links to the raw output, manifests and monitor samples, with their checksums.
- The exact command lines used.

## 8. Pricing appendix (optional)
- Published list prices only, with source and date, for the hardware, software and support terms used.
- Assumptions stated explicitly.
- No negotiated or confidential prices. A metric per dollar is labelled "list-price estimate".

## 9. Platform terms and support
- Confirmation that the platform's terms allow performance testing and publication, or that written permission was obtained (who and when, if it can be disclosed).
- Who provided access or support, and any constraints they set.
- Any factual review of a draft, and who did it. Nobody approves results.

## 10. Attestation
- Who ran it, and who independently re-ran or reviewed it (if anyone).
- The date.
