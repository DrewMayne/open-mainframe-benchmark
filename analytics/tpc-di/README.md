# Batch ETL module (Derived from TPC-DI)

> **This workload is derived from the TPC-DI Benchmark and is not comparable to published TPC-DI Benchmark results, as this implementation does not comply with all requirements of the TPC-DI Benchmark.** This module is academic research with no
> marketing claims (TPC EULA clause 4.c). TPC metric names are never used for its figures (SPEC.md section 6).

**The TPC lists TPC-DI 1.1.0 as obsolete: no new official TPC-DI results can be published, by anyone.** We use it
because it is still the only standard description of the workload we want: nightly batch ETL from mixed source
formats into a warehouse, a historical load followed by incremental updates. Comparability with TPC results is
therefore not a goal, and was never possible; following the spec closely is still useful, because it makes the
workload well defined and checkable by others.

Milestone **T2** (ROADMAP.md), SPEC.md 1.7. Status: **built, not yet run** (see STATUS.md).

## What it measures
The "nightly batch" in its most mainframe-like form: a retail brokerage's OLTP extracts, an HR file, a prospect list,
a hierarchical customer-management feed (XML) and a financial newswire of **fixed-width records** in three layouts
(FINWIRE), loaded into a dimensional warehouse with type 2 history dimensions and fact tables.
- **Phases** (spec clause 7.2): initialization, historical load (Batch1), incremental update 1 (Batch2), incremental
  update 2 (Batch3), automated audit. Each timed phase ends with batch validation and a phase-completion record whose
  timestamp gives the elapsed time (clause 7.5).
- **Our figures:** elapsed time and CPU split per phase and per step, and source rows per second for each phase,
  computed with the clause 7.5/7.6 arithmetic but reported under neutral names (`py/common.py`).
- **Targets:** PostgreSQL on x86_64 first, then the same code on Linux on Z (s390x). A z/OS variant is an idea only
  (below).

