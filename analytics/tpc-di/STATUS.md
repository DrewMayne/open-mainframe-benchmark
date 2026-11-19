# Status: batch ETL module (Derived from TPC-DI)

> **This workload is derived from the TPC-DI Benchmark and is not comparable to published TPC-DI Benchmark results, as this implementation does not comply with all requirements of the TPC-DI Benchmark.**

Updated 2026-11-19. Built on a PC that was running a timed benchmark at the time, so **nothing has been run**: no
Java, no DIGen, no PostgreSQL, no data. Everything below marked "built" is code reviewed by reading only; the pure
Python tests are the only things executed.

## Verified (by reading or by running here)
- Kit extracted to `E:\tpc\TPC-DI\Tools\` (from `dev\tpc-kits\8671564C-...-TPC-DI-Tool.zip`, 108 files, 8.6 MB).
  Wrapper source unpacked beside it in `E:\tpc\TPC-DI\DIGen_Build_src\` (read only, for the risk check).
- Kit contents: DIGen 1.1.0 wrapper, PDGF R1343 (binary only), tpc-di plugin; **no specification, no TPC EULA file,
  no appendix SQL, no comparison tool**. FEASIBILITY.md lists the facts and their sources.
- Specification `TPC-DI_v1.1.0.pdf` downloaded separately from tpc.org to `E:\tpc\TPC-DI\spec\` (117 pages). Its last
  page says Appendices A-C (audit, batch validation, visibility SQL) are a separate download.
- Risk check done (FEASIBILITY.md): workable on Java 8; Linux needs the `pdgf -> PDGF` link (handled in
  build_kit.sh).
- `python3 -I tests/test_pure.py`: 10 tests pass (timing rounding, figures and the 1800 s floor, digen_report parsing,
  result formatting, FDR rendering with no TPC metric names).

## Built, not yet run
- scripts: build_kit.sh, gen_data.sh (+ py/gen_data.py), setup_pg.sh, env.sh; pg/postgresql.tpcdi.conf.
- py: run.py (phases, PCR, visibility session, CPU split), metrics.py, manifest.py, fdr.py, extract.py, common.py,
  db.py.
- DI application: sql/ddl/warehouse.sql (all 20 clause 3.2 tables), sql/ddl/staging.sql (UNLOGGED staging tables,
  helper functions, auxiliary indexes), sql/hist/01-12, sql/incr/01-08, py/etl.py (init, run_phase, load_audit;
  COPY streaming, FINWIRE as raw lines cut in SQL, CustomerMgmt.xml by iterparse), py/checks.py (our stand-in batch
  validation, visibility and audit; the clause 8.2.2 result-file definitions). Three DEVIATIONs, listed in
  COMPLIANCE.md. `python3 -I tests/test_etl_pure.py`: 20 tests pass.

## Needs a quiet machine (Linux, not WSL while another study runs)
| Step | Command | Estimated time | Disk |
| --- | --- | --- | --- |
| Install Java 8, PostgreSQL 17, psycopg | packages | minutes | < 1 GB |
| Stage the kit | `scripts/build_kit.sh` | seconds | 10 MB |
| Smallest generation (proves DIGen on this OS) | `scripts/gen_data.sh 3` | a few minutes (estimate) | about 0.3 GB |
| Qualification data | `scripts/gen_data.sh 5` | a few minutes (estimate) | about 0.5 GB |
| First run, SF 5 | `py/run.py --sf 5 --db tpcdi_qual --qual ...` | unknown; expect tens of minutes on 2 vCPU, mostly the historical load (the visibility queries run beside the updates and add little) | about 2.5 GB with the warehouse and WAL |
| Measured run, SF 10 | `gen_data.sh 10`, `run.py --sf 10 ...` | unknown; roughly twice SF 5 | about 5 GB |
| s390x check | the same build and SF 3 generation on Linux on Z, then compare `data-manifest.txt` with x86 | minutes | 0.3 GB |
Size figures use a third-party "about 1 GB of source per 10 SF" (README); every figure here is an estimate until
the first run replaces it.

## Blocks
1. **The TPC appendix archive** (audit, batch validation and visibility SQL; spec Appendices A-C) is a separate
   tpc.org download, behind the TPC EULA. Andrew needs to download it and set `TPCDI_APPENDIX`. Until then the audit
   phase runs our stand-ins and no run can say the TPC audit passed.
2. **The comparison tool and the SF 5 expected results** (clause 8.2.1.1): also a separate download. Without them,
   clause 8.3 qualification cannot be shown.
3. **The first real run** will surface SQL errors: the ETL has not been executed against any database (see the
   expected-breakage list below).
4. **Linux on Z access** (ROADMAP M3) for the s390x leg.

## Expected to break on the first run
None of the SQL has been executed, so expect a debugging pass at SF 3 before any timed run:
1. Syntax or typing errors, most likely in the less common constructs: writable CTEs (`insert ... returning`), the
   `tpcdi_last` aggregate used as a window function, `RANGE ... interval '1 year -1 day' PRECEDING` on a date column,
   and `row(...)::text` comparisons.
2. NOT NULL violations that would reveal a logic gap: DimTrade.SK_CreateDateID (a trade with no PNDG history and no
   market-order SBMT row), DimCustomer.DOB or TaxID on a NEW action, DimCompany.Industry or Status on a missed lookup.
3. Values longer than the warehouse types: the kit's schema allows 80 characters for Prospect state and country
   (warehouse 20 and 24); Prospect NetWorth may not fit DimCustomer.NetWorth NUMERIC(10). If DIGen really emits such
   values, that is a spec/kit conflict to record, not to hide by widening columns.
4. CustomerMgmt.xml quirks seen in the kit template: an empty address line written as `<C_ADLINE/>` (read as absent),
   and UPDCUST branches that could emit malformed XML if those fields ever change (iterparse would raise).
5. PE ratio: if FI_YEAR/FI_QTR do not line up with calendar quarters, every PE ratio is NULL and alerts flood.
6. digen_report.txt: the TotalRecords line format is assumed; metrics.parse_digen_report stops the run if it differs.
7. The TPC appendix SQL, once downloaded, may need PostgreSQL minor modifications (clause 3.4.11.1); none are coded.
8. Speed at larger SFs: the DimTrade and FactMarketHistory joins and the CTE over all of DailyMarket; CustomerMgmt
   account rows are held in memory until the customer COPY ends.

## Questions for Andrew
- Download the TPC-DI appendix archive and the comparison tool from tpc.org (EULA acceptance is yours to give)?
- PDGF asks for its own (bankmark) EULA to be accepted with `YES`. Do you want to accept it yourself at the first
  generation, or set `PDGF_EULA_ACCEPTED=YES` in your local config.env after reading `PDGF/LICENSE.txt`?
- OK to keep this module's PostgreSQL cluster on port 5433, beside the TPC-H one on 5432?
- docs/methods/energy.md found that z17 / LinuxONE Emperor 5 report firmware-apportioned power per partition to the
  OS (Linux `zpwr`, z/OS SMF 70.1 via APAR OA66018, z/VM 7.4 monitor). SPEC.md 1.7 says shared Z "can't be measured".
  Keep the energy doc's proposed optional "IBM firmware-apportioned partition power (vendor estimate; not measured)"
  FDR field, and reword SPEC 1.7 to match?
