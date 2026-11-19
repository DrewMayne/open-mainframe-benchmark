# Feasibility: will the TPC-DI data generator run on current Linux and on s390x?

> **This workload is derived from the TPC-DI Benchmark and is not comparable to published TPC-DI Benchmark results, as this implementation does not comply with all requirements of the TPC-DI Benchmark.**

Checked on 2026-11-19 by reading the kit, without running the generator (a timed benchmark was using the PC). The
kit is the TPC-DI Tools download, extracted to `E:\tpc\TPC-DI\Tools\` (never into this repository).

**Verdict: workable, with Java 8 as the pinned runtime.** The generator is pure Java bytecode with no native code,
so nothing ties it to x86, and Java 8 runtimes exist for x86_64 and s390x. The risks below are real but each has a
cheap check or workaround. The fallback at the end is written down in case the first generation run fails on them.

## What the kit contains
| Item | Facts found | Risk |
| --- | --- | --- |
| `DIGen.jar` | DIGen 1.1.0 wrapper (version string in its README and `-v`). Built with Eclipse's jar-in-jar loader (class files version 45), bundles commons-cli 1.2. It starts a **second JVM** running PDGF and pipes stdin to it. Source is in `DIGen_Build.zip`. | low |
| `PDGF/pdgf.jar` | PDGF build R1343 (`Version.txt`; manifest `2.5_#1343_b4177`, built 2014-11-25 with JDK 1.7.0_71; class files version 50). **Binary only**: there is no PDGF source in the kit, so PDGF cannot be rebuilt or patched (and clause 6.2.2-6.2.4 forbid modifying it anyway). | medium |
| `PDGF/plugins/tpc-di.jar` | The TPC-DI generator plugin (class files version 51, Java 7); its source is in `tpc-di_src.jar`. | low |
| `PDGF/extlib/` | javassist 3.18.0.GA (2013), log4j 1.2.15, xercesImpl, commons-net 3.3. The manifest's class path names many more jars (Hadoop, Lucene, guava) that are absent; they are only needed for PDGF features TPC-DI does not use. | see below |
| Specification | **Not in the kit.** Downloaded separately from tpc.org (public PDF, `TPC-DI_v1.1.0.pdf`) to `E:\tpc\TPC-DI\spec\`. | none |
| Appendices A-C (audit, batch validation and data visibility SQL) | **Not in the kit, not in the spec PDF.** The spec's last page says they are a separate archive in the tpc.org download section. | blocks the TPC audit (we run our own stand-ins until it is downloaded) |
| Comparison tool and expected results for SF 5 (clause 8.2.1.1) | **Not in the kit.** Also a separate tpc.org download. | blocks clause 8.3 qualification |
| EULA | No TPC EULA file in this kit (the TPC-H kit has one; the terms are accepted on tpc.org at download). PDGF carries its own **bankmark EULA** (`PDGF/LICENSE.txt`) and asks for it to be accepted interactively. | handled, see R4 |

## Risks
- **R1. Java version (medium).** The README asks for Java SE 7 or later, and clause 6.2.6 requires at least SE 7.
  - PDGF compiles its file-name templates at run time with javassist 3.18 (`pdgf.util.CompileableTemplateFactory`,
    used by the `CompiledTemplateOutput` entries in `tpc-di-generation.xml`). Javassist of that age loads the compiled
    class through reflective access to `ClassLoader.defineClass`. Java 9-16 allow this with a warning. From Java 17,
    strong encapsulation (JEP 403) refuses it unless the JVM is started with
    `--add-opens java.base/java.lang=ALL-UNNAMED`. That is our reading of the code, not yet an observed failure.
  - PDGF also contains a JavaScript template class (`javax.script`); the Nashorn engine was removed in Java 15. The
    TPC-DI configuration does not appear to use it (no script templates in the config files), so this is low risk.
  - The most-used third-party DIGen wrapper we found
    ([databricks-tpc-di, `digen_runner.py`](https://github.com/shannon-barrow/databricks-tpc-di)) refuses Databricks
    runtimes newer than 15.4, saying the jar "has not been validated on newer runtimes". Databricks moved its default
    JDK from 8 to 17 at runtime 16.0
    ([release notes](https://docs.databricks.com/aws/archive/runtime-release-notes/16.0)). That is circumstantial,
    but it agrees with the javassist reading.
  - **Mitigation:** pin Java 8 (OpenJDK 8 is packaged for both architectures, e.g. Ubuntu 24.04 `openjdk-8-jre-headless`
    for amd64 and s390x: [packages.ubuntu.com](https://packages.ubuntu.com/noble/openjdk-8-jre-headless)).
    `scripts/build_kit.sh` refuses to continue on anything older than 8, and on 9 or newer it warns and records
    the `--add-opens` option that `gen_data.sh` then passes through DIGen's own `-jvm` switch (a documented DIGen
    option, so no kit change). The Java version is in every manifest.
- **R2. Case-sensitive file systems (high, but trivial).** DIGen's bundled `digen.properties` sets `PDGF_HOME=pdgf`
  (lower case), and DIGen starts PDGF in that folder. The kit's folder is `PDGF`. That works on Windows and fails on
  Linux. **Mitigation:** `build_kit.sh` copies the kit to the work area and adds a `pdgf -> PDGF` symbolic link in the
  copy. No kit file is changed.
- **R3. Paths with spaces (low).** DIGen passes the output folder to PDGF wrapped in single quotes, without a shell.
  **Mitigation:** `env.sh` refuses a `TPCDI_WORK` containing spaces.
- **R4. Interactive licence prompt (low).** PDGF stops at start-up and waits for the bankmark EULA to be accepted by
  typing `YES`. **Mitigation:** `gen_data.sh` never accepts it on the operator's behalf: it pipes the answer only if
  `PDGF_EULA_ACCEPTED=YES` is set in `config.env` by the person who has read `PDGF/LICENSE.txt`; otherwise it runs
  DIGen interactively.
- **R5. Unmaintained code (medium, structural).** DIGen 1.1.0 and PDGF R1343 date from 2014-2015, and the TPC lists
  TPC-DI 1.1.0 as obsolete. Nobody fixes bugs. Clause 6.2.5 makes any DIGen error compliant by definition, and the
  generator is deterministic, so a working run stays working; but a failure on a new JVM or OS will not be fixed
  upstream. **Mitigation:** Java 8 pinned; the build records SHA-256 sums of DIGen.jar, pdgf.jar and tpc-di.jar; the
  data manifest records the sums of every generated file, so s390x output can be compared byte for byte with x86.
- **R6. Line endings (low).** `PDGF/Constants.properties` sets the default line separator to CRLF. The ETL strips a
  trailing CR from every record, and the first generation run records which separator was actually written.
- **R7. Memory (low).** DIGen starts PDGF with `-Xmx1g` unless `-jvm` says otherwise. `gen_data.sh` passes
  `TPCDI_JVM_OPTS` (default `-Xmx2g`).

Unknowns that only a run can settle: whether PDGF R1343 runs on Java 8 on s390x (expected, since it is bytecode;
the plugin may still depend on platform defaults such as the default charset, so the run sets `-Dfile.encoding=UTF-8`
and the byte comparison with x86 output is the test), and how long DIGen takes per scale factor on each machine.

## First check on a quiet machine (about 10 minutes, under 1 GB)
1. `scripts/build_kit.sh` (copies the kit, checks Java, links `pdgf`, records checksums). Seconds.
2. `scripts/gen_data.sh 3` (the smallest SF DIGen accepts). Expect a few minutes and roughly 0.3 GB.
3. Repeat on Linux on Z when access exists, and compare `data-manifest.txt` checksums. Identical files close R1, R2
   and R5 for s390x; any difference is a finding, recorded as such.

## Fallback if DIGen cannot be made to run
Keep the module's purpose (nightly batch ETL from mixed-format files, historical then incremental) without the TPC
generator:
- **Source data:** TPC-H `dbgen` (already built by `analytics/tpch`, portable C, proven on x86 and s390x), with its
  refresh sets (`-U`) as the daily change files. The result would be labelled "derived from TPC-H" and never "TPC-DI".
- **Formats:** convert dbgen's `|` files into the formats the batch world uses: fixed-width records (the
  mainframe-like case, with a published copybook-style layout), CSV, and one hierarchical XML feed (customers),
  each conversion step outside the timed window and checksummed.
- **ETL workload:** CardDemo's batch jobs (POSTTRAN, INTCALC, CREASTMT; SPEC 1.3) fed by those files, plus a
  dimensional load into PostgreSQL using our own SQL (type 2 dimensions, fact appends), timed per phase as here.
- **Cost:** our own spec text and validation (row counts and checksums against a reference run), no TPC audit at all.
  It answers the same question with less external comparability.