## Before you start: get the TPC material (never into this repository)
SPEC section 6 applies: kits are downloaded by each user from tpc.org, accepted under the TPC EULA, and extracted
outside the repo (we use `E:\tpc\TPC-DI\`). Scripts point at them through `config.env`.
1. **The TPC-DI tools kit** (DIGen 1.1.0 with PDGF). Extract it so that `TPCDI_KIT` is the folder holding `DIGen.jar`
   and `PDGF/`.
2. **The specification** is *not* in the kit. Download `TPC-DI_v1.1.0.pdf` from the TPC's current specifications page.
3. **Not in the kit, not in the spec (separate tpc.org downloads):**
   - the appendix archive with the TPC's audit, batch validation and data visibility SQL (spec Appendices A-C).
     Point `TPCDI_APPENDIX` at it. Until then, `py/checks.py` runs our own stand-ins, clearly labelled, and a run
     cannot claim the TPC audit passed;
   - the comparison tool and the SF 5 expected results used for qualification (clause 8.2.1.1).
4. **PDGF's own licence.** PDGF (by bankmark) shows its EULA (`PDGF/LICENSE.txt`) and asks you to type `YES`. Read it
   yourself; `gen_data.sh` only answers for you if you set `PDGF_EULA_ACCEPTED=YES` in `config.env`.

## Requirements
- Linux on x86_64 or s390x, Python 3.10+, `psycopg==3.2.3` (requirements.txt), PostgreSQL 16 or 17.
- **Java 8** for DIGen (FEASIBILITY.md R1: the bundled PDGF is from 2014 and uses an old javassist that newer JVMs
  restrict). OpenJDK 8 is packaged for both architectures.

## Quick start
```
cp config.example.env config.env          # edit TPCDI_KIT, TPCDI_WORK, PG_BIN, JAVA_HOME
scripts/build_kit.sh                      # stage the kit, check Java, record versions and checksums (seconds)
scripts/gen_data.sh 5                     # SF 5: the qualification scale factor (clause 8.1.3)
scripts/setup_pg.sh init                  # private cluster on port 5433
python3 -I py/run.py --sf 5 --db tpcdi_qual --out ~/tpcdi-work/results/qual-sf5 --qual
python3 -I py/extract.py --db tpcdi_qual --out ~/tpcdi-work/results/qual-sf5/result-data   # for the TPC tool
python3 -I py/fdr.py --run ~/tpcdi-work/results/qual-sf5
```
For a measured run, generate a larger SF (any integer >= 3; clause 7.2.1.1.1 asks for one where each incremental
update finishes within 3600 s) and use a new `--db tpcdi_sf<N>`. `run.py` drops and recreates that database at the
start of every run (clause 7.2.3). It refuses database names that do not start with `tpcdi_`.

## Size and time estimates (to be replaced by measurements)
- Source data: about 1 GB per 10 SF (a third-party figure from
  [databricks-tpc-di](https://github.com/shannon-barrow/databricks-tpc-di); not yet measured here). SF 5 is
  roughly 0.5 GB, SF 10 roughly 1 GB, SF 100 roughly 10 GB.
- Disk for a run: the Staging Area, plus the warehouse and WAL, which we expect at 2-4 times the source size; allow
  about 5 times the source size in total.
- Elapsed time: unknown until the first run. Generation should take minutes at SF 5-10. The historical load is
  expected to be most of the work, since Batch1 holds most of the rows. At small SFs the incremental updates take
  seconds, so the clause 7.5.3 1800 s floor dominates the incremental figures; we also report the raw rows per
  second without the floor, under its own name.

## Layout
| Path | What |
| --- | --- |
| `FEASIBILITY.md` | Risk check of DIGen on current Linux and s390x, and the fallback |
| `COMPLIANCE.md` | Clause-by-clause: how each rule is met, or why not |
| `STATUS.md` | What is built, what has run, what blocks the next step |
| `scripts/` | `build_kit.sh`, `gen_data.sh`, `setup_pg.sh`, `env.sh` |
| `py/run.py` | The runner: phases, PCR records, the visibility session, timings, CPU split |
| `py/etl.py`, `sql/` | The DI application: our own SQL and Python (`sql/ddl`, `sql/hist`, `sql/incr`) |
| `py/checks.py` | Our stand-ins for the batch validation, visibility and audit queries; the result-file definitions |
| `py/extract.py` | Clause 8.2.2 result files for the TPC comparison tool |
| `py/manifest.py`, `py/fdr.py`, `py/metrics.py` | Run manifest, FDR generator, figures |
| `pg/postgresql.tpcdi.conf` | The only PostgreSQL settings changed, disclosed in every FDR |
| `tests/` | Pure unit tests (no database): `python3 -I tests/test_pure.py` and `tests/test_etl_pure.py` |

## Design choices
- **Lean and clearly ours.** Set-based SQL does the transformations; Python reads the files, parses the XML and
  streams rows into PostgreSQL with COPY. FINWIRE's fixed-width records are loaded as raw lines and cut with
  `substring()`, as a mainframe job would cut a record with a copybook.
- **Not a commercial DI product.** The spec requires one (clause 4.2.1, 4.3.1); we use our own code so that the same
  logic runs unchanged on x86 and s390x and can be read by anyone. This is a standing deviation (COMPLIANCE.md).
- **Same code on both architectures,** nothing architecture-specific, and the data manifest's checksums let the s390x
  Staging Area be compared byte for byte with x86's.

## Later: a z/OS variant (idea only, not designed or built)
How the same workload could run in its native habitat, for the phase 2 z/OS work (ROADMAP M5):
- **Staging Area:** the DIGen files transferred to z/OS as sequential data sets (FB for FINWIRE, whose records are
  fixed width per type; VB for the delimited files), or read in place from zFS by batch jobs.
- **Transformations as a JCL job stream:** DFSORT/ICETOOL for the sort, select, reformat and join-like steps
  (`JOINKEYS` for key lookups; `IFTHEN` to split FINWIRE by record type; `INREC/OUTREC` for field cuts and date
  conversion), COBOL batch programs for the rules DFSORT cannot express (type 2 expiry chains, alerts, the 52-week
  window), and the XML feed parsed with COBOL `XML PARSE` or converted on USS first.
- **Warehouse:** Db2 for z/OS, loaded with the LOAD utility (historical) and with SQL MERGE/INSERT from COBOL or
  batch SPUFI/DSNTEP2 steps (incremental), so it can be compared with the PostgreSQL warehouse by the same audit.
- **Measurements:** elapsed time per job and step from SMF type 30, CPU split by GP and zIIP, and the same
  phase-completion records written to DImessages.
- **Open questions:** whether the clause 8.2 comparison can be reproduced on Db2 output, and whether DFSORT-heavy
  steps should run with and without zIIP offload. None of this is planned work yet.

TPC, TPC-DI and TPC-H are trademarks of the Transaction Processing Performance Council.
